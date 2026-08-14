---
trigger: always_on
description: 1. `memory.md`, starting with the CURRENT STATE block at the very top, then the top session entries (the running log: what we did, learned, decided, and what is still open).
---

# Workspace: ocho-frames

## Read these first, every session, before doing anything else

1. `memory.md`, starting with the CURRENT STATE block at the very top, then the top session entries (the running log: what we did, learned, decided, and what is still open).
2. The active client's `brand.md`, currently `clients/sportif/brand.md`, and `clients/sportif/voice-guidelines.md` before writing any content.
3. Any status or resume note in the active client's folder, e.g. `clients/sportif/intake/RESEARCH-RUN-STATUS.md`.

Doing this is what gives a new session continuity. Do not start work cold.

## What this workspace is

Hugo's (Ocho's) personal creative and marketing workspace. Two tracks:

1. A creative-strategy pipeline: competitor analysis, then a synthesis brief, then AI-generated production media. See `docs/pipeline-architecture.md` and `docs/marketing-fundamentals.md`.
2. Client work under `clients/`.

It is also a HyperFrames video workspace (video as code). See `README.md`.

## Active work

- Client: **Sportif** (founder Lucy Wayne). Affordable-luxury fitnesswear accessories, Australian, launching September 2026. All context lives in `clients/sportif/`. Launch products are accessories (booty bands + vegan ankle strap), not apparel.

## Two environments, one workspace (sync protocol)

Hugo works on this folder from TWO places, often alternating within the same day:

1. **Claude Code** (terminal / VS Code on his Mac): full shell, background processes survive, all local fonts, direct file deletion.
2. **Cowork** (Claude desktop app): sandboxed Linux shell, see the Cowork-specific gotchas below.

Which one am I? If shell paths look like `/sessions/<name>/mnt/hyperframes/`, this is Cowork. If they look like `/Users/hugobrizuela/...`, this is Claude Code.

**The handoff protocol (both environments, no exceptions):**

1. **Session start:** read the CURRENT STATE block, then `git log --oneline -5` to see what the other environment did last. If the working tree is dirty with changes you did not make, a session somewhere did not close out; commit or flag before working. To orient fast: `python3 scripts/memory_tools.py open --client <client>` (open loops) and `... search "<term>"` (find past context).
2. **Session end (the close-out ritual):** (a) update the CURRENT STATE block (including the handoff line); (b) add a session entry to the top of `memory.md` tagged with the environment AND with `Client:` + `Tags:` lines, e.g. `## Session 026 (2026-07-24, Claude Code): ...`; (c) record any new settled decisions in `DECISIONS.md` and any new/resolved open loops in `OPEN-QUESTIONS.md` (schemas are in those files); (d) run `python3 scripts/archive_memory.py` (no-op unless memory.md crossed 90KB) and `python3 scripts/memory_tools.py index` (regenerates `memory-index.md`); (e) run `python3 scripts/memory_tools.py check` (must print OK) and optionally `... reconcile`; (f) `git add -A && git commit`. The commit is what makes the work visible to the other side. (A pre-push hook runs `check` as a warn-only reminder; `MEMORY_ENFORCE=1` makes it block.)
3. **Session numbers are continuous across both environments.** Check the top of memory.md for the last number before starting a new entry.
4. **Do not run both environments on the same files at the same time.** If both are open, one is the builder and the other is the sounding board; the sounding board reads but does not write.
5. **Environment-specific learnings** go in "Tools and gotchas" below, labelled with which environment they apply to.

## Conventions (follow these)

- **Log every session.** At the end of any session with real work, add an entry to the TOP of `memory.md` (date, what we did, learned, decided, open). Match the existing format, and include a `Client:` line (client name, or `Ochoproductions` for workspace-wide) and a `Tags:` line. Mirror settled decisions into `DECISIONS.md` and open loops into `OPEN-QUESTIONS.md` (extractable registries, filterable by client, queried via `scripts/memory_tools.py`). The full memory system is documented in `docs/memory-system.md`.
- **Memory scales by client.** One unified chronological log (keeps cross-client learnings), but everything is tagged and filterable per client. When a second client goes active, split the CURRENT STATE block into per-client mini-blocks under a shared workspace header.
- **Voice rule: no em dashes and no en dashes** anywhere in written output or files. Use commas, periods, or parentheses instead.
- **Secrets:** API keys go in `.env` only (gitignored). Never put a real key in `.env.example`, never commit secrets.
- **Work with Hugo:** ask for project context before assuming, and flag trade-offs rather than defaulting to one approach. Background on how he works is in the `hugo-working-style` skill.
- **Generated media** goes in `clients/<client>/generated/images/` or `generated/videos/`, and every keeper's prompt is saved to the client's `image-prompts.md` (the prompt is the source of truth, binaries are gitignored). Iterate at quality low in Cowork (45s cap), render finals in Claude Code.

## Tools and gotchas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OchoOcho88/ocho-frames](https://github.com/OchoOcho88/ocho-frames) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
