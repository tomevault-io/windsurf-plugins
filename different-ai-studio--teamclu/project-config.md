---
trigger: always_on
description: This file is the source-of-truth for the **web / desktop** visual language
---

# AGENTS.md — UI / Visual Design Spec (Web / Desktop)

This file is the source-of-truth for the **web / desktop** visual language
(`packages/app/`). For the **iOS client** (`apps/ios/`) the source-of-truth is
`apps/ios/DESIGN.md` (the Hai 灰 / wabi-sabi system) — don't apply the rules
below to iOS work. For repo conventions (architecture, commands, release
process) see `CLAUDE.md`. If `docs/SDLC.md` exists in this checkout, read it
before starting changes; it is a local, git-ignored SDLC override for this
workspace.

The web/desktop design direction is **"Editorial Calm"** — paper-feel
neutrals, brand coral used only as small accents, Chinese-first typography,
higher information density than a typical chat app but with breathing room
inside each card.

It came from the Claude Design handoff at
`https://api.anthropic.com/v1/design/h/OLWqffBkDMYRHp_p7cFRNg` (Direction B).
The local prototype copy lives in `/tmp/design-OLWqff/` when fetched.

> **Git note:** Work on a task-scoped branch, never on `main`, and never push
> to `main` directly. See `CLAUDE.md` → Git Workflow for the full rule. If
> `docs/SDLC.md` exists in this checkout, it may add personal worktree /
> preview-integration conventions on top.
>
> **PR rule:** Do **not** push the branch or run `gh pr create` on your own
> initiative. Until the user explicitly says "open the PR" / "ship it" /
> "提 PR", stop after committing and report back — even if everything looks
> "done", tests pass, and the user previously approved the diff.

---

## 1. Design tokens

All tokens live in `packages/app/src/styles/globals.css` and are exposed to
Tailwind via `@theme inline`. Use the token, not a hardcoded color.

### Color palette (light — primary theme)

| Token              | Value                       | Use                                                          |
| ------------------ | --------------------------- | ------------------------------------------------------------ |
| `--background`     | `#fbfaf7`                   | App background / chat pane                                   |
| `--paper`          | `#ffffff`                   | Cards, message surfaces, popovers                            |
| `--panel`          | `#efece4`                   | Sidebar / sub-panel surface                                  |
| `--selected`       | `#e7e2d6`                   | Selected row in panel sections                               |
| `--foreground`     | `#1a1a14`                   | Primary ink                                                  |
| `--ink-2`          | `#3d3c34`                   | Secondary ink (body copy)                                    |
| `--muted-foreground` | `#75736a`                 | Tertiary / hint text                                         |
| `--faint`          | `#a8a6a0`                   | Quiet ink — timestamps, meta, divider labels                 |
| `--border`         | `rgba(26,26,20,0.08)`       | Standard line                                                |
| `--border-soft`    | `rgba(26,26,20,0.05)`       | Internal card divider, dashed dividers                       |
| `--coral`          | `#e85a4a`                   | **Brand accent — used sparingly**                            |
| `--coral-soft`     | `#f5d6cf`                   | Coral on coral, e.g. permission popover background           |

Dark mode keeps the existing oklch values; only coral and font tokens are
extended there. Light is the canonical theme.

### Where coral is allowed

Coral is the brand accent. Use it for **at most 2 spots in any frame**.
Approved locations:

- Active session left bar (2px wide)
- Unread / new-message badge background
- Primary send button (chat input)
- Small "AI" pill border, when used in a row of mixed actors
- AI-avatar ring + tiny indicator dot (lobster mark on agent rows)
- Permission popover border / pill (when re-introduced)

If you find yourself reaching for coral for anything else (success state,
focus ring, link, hover), use ink/muted/border instead.

### Typography

```
--font-sans: "PingFang SC", "Noto Sans SC", "Source Han Sans SC",
             -apple-system, BlinkMacSystemFont, "Microsoft YaHei",
             system-ui, sans-serif
--font-mono: "JetBrains Mono", "SF Mono", ui-monospace, Menlo,
             Consolas, monospace
```

Chinese-first. Latin glyphs fall back to system. **Use mono for:**
timestamps, tool-call argument strings, model identifiers, version
numbers, keyboard hint pills (`⌘↵`, `↵`, `esc`).

### Type scale

| Size                 | Use                                                |
| -------------------- | -------------------------------------------------- |
| `text-[15px]` (700)  | Section title (e.g. "会话")                        |
| `text-[13.5px]`      | Message body, chat bubble text                     |
| `text-[13px]` (600)  | Card title (conversation row, agent name)          |
| `text-[12.5px]`      | Secondary body, agent reply text                   |
| `text-[12px]`        | Card preview, meta line                            |
| `text-[11.5px]`      | Footer captions                                    |
| `text-[11px]` (mono) | Timestamps, version, "⌘↵"                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [different-ai-studio/teamclu](https://github.com/different-ai-studio/teamclu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
