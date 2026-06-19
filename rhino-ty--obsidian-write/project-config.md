---
trigger: always_on
description: >
---


# obsidian-write — Obsidian Vault Writing Convention SSoT

> A single authoritative document for repeatedly-applied decisions when writing Obsidian notes: heading depth, emphasis-breakage rule, 5-axis tags, sticker frontmatter, PARA classification. Other writing skills delegate to this file as the convention authority via a vault signal.

## Quick Map

| Scenario | What happens |
|---|---|
| User writes a new `.md` directly in the vault | This skill's full conventions apply + post-write grep self-check |
| User edits an existing `.md` (heading / emphasis / tag changes) | Conventions applied to the changed region + verification of touched area |
| Another skill (polymedia / knou-note-writer / etc.) triggers in the vault | That skill owns its workflow (interview, lecture analysis); *format conventions* of the produced note are delegated here |
| Outside the vault (plain markdown / Logseq / Notion / Bear) | This skill is inactive — host skill applies its own generic-markdown case |

## Core vs Recommended vs Optional

| Tier | Items | Apply when |
|---|---|---|
| **Core** (all Obsidian users) | Heading (§3) · Horizontal rule (§4) · Indentation (§5) · Emphasis-breakage rule (§6) | Always in Obsidian |
| **Recommended** (CJK authors) | CJK-specific emphasis-breakage procedure (§10 + `ref/cjk-language-extra-checks.md`) | Note is written in Korean, Japanese, Chinese, or mixed CJK + Latin |
| **Recommended** (Make.md or folder emoji users) | Sticker frontmatter (§2) | Make.md plugin or folder-emoji management is in use |
| **Optional** (your operational policy) | Writing style §1 · 5-axis tags §7 · PARA §8 · Two-tier emphasis hierarchy (§6 Optional) | Your vault adopts these specific patterns |

Adopting only the four core items already eliminates almost all Obsidian rendering breakage. The rest is *opinionated* — patterns one user validated in their own vault. Take them if they fit; ignore if not.

---

## Output Language Policy

This skill is convention infrastructure, not a content generator. The convention text in this file is in English so any audience can read it, but **the notes you produce follow the user's language**.

- User writes in Korean → produce Korean notes. CJK emphasis-breakage procedure (§10) kicks in.
- User writes in English → produce English notes. CJK procedure is skipped (or run as a sanity pass anyway, since it's cheap).
- User writes in Japanese / Chinese → same as Korean. CJK procedure applies.
- User mixes languages → the dominant language drives §10; mixed segments run the relevant checks per language.

**Never translate the user's content into English just because this skill is in English.** Conventions are language-agnostic; voice and substance stay in the user's language.

---

## 1. Writing Style (Decision Logging)

1. **Record the why, not just the what** — conclusions decay without their reasoning. Six months later you should still see *why* you chose this path.
2. **ADR-style structure** — Context → Decision → Rationale → Consequences. Use this for any non-trivial decision recorded in a note.
3. **TODO markers** — Mark unresolved items with `TODO:` so they grep cleanly.
4. **Preserve deprecated decisions** — Use ~~strikethrough~~ rather than deleting, so the historical context survives.
5. **Tables for comparative analysis** — Markdown tables for trade-off matrices, not prose lists.
6. **Inline personal commentary** — Drop your own thinking, doubts, half-formed reactions into the note. Notes are for you, not for an audience.

> A note that only states the final decision is a dead artifact. A note that captures the deliberation is a living tool.

---

## 2. Frontmatter (Sticker / Folder Emoji)

**Note emoji**: Don't put emoji in the filename. Put it in frontmatter `sticker` using Make.md compatible format:

```yaml
---
sticker: emoji//{unicode-codepoint(lowercase hex)}
---
```

Examples:
- `sticker: emoji//1f48a` → 💊
- `sticker: emoji//1f34a` → 🍊
- `sticker: emoji//1f4d6` → 📖 (book)
- `sticker: emoji//1f3ae` → 🎮 (game)
- `sticker: emoji//1f3ac` → 🎬 (movie)
- `sticker: emoji//1f3b5` → 🎵 (music)
- `sticker: emoji//1f9e0` → 🧠 (thinking / essay)
- `sticker: emoji//1f4d8` → 📘 (academic note)

**Folder emoji**: Don't put emoji in the folder name (sync / cross-platform breakage). Use the `.md` file that shares the folder's name (a "folder spec note") and put sticker in its frontmatter:

```yaml
---
_filters: []
sticker: emoji//1f48a
color: ""
---
```

The folder spec note doubles as a place to record what the folder is *for* — directory README-style context.

---

## 3. Heading Rules

Obsidian's *Inline Title* renders the filename as h1. Using h1 in the body produces a duplicate title. Therefore:

- Body starts at **h2 (`##`)**
- Body ends at **h4 (`####`)** maximum
- `h1` (`#`) is **forbidden in the body**

```markdown
##(h2)  — main section
###(h3) — subsection
####(h4) — detail
```

Deeper hierarchy: use `**bold labels**` inside the body. Don't create h5 / h6.

**No numbered headers**: `## 1. Topic` ❌ → `## Topic` ✅. Outline view gets cluttered with numbers and reordering breaks them.

> This diverges from the CommonMark MD041 lint, but Obsidian is the deployment target — in-vault readability beats portability here.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhino-ty/obsidian-write](https://github.com/rhino-ty/obsidian-write) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
