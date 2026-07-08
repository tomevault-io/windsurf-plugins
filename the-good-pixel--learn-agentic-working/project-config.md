---
trigger: always_on
description: You are helping write **Learn Agentic Working** — an open-source book that teaches teams how to use AI coding agents (Claude Code, Codex, OpenCode, etc.) to do real work, for both technical and non-technical audiences.
---

# CLAUDE.md — Instructions for agents working on this book

You are helping write **Learn Agentic Working** — an open-source book that teaches teams how to use AI coding agents (Claude Code, Codex, OpenCode, etc.) to do real work, for both technical and non-technical audiences.

Read `README.md` first for the full vision, structure, and audience. This file is the *style and working* guide for any agent (you) helping write or edit the book.

---

## The reader

Default to writing for **two audiences side-by-side**:

1. **The technical reader** — engineer, data analyst, devops. Comfortable with CLI, git, JSON.
2. **The non-technical reader** — PM, marketer, ops, designer, founder. Comfortable with Notion, Google Sheets, Slack. Has chatted with ChatGPT or Gemini before, but has never used a CLI tool or an "agent".

**Rules of thumb:**
- Assume they know what an LLM is. Don't assume they know what an "agent", "MCP", "tool call", "skill", or "hook" is — introduce each on first use.
- When something is CLI-only, say so plainly, and tell the non-technical reader "you can skim this section, the idea matters more than the syntax."
- Every concept is taught **with a runnable example**. No abstract definitions without a concrete task next to them.
- Bias toward **"here's a thing you'd actually do at work"** over toy examples.

---

## Tone & voice

- Conversational, second-person ("you"), opinionated but humble.
- Short sentences. Active voice. No corporate hedging.
- It's okay to say "this is the part that always trips people up" or "skip this if you don't care about X."
- No marketing speak. No "unleash the power of." No "in today's fast-paced world."
- No emojis in the book content (the audience is international and many corporate readers find them unserious). Diagrams and screenshots are fine.

---

## Tool-neutrality policy

The book is **Claude-Code-led but not Claude-only**.

- Primary running examples use **Claude Code** because that's what the author uses.
- Every chapter that teaches a Claude-Code feature must include a short **"In other tools"** call-out noting the equivalent in **Codex**, **OpenCode**, **Cursor agent**, and **Gemini CLI** where one exists. If no equivalent exists, say so.
- Never claim a feature is unique to Claude Code without checking. Agent tooling moves fast — when in doubt, hedge ("at the time of writing…").

---

## Sourcing examples — use real history

When you need an illustrative use case, **prefer real, paraphrased examples from the author's actual Claude Code usage** over invented ones. The author's transcripts live at:

```
/Users/darrenchiu/.claude/projects/
```

Each subdirectory corresponds to a working directory; each `.jsonl` file is a session. The **first user message** in a session is usually the cleanest statement of intent.

Good real categories already mined (use these freely, vary which you cite):

- **Engineering**: feature work from a Linear ticket; cross-repo bug hunts ("Malaysia inquiries aren't being forwarded"); memory-leak hunt in a browser extension; compiling an unfamiliar repo on Mac+iOS.
- **DevOps/Infra**: scaffolding a new AliCloud backend; retrieving a UAT DB password from an IaC repo; investigating a prod incident with SQL only, no writes.
- **Data**: "study my TransactionHistory.csv — why does my balance keep dropping?"; folder-diff script for Downloads; spreadsheet-image extraction feasibility.
- **Research/Learning**: "why doesn't this Threads post show a link preview?"; debating GTM strategy with Gemini as a sparring partner; asking a codebase questions instead of changing it.
- **Automation/Tooling**: installing an MCP server; installing a skill from a GitHub PR; sweeping a model-version upgrade across configs.
- **Product/Business**: turning client feedback into code changes; auditing for UX bugs from a user complaint; adding analytics to find what features are actually used.
- **Everyday**: planning mesh-WiFi placement from a floor-plan photo; diagnosing a strange TLS cert error.
- **Bridging engineering and marketing**: generating product demo videos directly from a running app via a Playwright + Remotion pipeline (the `demo-video` skill on `dipping.ai`) — a key example for chapters that need to land for non-technical readers.

### Real skills to reference as worked examples

When a chapter needs a concrete skill, prefer these (paths are the source of truth — **read them before writing about them**). Skills are grouped by what they *teach*, not by what they technically do.

**Workflow & PR shepherding**
- **`ship-pr`** (user-level, `~/.claude/skills/ship-pr/SKILL.md`) — push → open PR → monitor CI → resolve AI review comments → notify when mergeable. The "AI reviewer thread is the audit trail; silent dismissal is forbidden" rule is quotable. Best for: Ch. 25 (engineers), Ch. 16 (long-running work), the book's "agents close loops, not just open them" thesis.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-good-pixel/learn-agentic-working](https://github.com/the-good-pixel/learn-agentic-working) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
