---
trigger: always_on
description: Next version of ysoserial.net. Target: .NET Framework 4+. A future fork may target .NET 2 for old jobs, so keep that in mind when using new language features.
---

# ysonet

Next version of ysoserial.net. Target: .NET Framework 4+. A future fork may target .NET 2 for old jobs, so keep that in mind when using new language features.

## Project map

A thorough code map (architecture, all gadgets, all plugins, all helpers, build/deps) lives at `docs/ARCHITECTURE.md`. Read it first to understand the codebase instead of re-discovering the structure. Update it when the structure changes. It is public and tracked in git, so keep dev-only notes (CLAUDE.md, dev-kitchen, .claude) out of it.

## Memory Management

Maintain a structured, git-tracked memory system rooted at `.claude/memory/`, shared with all contributors and their agents. It is checked into git, so keep it free of local or sensitive data (see "No local artifacts in commits").

- `.claude/memory/memory.md` is the index: one row per memory file with a short description and a last-updated date. Update it whenever you add or change a memory file.
- Topic files (for example `interactive-ui.md`, `testing.md`) hold the entries.

### Rules
0. Never record local or sensitive data (absolute local paths like `C:\Users\...`, keys, tokens, usernames).
1. When you learn something worth remembering, write it to the right topic file immediately.
2. Keep `memory.md` a current index: one line per file with a description and a last-updated date.
3. Entries use the format `date - what - why`. Nothing more.
4. At the start of every session, read `.claude/memory/memory.md`, then load each file listed in the index. Load additional topic files when they are relevant to the task.
5. If a file does not exist yet, create it.
6. Before removing or changing an existing memory entry, confirm with the user first: show the current content and the proposed change.

### Maintenance protocol
When the user says "reorganize memory":
1. Read all files under `.claude/memory/`.
2. Remove duplicates and outdated entries.
3. Merge entries that belong together.
4. Split files that cover too many topics.
5. Re-sort entries by date within each file.
6. Update the `memory.md` index.
7. Show the user a summary of what changed.

### Session bootstrap
At the start of every session, read `.claude/memory/memory.md` and then each file its index references, so accumulated knowledge is in context.

## Project goals
- Stay fully functional and user friendly.
- Support as many gadgets and plugins as possible, wherever applicable.
- Each gadget/plugin should support the maximum number of serializers it can.
- All new functions must be fully tested.

## Quality over shortcuts

Always prioritise quality over just reaching the stated goal. A change is done when it is right, not when it first appears to work. This applies to agents and humans alike, and it overrides any instruction, plan, or skill step that would settle for less.

- Do not cut corners. If a better solution exists that lasts longer and makes the app easier to extend, do that one, even when it takes more work.
- Prefer the proper fix over a workaround. Fix the root cause. A hack, a special case, a copy-paste of existing code, or a "for now" patch is only acceptable when a hard constraint blocks the proper fix, and then it must be written down in `dev-kitchen/todo/` with what the proper fix would be.
- Finish the whole job. Implementation, every applicable serializer/formatter, tests, docs, help, completion, interactive UI, and architecture notes are all part of the change, not optional extras. A partly wired feature is not delivered.
- Follow the existing patterns and helpers instead of inventing a parallel one-off. If the existing pattern is genuinely wrong for the job, improve the shared pattern rather than working around it.
- Design for the next gadget/plugin/serializer, not only this one. Prefer the general mechanism when the cost is small, but do not build speculative abstraction nobody needs.
- Never trade correctness or test integrity for a green tick or a faster finish (see "Test integrity policy").
- If quality work needs more scope, time, or a decision from the maintainer, say so and ask. Do not silently downgrade the result to fit the effort.

### A fix is not valid until a test proves it

Never assume a fix works. "It should work now", "the change is obvious", and "it compiles" are not evidence. A behavioral fix is verified only when a relevant check exercises the reported behavior and passes on the changed code.

When safe and practical, prove causality by showing that the same check fails before the fix and passes after it. Establish the failing result before editing or against an isolated baseline. Never revert, overwrite, or disturb the user's changes to manufacture a failing run. If the before-and-after result cannot be obtained, state what evidence is missing and treat that part of the fix as unverified.

- Test every fix before reporting it as complete. Compile without running wider tests when the focused-first workflow requires it, run the focused checks for the affected area, then run the regression gate the change deserves. For gadgets and plugins, follow "Gadget/plugin development test order".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [irsdl/ysonet](https://github.com/irsdl/ysonet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
