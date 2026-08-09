---
trigger: always_on
description: Debugger for the [HashLink](https://hashlink.haxe.org) VM.
---

# hashlink-debugger

Debugger for the [HashLink](https://hashlink.haxe.org) VM.

## Code style

**Never let a tool change a file's line endings.** Files here are a mix of CRLF and LF, per file, in
this repo and in the HashLink and Haxe sources. `sed -i` under MSYS rewrites a CRLF file as LF, so a
one-line substitution turns into a whole-file diff that buries the real change — and it does it to
every file the glob matched, including the ones the substitution did not touch. Use the editing tools,
which preserve the convention; if a batch edit is genuinely needed, do it in a way that writes bytes
back unchanged, and check `git diff --stat` afterwards: a file whose insertions plus deletions equal
its length was rewritten, whatever the visible diff says.

**Do not add comments**, unless one is genuinely necessary. This holds for every file you touch —
this repo, the tests, and the HashLink and Haxe sources. Put whatever a change needs explained in
this file instead.

Existing comments are **not** off limits: edit, adapt or delete them as the code around them changes.
A comment left describing behaviour that no longer exists is worse than no comment, so a change that
invalidates one must update it rather than work around it.

## Architecture

- The debugger gets its data from the HashLink VM by **connecting to a socket**; the other end of
  that socket lives in the HashLink sources.
- Debug info (line numbers, variable names, register mapping) is embedded in the `.hl` file
  **by the Haxe compiler**.

VM and compiler internals are deliberately **not** documented here: they move independently of this
repo, and a stale description is worse than none. Where the debugger depends on something the VM
produces, this file names the debugger-side code that decodes it — read the current layout there, and
check the VM sources when that is not enough.

### Walking the stack

`Debugger.makeStack` does not unwind, it **scans**: every stack word that looks like a saved ebp
followed by a word that resolves into JIT code is taken for a frame. Three things keep that from
inventing frames, and all three are needed.

- The VM **erases the return address** of every HL call once the call returns, so the debris of calls
  that already completed cannot match. Without it the scan reports frames from a chain that is long
  gone. Erasing is on only under `--debug` without `--debug-opt`; V1 has always done this, V2 lost it
  and had it restored.
- `Debugger.isCallerOf` accepts a candidate only if it is **suspended on a call to the frame already
  found below it**, which anchors the whole chain on the innermost frame — that one is exact, it comes
  from the instruction pointer. The target is compared where the bytecode names it; for a closure,
  method or `this` call it cannot, and any callee is then accepted.
- A candidate's ebp must be **strictly above** the ebp of the frame already found below it. The stack
  grows down, so a caller always sits at a higher address, and a candidate that does not is debris —
  it is rejected rather than pushed, and the scan keeps looking. This is the only guard that survives
  a candidate `isCallerOf` cannot judge: it costs nothing, because the scan already walks in
  increasing address order.

The erase decides where a caller frame *resolves*, not only whether it is found: a return address
lands on the op **after** the call whenever the call is its op's last instruction, and the erase
instruction is what keeps it inside the call op. So without it every caller frame in `bt` reports the
following line — `tests/unit/TestCallerLine` pins that. `isCallerOf` tolerates both, since a native
call is never erased.

The erase also only covers a call that **returned**: an exception unwinds past its frames instead, so
everything the throw skipped keeps a real saved-ebp/return-address pair. A function entered after the
catch stands on that debris, and a frame with enough locals holds several of those pairs — each one a
candidate the ebp order rejects and nothing else would. Where the debris was left by a closure, method
or `this` call, `isCallerOf` waves it through, and the resulting frame reads as a wrong object of a
wrong type or throws a memory read failure that ends the session.

`tests/unit/TestStaleFrames` pins it. It reproduces on V2 and not on V1, whose frame layout puts no
usable pair inside the stopped function — so it guards V2 and merely runs on V1, which is still reason
to keep it in `tests/unit/`: it needs nothing from v6.

The ebp order is not checked for the synthetic frame the scan pushes at the bottom slot of the
scanned range while the innermost function is still in its prolog: that frame's ebp is `esp`, not a
scanned value, and the caller's saved ebp is not on the stack yet.

The scanned range does not always start at `esp`. A JIT frame of tens of kilobytes is subtracted from
`esp` in one go, and a function that never touches its lowest slots leaves those pages **uncommitted**
— one `api.read` over `esp → stackTop` then fails on the whole live stack, at the first stop, before a
single command. So `makeStack` falls back to `findReadableBase`, which binary searches the lowest
address the rest of the range reads from. That is sound because committed stack pages are contiguous

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vshaxe/hashlink-debugger](https://github.com/vshaxe/hashlink-debugger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
