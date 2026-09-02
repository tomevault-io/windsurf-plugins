---
trigger: always_on
description: Accessibility mod/contribution for **Star Wars: Knights of the Old Republic** (KOTOR 1, BioWare 2003, Steam release) enabling blind and visually impaired players to play using screen readers (NVDA/JAWS/Narrator).
---

# Voice of the Old Republic — KOTOR 1 Accessibility Mod

## Purpose
Accessibility mod/contribution for **Star Wars: Knights of the Old Republic** (KOTOR 1, BioWare 2003, Steam release) enabling blind and visually impaired players to play using screen readers (NVDA/JAWS/Narrator).

## Developer accessibility
The developer of this mod is blind. Any workflow you propose has to be doable with a screen reader and keyboard alone. Things to avoid unless there is no alternative:
- Mouse-only steps ("click here", "drag this", "hover over X")
- "Check the screen", "look at the colour", "see if the icon shows" — anything that needs visual confirmation
- GUI tools that aren't screen-reader-friendly (most Ghidra panels, raw imhex hex dumps, image diff viewers, etc.)
- Asking the user to read what's on a loading/black screen, or to time something visually
- Sighted-only verification ("does the cutscene look right?", "is the character in the right spot?")

Prefer instead: CLI commands the user can run, log lines we can grep, deterministic test states, screen-reader-announceable outputs, and "speak what you hear" as the user's verification channel. When a sighted check would normally be the fast path, treat it as a last resort and ask the user explicitly before proposing it.

## Accessibility Goals
- Well-structured text output (no tables, no graphics)
- Linear, readable format for screen readers
- Full keyboard navigation support
- Announce context changes, focused elements, and game state

## Claude Response Formatting
- Never use markdown tables (| symbols are read aloud by screen readers)
- Use headings and bullet lists for comparisons
- Present information linearly, one item per line
- Group related info under clear labels
- Don't write raw memory addresses in chat answers. Use the function name, field name, or a short recognisable description instead ("PlayFootstep's field6_0x20 check", "the engine's natural early-out JZ", "the wrapper's TEST EAX,EAX"). Addresses bloat answers without informing the reader. Only include an address when it is itself the point — e.g. when distinguishing two adjacent hook candidates by their exact offset, or when reporting a finding that has to be re-typed into Ghidra/hooks.toml. In code, hooks.toml comments, memory entries, and docs, addresses are still encouraged.

Example - instead of tables, format like this:
**Item Name**
- Property: Value
- Property: Value

## Permissions
- NEVER add broad PowerShell wildcard permissions like `Bash(powershell -Command:*)` or `Bash(powershell:*)` to settings.local.json - always use specific, scoped commands only

## Code Standards
- Modular, maintainable, efficient code
- Avoid redundancy
- Consistent naming
- Verify changes fit existing codebase before implementing - read and understand surrounding code first
- When contributing to an existing project, match the existing style, patterns, and conventions
- When fixing UI interaction bugs (e.g., keyboard event handling, focus management), always test edge cases where the fix might interfere with normal component behavior
- Prefer editing existing files over creating new ones
- Only make changes that are directly requested or clearly necessary - no drive-by refactors
- Before adding a helper, search for an existing one — duplicate utilities are a recurring failure mode across these accessibility projects
- Name a helper for what it DOES, never for the screen it is used on. `ReadCharSheetLabel` / `ReadEquipLabel` hid three copies of `engine_reads`' `ReadLabelTextAt`, because no grep could find them from either direction
- The same applies to infrastructure: check whether a mechanism exists before proposing to build one. `acclog` already has `Trace` (line dedup), `BlockLog` (block dedup), `Once` and `Edge`

## Non-obvious constraints
- **MSVC C2712: a function using `__try` cannot contain anything needing unwinding** — no objects with destructors, and no function-local `static` (its thread-safe init guard counts). Engine-reading code here is SEH-heavy by convention, so RAII and lazily-cached constants are unavailable across much of the patch. Workaround when you need an object anyway: declare it in an SEH-free CALLER and pass a pointer in — a pointer is trivially destructible, so the constraint never fires and the SEH code needs no restructuring (see `map_ui_cursor.cpp` / `combat_special_watch.cpp` passing `acclog::BlockLog*`)

## Bash Tool on Windows
- Bash tool runs through bash shell (Git Bash), NOT CMD or PowerShell
- NEVER use `cd /d` — that's CMD syntax, invalid in bash
- NEVER use `cd` — use absolute paths in tool calls
- For git: use `git -C "<absolute project path>"` instead of cd'ing first
- For other tools: pass full paths as arguments
- Windows paths with backslashes work in Git Bash when quoted

## Project Reference

### Game install
- **Steam path:** `C:\Program Files (x86)\Steam\steamapps\common\swkotor`
- **Executable:** `swkotor.exe`
- **Config:** `swkotor.ini` (in install root; `[Alias]` section defines folder mapping)
- **Engine:** BioWare Odyssey Engine (Aurora Engine derivative, also used in NWN)

### Project workspace

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JeanStiletto/voice-of-the-old-republic](https://github.com/JeanStiletto/voice-of-the-old-republic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
