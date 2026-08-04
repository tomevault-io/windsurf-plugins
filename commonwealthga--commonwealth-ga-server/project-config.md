---
trigger: always_on
description: Entry point for AI assistants (Claude Code, Copilot, etc.) working on this repo. Read this every session before touching code.
---

# CLAUDE.md

Entry point for AI assistants (Claude Code, Copilot, etc.) working on this repo. Read this every session before touching code.

## What this is

A custom server reimplementation for an old UE3 multiplayer game. The original server is gone; we reconstruct it by hooking the **client binary** with a DLL injected via Detours. Most "missing" behavior we add is server-side reimplementation of native UC functions that the shipped binary stripped or stubbed.

See `README.md` for project background, roadmap, and Discord/community links.

## ‼️ HARD RULES — non-negotiable

These have been learned the hard way over hundreds of sessions. Re-deriving why each one exists wastes the human's time, so just follow them.

### Build / commit / tooling

- **NEVER build the project.** Never run `make`, `make server`, or any compile/link command. The human builds locally. Static re-reading of the code is the only acceptable verification.
- **NEVER auto-commit.** The human controls all `git commit`s. Even if a plan you wrote and the human approved lists "git commit" as a step, do NOT commit. Stage at most. Plans should omit commit steps entirely.
- **Add every new `.cpp` to `Makefile`** in the same response that creates it. The Makefile uses an explicit source list, NOT wildcards. Forgetting → silent skip → linker `undefined reference` → wasted build cycle. Treat `.cpp` creation + `Makefile` edit as atomic.
- **No filesystem snooping.** Do not `find /`, do not read or list paths outside the project repo without asking first. If you think you need a file outside `src/`, ask.

### Project mindset

- **Everything is server-fixable.** The game client is unchanged from the original-server era. If a feature used to work and doesn't now, it is a SERVER gap — not a client bug. Never say "client-side, can't fix from server." Find what marshal / RPC / native / event the original server emitted, and emit it.
- **Do it right — never offer "correct vs. workaround" as a choice.** When a canonical/faithful implementation path is identifiable, just do it. Don't surface big-bang vs. stopgap as a user question. Big-bang risk is mitigated by static review of the correct impl, not by shipping a hack first.
- **Don't pile hacks on working code.** "This used to work, the rewrite broke it" → `git log`, revert the speculative manipulation, find the regression. Don't add `SetCollision` toggles, `SetLocation` cargo-cult fixes, `ForceUpdate` flushes etc. on top of a broken rewrite.
- **No UC modifications.** The client UnrealScript is unmodified retail. You cannot add classes, fields, or `defaultproperties`. Translate UC idioms to C++ vtable / ProcessEvent detours.
- **Apply, don't describe.** When the fix is concrete and low-risk, make the edit. Don't end the turn with a how-to paragraph.
- **No speculation.** Only state facts from logs, code, and test results. Theories about why something might work go in conversation, not in source comments and not as final answers.
- **Comment out, don't delete.** When disabling code, use `//` not deletion. Code you removed is easier to restore from `//` than from `git log`.
- **Short verified comments only.** No long comments, no theories in source. The "why" of a tricky line is one sentence. Speculation belongs in the conversation.

### Hooking & native reimplementation

- **Stub-native hooks must call the parent hook directly, NOT `CallOriginal`.** When the hooked native is a stub (no-op in the binary), `CallOriginal` is a no-op and does NOT chain to parent class overrides. You must explicitly `ParentHook::Call((AParentType*)derived, edx)`. When the parent native is intact, `CallOriginal` works. Decision is per-hook, based on stub-vs-intact.
- **MSVC struct-return convention in Detours hooks.** Hooks on natives returning `FVector` / `FRotator` / any struct must declare return type as `OutType*` and `return outLoc;` at every exit. Void return → EAX junk → caller dereferences null → crash.
- **Reimplement on the engine class, never build a parallel module.** When the game has a purpose-built class (e.g. `ATgTeamBeaconManager`, `UTgEffectManager`) with stripped natives, fill in the natives ON THAT CLASS. Don't invent a sibling C++ module that re-stores its replicated state.

### Class identity & object lookup

- **NEVER `obj->IsA()`.** Unreliable on this build (SDK `StaticClass()` indices misalign with the binary's `GObjObjects`).
- **NEVER iterate `GObjObjects()` directly.** It's slow, scales linearly with world object count. Use one of the caches below.
- **Class identity → `ObjectClassCache::ClassNameContains(obj, "TgPawn")`** (or `GetClassName(obj)` for the underlying `const std::string&`). Resolves once per `UClass*`, hands out a stable string thereafter. Solves the GetFullName shared-buffer hazard automatically.
- **Need a `UClass*` to spawn / compare → `ClassPreloader::GetClass("Class Pkg.ClassName")`.** Do NOT add new per-class helpers (`GetTgFooClass()`); the existing ones are legacy.
- **Find an object by full name → `ObjectCache::Find(fullName)`.** Lazy progressive scan, caches as it goes.
- **Map-baked actors (player starts, mission objectives, volumes, …) → `ActorCache`.** Typed vectors, populated once at map load.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [commonwealthga/commonwealth-ga-server](https://github.com/commonwealthga/commonwealth-ga-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
