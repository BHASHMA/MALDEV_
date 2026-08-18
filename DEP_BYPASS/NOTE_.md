
Data Execution Prevention (DEP), simply the CPU makes the .txt part NX i.e Non Executable. We can write data to the memory , but cant execute it.

Enabling DEP....

1. Search :  Windows Defender Security Center
2. Go-to   :   App and Browser Control
3. Scroll Down : Exploit Protection Setting
4. Go-to : Program Setting
5. Add [+] Program to customize , feed the relative path of the binary
6. On Program Setting .... Scroll Down till...

![572](img_/enablin_dep.png)

Checkin' it with Windbg...

![](img_/chekin_dep.png)

We just added dummy shellcode of four NOP's to the EIP i.e the current shellcode, then found the access violation...The **Access Violation (c0000005)** error occurs when a program attempts to **read, write, or execute** a memory address it does not have permission to access.


Cool ! Now as an attacker the first thing in mind is how can I bypass this or exploit this. So for that we use Return Oriented Programming (ROP) or Jump Oriented Programming (JOP). JOP is mostly used where we do not have ret. basically when exploiting ARM. 


### Return Oriented Programming (ROP)

Attack where we use assembly instructions (called gadgets) and API's which are already loaded in the process.

Simply...as DEP is enabled it makes some area non-executable(NX), So we need some way to bypass. There we use ROP, where we use loaded instructions to either allocate executable memory, or changing the permission, or even write the shellcode in (.txt) section to bypass DEP.

First Bypass was ret-2libc (from Linux back in the days), then over the time this exploit's called ROP Chain attack. There are basically two approaches:
1. Build 100% ROP Shellcode (Complex - sans 760)
2. Build ROP chain that leads to executing our shellcode.

We discussin' about the Rop chain.

One way to implement this attack is to allocate memory with write and execute permission and copy our shellcode [Win 32 Virtual Alloc]

OR, Change the permission of the memory page where shellcode already resides. [Virtual Protect]

OR, [Win32 Write Process Memory] - hot patches the code section (.txt) of running process, then injects the shellcode to it. Completely bypass DEP/

ROP Chain is sequence of gadgets  address placed on the stack.

Buffer Overflow ---> Control EIP ---> Execute ROP Chain ---> Call Virtual Protect() ---> Memory Becomes Executable ---> Jumps to the shellcode.



### Finding Gadgets (Small assembly instructions ending with RET)

Why RET ? - RET pops the next address from stack and jumps to it..


We use Rp++ (Open Source Gadgets finder)
1. Copy the executable / dll
2. rp++ -f file.exe/dll -r 5 > rop.txt

Vs-Code Regex 

1. push *** ; push *** ; pop *** ; pop *** ; ret [Best]
```
push\s+esp[\s\S]*?pop\s+(esi|eax|ebx|ecx|edi|edx)[\s\S]*?ret
```


2. push *** ; push *** ; pop *** ; mov 
```
push\s+esp[\s\S]*?pop\s+(esi|eax|ebx|ecx|edi|edx)[\s\S]*?pop\s+(esi|eax|ebx|ecx|edi|edx)[\s\S]*?ret
```