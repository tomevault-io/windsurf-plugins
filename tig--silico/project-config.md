---
trigger: always_on
description: Canonical guidance for AI coding agents (Claude Code, Grok Build, Copilot, Codex, and kin) working in **silico** or scaffolding a **GCU** that depends on it.
---

# AGENTS.md

Canonical guidance for AI coding agents (Claude Code, Grok Build, Copilot, Codex, and kin) working in **silico** or scaffolding a **GCU** that depends on it.

Human overview: [README.md](README.md). Tenets: [specs/tenets.md](specs/tenets.md). Phrase book: [specs/lexicon.md](specs/lexicon.md). first-ship narrative: [specs/prfaq.md](specs/prfaq.md) FAQ 5. Build target: [specs/silicov1.md](specs/silicov1.md).

## FIRST ACTION — Stage 0 (first ship / “getting started for agents”)

When the human points at silico / first ship / “getting started for agents”, do **this exact order**.

### 0 — Load this file (full text, not a fetch digest)

**You are not following silico until you have the full text of this `AGENTS.md`** (local silico clone, GCU plate after scaffold, or download `https://raw.githubusercontent.com/tig/silico/main/AGENTS.md` and **read the file**).

| Fail | Correct |
|------|---------|
| WebFetch `github.com/tig/silico` (README/homepage) and start from the digest | Open **this file** (or the raw URL), then Stage 0 tools |
| **WebFetch / “fetch summarizer” of AGENTS.md** — act on a garbled or truncated summary | Prefer local path, `curl`/`Invoke-WebRequest` to a temp file + Read, or a tool that returns **full** body. If a summarizer garbled it: **re-download and read full text before any Stage 0 claim** |
| Treat a README one-liner about Stage 0 as the full contract | README is human entry; **this file is Priority 1** for agents |

A second-hand README or **lossy fetch summary** is luck, not diligence — even if it mentions `silico welcome`.

### 1 — Stage 0 tools (after this file is open in full)

**Do not** open with tooling narration, COM folklore, PR strategy, `bedside init`, vendoring `third_party/`, scaffold, or a start-gate chooser.

```text
# --- TURN 1 (0a only): run tool, paste orientation, name the human act, END TURN ---
silico welcome
# Paste/adapt that output as the ONLY operator-facing content of this turn.
# End with the operator next-step line from welcome (reply ok/go when ready).
# Do NOT call bedside ask / host AskUserQuestion in this same turn (modal hides 0a).
# Do NOT leave a free-text cliff ("Start gate is next") with no human act named.

# --- TURN 2 (0b only): FIRST act after any short operator reply (ok / go / yes / …) ---
# Open the structured chooser immediately — do not re-welcome, do not ask free-text
# "shall I open the gate?", do not wait for a second free-text "yes".
bedside ask --id start-first-ship --prompt "Start first ship on this machine?" --choices yes,adjust --default yes
# or host structured picker with the SAME id / prompt / choices (yes, adjust only).
# Say once if using the host gate shell.
```

| Rule | Detail |
|------|--------|
| **Load full AGENTS first** | README/homepage alone fails. **Fetch digests of this file fail.** Full text or re-download. |
| **Two turns: 0a then 0b** | **Hard rule.** Orientation is turn 1. Start gate is the **first** act of turn 2. Same-turn “welcome then picker” fails when the host UI steals focus. |
| **0a ends with a human act** | Operator-facing close must say what to do next (e.g. reply `ok` / `go`). Never leave only “Start gate is next” at a free-text prompt with no chooser and no next step. |
| **Turn 2 = chooser first** | After any short reply, open the **structured** start gate immediately. Free-text-only turn 2 fails Stage 0b (Bedside: never leave them at a cliff; no free-text choice wall). |
| **0a is a completed chat message** | Tool log only fails 0a. 0a must be a finished assistant message the operator can scroll to **without** a modal open. |
| **Welcome is the skeleton** | Do not hand-build a multi-section status report instead of `silico welcome`. |
| **Gate prompt is short** | Title/prompt = one question. **No** Stage A–D monologue in the chooser. Optional: “Orientation is in the message above.” |
| **Choices only `yes` / `adjust`** | Do **not** invent `Go`, `Host-only`, `Look around`. Host-only is a named anti-pattern unless the operator **adjusts** after a short re-gate. |
| **No mutate before go** | Before **yes** on start-first-ship: no `bedside init`, vendor, scaffold, commit/push, product-tree installs. |
| **Doctor fail ≠ skip 0a** | Missing GCU `bedside.toml` is normal. Still show 0a; fix pin after go via plate. |
| **Host picker OK** | Same id/prompt/choices; say once. Still **turn 2 only**. Decline / exit 10 = halt writes. |

**Harness note (Claude Code / similar):** Host structured questions take focus immediately → **split turns**. After 0a, the session **must** resume into a **chooser**, not a bare free-text prompt. If the operator only saw free text after orientation, you failed 0b — open the start-gate picker on the next opportunity without redoing a long 0a.

After **go**: Stage A tools → B workspace → C plate + host gate → D metal. Full stages and metal rules are below.

---

## Agent context load path (read once)

Context is finite. **Do not** open every manners file into the active window.

| Priority | Open | Job | Skip when |
|----------|------|-----|-----------|
| 1 | This file (`AGENTS.md`) | Silico spine: first-ship stages, silico CLI, plate, host/metal DoD | — |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tig/silico](https://github.com/tig/silico) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
