---
trigger: always_on
description: Guidance for AI coding sessions. [VISION.md](VISION.md) = product north star · [README.md](README.md) = short pitch.
---

# CLAUDE.md — Accordion

Guidance for AI coding sessions. [VISION.md](VISION.md) = product north star · [README.md](README.md) = short pitch.

## Key URLs

- **Marketing site:** https://get-accordion.dev/
- **Public repo:** https://github.com/a-Fig/Accordion
- **Private repo:** https://github.com/a-Fig/accordion-private

## Terminology

- **pi** — the CLI AI coding harness whose context window Accordion visualizes. Not an Accordion product; it's the tool the user runs. `extension/accordion.ts` is a pi plugin that hooks into pi's `context` hook (fires before each model call).
- **block** — atomic unit of context: one chunk of a single kind (`user`, `text`, `thinking`, `tool_call`, or `tool_result`). See `engine/types.ts → Block`.
- **turn** — one user message plus all assistant content (thinking, text, tool calls, tool results) that follows it before the next user message.
- **fold / folding** — replacing a block's content in-place with something shorter, like a summary; the block stays on the wire to the LLM in compressed form. Always reversible.
- **held** — a block carrying a human override (manual pin, fold, or unfold). `ViewBlock.held = true`; the host refuses conductor commands on held blocks unless the conductor holds a `human-steering` involvement lock.
- **conductor** — a pluggable context-management strategy (`conduct(view) → Command[]`). Decides which blocks to fold, group, replace, pin, etc. between turns.
- **the wire** — the messages array sent to the LLM provider. "Wire-valid" = the outgoing array is well-formed. Distinct from the WebSocket between the app and the pi extension (that's the live link / accordion protocol).
- **browser-served** — mode where the pi extension HTTP-serves the SvelteKit UI on the same ephemeral port as the WS. Single-session; no Tauri desktop app required.
- **CC** — Claude Code (as in "CC transcript", "CC browsing"). Read-only mode; sessions loaded from `~/.claude/projects/`.

## Codebase map

| path | what |
|------|------|
| `app/` | Tauri 2 + SvelteKit desktop app — the active surface |
| `app/src/lib/engine/` | The model: types, parser, store — single source of truth |
| `app/src/lib/live/` | WS client, session discovery, CC transcript browsing |
| `app/src-tauri/src/lib.rs` | Native Rust: session discovery + `~/.claude` reads |
| `extension/accordion.ts` | Live pi extension — WS server + HTTP server (browser-served mode) |
| `conductors/` | All context strategies — see [conductors/README.md](conductors/README.md) |
| `conductors/contract/` | Shared conductor contract (dependency-free) |
| `docs/` | ADRs + developer references |
| `brand/accordion-brand-kit/brand.md` | Brand colors + typography source of truth |

**App structure.** One route (`routes/+page.svelte`), the **Map** shell: `SessionsSidebar` (source switcher — live pi sessions or read-only Claude Code transcripts) + `MapHeader` (composition strip + budget) + `ContextMap` + `Inspector`. `ContextMap` has a 2-way toggle: **Map** (uniform dice-square grid) | **Transcript** (scrollable full-chat; blocks as cards, kind-colored left spine; live blocks show full text, folded blocks show the exact `{#code FOLDED}` digest the agent sees; double-click to fold, single click = inspect). Arrow keys traverse blocks (←/→ = prev/next, ↑/↓ = ±one row).

## Engine — single source of truth

`app/src/lib/engine/` owns the model. **The UI only renders and calls its actions — never reach around it.**

- `types.ts` — `Block { id, kind, turn, order, text, tokens, toolName?, callId?, override, autoFolded, by }`. Kinds: `user · text · thinking · tool_call · tool_result`
- `parse.ts` — pi / Claude Code JSONL → typed blocks. `tool_call` and `tool_result` are separate blocks sharing a `callId`. An assistant message's thinking/text/call blocks share an `id` prefix before `:`
- `store.svelte.ts` — `AccordionStore` (Svelte runes); exposed as `window.__store`. `appendBlocks(blocks)` is the streaming seam used by the live link to add new blocks. **Protected working tail** (`protectTokens`, default `20_000`): `protectedFromIndex` marks the first block in the tail; both auto- and manual-`fold()` are refused inside it; a block that was auto-folded before entering the tail heals back to live; `pin()` remains allowed. `setProtect(n)` resizes and re-folds, wired to an on-bar draggable handle. Under the `tail-size` involvement lock (ADR 0011), the tail floor is lifted — the conductor may fold any block
- `tokens.ts` — chars/4 estimate · `digest.ts` — what a kind collapses to when folded

**Folding is content substitution, never removal** — provider-safe and fully reversible.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a-Fig/Accordion](https://github.com/a-Fig/Accordion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
