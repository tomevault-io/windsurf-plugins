---
trigger: always_on
description: This file is the style and structure reference for writing new labs in this series. Read it before drafting anything. The goal is consistency — every lesson should feel like it was written by the same person, in the same session.
---

# Writing Guide: Week 1 PM Agent Labs

This file is the style and structure reference for writing new labs in this series. Read it before drafting anything. The goal is consistency — every lesson should feel like it was written by the same person, in the same session.

---

## Who We're Writing For

Product managers and operators who are new to building with AI tools. They're smart. They're not developers. They've never touched n8n, Claude Code, or an API. They're skeptical of labs that feel like tutorials — they want to build something real and understand *why* it works, not just follow steps.

Write for someone who's done with passive learning and wants to produce something they can show to their team by the end of the session.

---

## The Voice

**Direct. Confident. No hedging.**

Short sentences punch. Then a longer one explains. Then move on.

Never use:
- "Simply" or "easily" — if it were simple, they wouldn't need the lab
- "Let's go ahead and..." — just say what to do
- Passive voice: "The file is uploaded by the user" → "The user uploads the file"
- Filler phrases: "In this section we will...", "Now that we've covered..."

Do use:
- Em dashes for rhythm — like this — mid-sentence
- Sentence fragments for emphasis. On purpose. When it lands.
- Rhetorical setups: "Here's the thing.", "But here's what's different."
- Second person throughout: "you", "your agent", "your web app"
- Italics for moments of insight: *Claude didn't become smarter. It became aligned with how you work.*

**Honest about friction.** If something will feel weird, say so before they hit it. "You'll likely see an error. That's completely normal — and it's actually a good sign." This builds trust more than pretending everything is seamless.

---

## The Structure of Every Lab

Every lab follows this shape. Don't skip sections.

```
1. Title + Banner image
2. Opening hook — 2–4 sentences
3. "By the end of this lab, you will..." — bullet list (use ✦ or prose)
4. Before You Start / Prerequisites — ✅ checklist
5. Phases or Parts — numbered, named
6. Steps within each phase — ### headers
7. Testing section
8. What You Built / What You Learned
9. What's Next — one short paragraph
10. Navigation links (← back | → forward)
```

Separate every major section with `---`.

---

## How to Write a Step

Every step has three layers:

**1. The instruction** — what to do, written imperatively. No "you can" or "you might want to." Just: "Click X. Set Y to Z. Add this field."

**2. A screenshot or image reference** — always `![flow](./assets/X.png)` after the instruction. If there's no asset yet, leave a placeholder comment.

**3. A blockquote with the WHY** — this is the most important layer. Use `>` for it. Every step that's non-obvious needs one. Answer the question the reader is silently asking.

Format for why-callouts:
- `> **Why X?** ...` for a single-concept explanation
- `> ✓ Tip. ...` for a practical shortcut or warning
- `> ★ ...` for a high-importance note (API keys, billing, irreversible actions)
- `> ...` for broader context that connects the step to the bigger picture

Example of a complete step:

```markdown
### Add the Extract From File Node

Click **"Add Node"**, search for **Extract From File**, and select it. Choose **"Extract from PDF"** as the action.

![flow](./assets/6.png)

> **Why this node?** When a user uploads a PDF, the file arrives as binary data — raw bytes the AI can't read. This node is the key that opens it. It pulls out the readable text so the AI can reason over the contract. Without this step, the agent would receive bytes it can't understand, and the workflow would fail silently.
```

---

## Introducing Concepts

When a technical concept appears for the first time, introduce it with this pattern:

1. **Analogy first** — something physical and familiar. Doorbell. Phone number. Assembly line. Sealed envelope. Locked box.
2. **One-sentence definition** — what it actually is, technically
3. **Why it matters here** — connect it to what the reader is building right now

Example (webhook):
> Imagine someone comes to your door to deliver a package. They don't just walk in — they ring the doorbell. That ring is the signal: "Someone is here. They have something for you."
>
> A webhook works exactly like that. It's a URL — a specific address on the internet — that your agent publishes and listens to. When your web app calls that address, the agent wakes up, receives what was sent, and sends an answer back.

Never define a concept by restating its name. "A webhook is a web hook that hooks into the web" is useless. The analogy always comes first.

---

## Tables

Use tables for:
- Comparing two approaches (Path A vs Path B)
- Showing system message vs user message differences
- Listing what two paired nodes do (Webhook / Respond to Webhook)
- Comparing three versions of a tool (Claude Chat / Cowork / Code)

Keep table headers short. Every cell should be one sentence or less. Don't use tables to explain — use them to compare or summarize what's already been explained.

---

## Code Blocks

Use fenced code blocks (triple backtick) for:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [initmahesh/MLAI-community-labs](https://github.com/initmahesh/MLAI-community-labs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
