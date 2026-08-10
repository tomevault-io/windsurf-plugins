---
trigger: always_on
description: Working rules for anyone (human or AI) touching this repo. They encode the
---

# AGENTS.md — fear2vr project rules

Working rules for anyone (human or AI) touching this repo. They encode the
conventions established while bootstrapping the project; TESTING.MD owns the
testing contract and is referenced (not duplicated) here.

## What this is

fear2vr: a 32-bit injected-DLL mod framework for F.E.A.R. 2: Project Origin
(LithTech Jupiter EX, D3D9), heading toward a VR mod. Rapid iteration via
inject → test → uninject with the game never restarting.

## Hard rules

1. **32-bit only.** Build is `-A Win32`; cmake fails hard on 64-bit. `Eip` not
   `Rip` in thread contexts; x86 `__thiscall` detours are written as
   `__fastcall(this, edx_dummy)`. The one 64-bit thing in the repo is
   `tools/xr64`, the OpenXR host, and it is deliberately NOT in this build --
   it configures on its own (`cmake -B build64 -A x64 -S tools/xr64`). Nothing
   in the mod calls OpenXR; the two processes meet only at the shared mapping
   in `shared/xr/SharedFrame.hpp`.
2. **No test code in the shipped mod (fear2vr.dll).** Assertions live host-side
   in `test/fixture_test_runner.cpp`. The DLL may expose *diagnostics*
   (`/health`, `/sdk/targets`, `/engine-hook`) — data, never pass/fail
   judgement. Rules of evidence are in TESTING.MD and are load-bearing.
3. **Commit frequently.** Small, build-clean commits. Never commit a tree that
   doesn't build; run `ctest --test-dir build -C RelWithDebInfo --output-on-failure`
   before committing. Commit after each coherent unit (scaffold, one SDK class,
   one test tier), not in one end-of-day blob. Never `git push`.
4. **REFramework/UEVR structure.** Singletons and the `g_framework` global, mir-
   roring `I:\Programming\projects\re2-barebones` and `ue4poc`:
   - `Framework` lives behind `extern std::unique_ptr<Framework> g_framework`
     (set once by the supervisor; `release()`d before unmap — the CRT must not
     run its destructor during `DLL_PROCESS_DETACH`).
   - Managers are `Xxx::get()` singletons: `Hooks::get()`, `Mods::get()`.
     `Hooks` is deliberately *leaked* (never deleted): `~InlineHook` frees
     trampoline memory that straggler game threads can still be executing
     during unmap. Do not "fix" that leak.
   - Features are `Mod` subclasses registered into `Mods::get()`; the framework
     fans out `on_initialize` / `on_frame` / `on_shutdown`.
   - Naming: snake_case methods/functions, PascalCase types/files, `m_`
     members, `g_` globals, `s_` statics. Errors as
     `std::optional<std::string>` (empty = success).
5. **Unknown reader or writer? Use `/watch/*`, never an offset scan.** The mod
   exposes HARDWARE data breakpoints — DR0-DR3 plus a vectored exception handler,
   four slots, byte-exact:

   ```
   /watch/arm?addr=0x02938740&size=4&type=write|rw|exec&max_hits=4000
   /watch/report      -> accessors, registers, value at trap, caller candidates
   /watch/clear?all=1
   ```

   Every hit resolves to `module + offset + static`, so it pastes straight into
   the right IDB. `ecx` is reported separately because `__thiscall` puts `this`
   there — which is how you tell *"writes +144 on something"* from *"writes +144
   on THE camera"*.

   **This is mandatory for "what touches X".** Scanning the binary for stores to
   a struct OFFSET answers a different question than the one you asked (67
   functions across unrelated classes, in the case that established this rule)
   and has produced a plausible WRONG answer every single time it was tried
   here. Every instance is recorded in `reversing/REVERSING_LESSONS.md`.

   **The workflow is: trap it, then read it.** Arm the watch, take the reported
   static address and caller candidates into IDA, decompile the writer, walk the
   stack upward until you find the function that *decides* — then hook THAT.
   Worked example, end to end: the engine clock's address was published, a write
   watch named the store (`add [esi+30h], eax` in `CLTTimer_TickNode`), the
   stack gave `CClientMgr__Update -> CLTTimer_AdvanceByWallClock ->
   CLTTimer_TickChildren`, and the decompiled tick gave the whole timer-node
   layout including the pause byte and rational time scale that turned out to be
   what alt-tab actually freezes. Two hardware semantics matter and are asserted
   in the suite rather than remembered: a data watch is a TRAP, so the reported
   address is the instruction *after* the accessor (`eip_after`), while an
   execute watch is a FAULT and reports the instruction itself; and x86 has no
   read-only data breakpoint, so a read request is served by read-or-write.

6. **SDK = class-per-concept with function-local static resolution.**
   `shared/sdk/` has one class per engine concept (`sdk::CClientMgr`,
   `sdk::CClientShell`, `sdk::Engine`, `sdk::DatabaseMgr`, `sdk::Modules`).
   Each class *owns* its patterns and derivation logic in its own .cpp;
   resolution happens inside the accessor:
   ```cpp
   uintptr_t CClientMgr::update_fn() {
       static const uintptr_t s_fn = Modules::get().scan_exe(kUpdate, "CClientMgr::Update");
       return s_fn;
   }
   ```
   - A global engine object gets `SomeClass::get()` on its owning class.
   - **No dumping-ground files.** No `detail/Scan.*`, no "helpers" TU holding

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [praydog/FEAR2VR](https://github.com/praydog/FEAR2VR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
