---
trigger: always_on
description: > **You are an AI coding agent reading this file because your user asked you to install
---

# CodexCont — Installation Runbook for an AI Agent

> **You are an AI coding agent reading this file because your user asked you to install
> CodexCont for them.** This is not documentation to summarize — it is a runbook to
> *execute*, step by step, on the user's machine. Follow it top to bottom.

CodexCont is a small local proxy that sits between a coding agent (Codex, Pi, …) and an
upstream OpenAI **Responses**-compatible endpoint. It detects a reasoning-truncation
fingerprint and silently asks the model to keep thinking, folding several upstream rounds
into one clean downstream response. The user runs it locally; their coding agent is then
pointed at it instead of at the real upstream.

---

## §0 — How to use this runbook (read before doing anything)

1. **Execute in order.** Each section has concrete commands and a **Verify** check. Do not
   advance until the current section's Verify passes. If a Verify fails, go to **§9
   Troubleshoot** before continuing.
2. **Speak the user's language.** This file is in English so you parse it reliably, but
   **all of your messages to the user must be in the language they are speaking to you.**
   Translate prompts, warnings, and questions as needed.
3. **🛑 markers are hard stops.** A 🛑 step requires the user. **Never fabricate
   credentials, never skip the disclaimer, never modify an agent config without first
   backing it up.** Ask, wait for the answer, then proceed.
4. **Detect the OS first** and use the matching command variant throughout:
   ```bash
   uname -s    # Linux / Darwin (macOS); on Windows you are usually in Git Bash or PowerShell
   ```
   - Windows venv interpreter: `.venv/Scripts/python.exe`
   - macOS / Linux venv interpreter: `.venv/bin/python`
5. **Idempotent.** It is safe to re-run any step. Check before you create; don't clobber.
6. **Repo root.** Run repo commands from the directory that contains `run.py`,
   `pyproject.toml`, and `config.example.toml`. Confirm with `ls`.

---

## §1 — 🛑 Tell the user two things, get consent

Before touching anything, say the following to the user **in their language** and wait for
an explicit "yes":

**(a) Disclaimer.** CodexCont *explicitly bypasses* the observed OpenAI Codex
reasoning-truncation behavior. If this use is considered abusive, violates service terms,
increases costs unexpectedly, or causes any other adverse consequence, **the user is solely
responsible.** They must accept this before you continue.

**(b) What you are about to explore.** To configure this correctly you will **look around
their operating system**: locate which coding agents they have (primarily **Codex** and
**Pi**) and **read those agents' config files** (e.g. `~/.codex/config.toml`,
`~/.pi/agent/models.json`) so you can wire them up and back them up first. Tell them this
plainly and get their OK before reading anything under their home directory.

> If the user declines either point, **stop here.**

---

## §2 — Preflight

Confirm the environment. Python **3.12+** is required.

```bash
python --version    # or: python3 --version  / py --version  (Windows)
```

You will offer the user **two installation methods** in §4. Detect what is available now so
you can recommend one:

```bash
uv --version        # present? -> Method A (recommended)
python -m pip --version
```

**Verify:** Python ≥ 3.12 is available, and you are in the repo root (`ls` shows `run.py`,
`pyproject.toml`, `config.example.toml`). If Python is too old, stop and ask the user to
install Python 3.12+ (or point you at an existing one).

---

## §3 — 🛑 Explore the user's agents, then interview them

### 3.1 Explore (read-only)

Find the user's coding agents and how they reach their model. Do **not** modify anything
yet. Likely locations (adapt to the detected OS):

| Agent  | Config to read | What to determine |
|--------|----------------|-------------------|
| Codex  | `~/.codex/config.toml`, `~/.codex/auth.json` | Is the user on **official ChatGPT OAuth login** (`auth.json` holds OAuth tokens, `codex login` was used) **or a generic Responses API** (a `[model_providers.*]` with a third-party `base_url` + API key)? Note the current `model` and `model_provider`. |
| Pi     | `~/.pi/agent/models.json`, `~/.pi/agent/settings.json` | Which providers exist, their `baseUrl`, and crucially each provider's `api` field (`openai-responses` vs `openai-completions`). |

> ⚠️ These files often contain **live secrets** (API keys, OAuth JWTs). Read them only to
> understand structure. **Never copy secret values into this repo, into chat, into memory,
> or into any file you commit.** When you show config to the user, use placeholders.

### 3.2 Report and ask (🛑)

Summarize for the user, in their language: which agents you found, and for each, how it is
currently reaching its model. Then ask:

1. **Which agent(s)** should be pointed at CodexCont? (Codex / Pi / both)
2. **Auth mode** for the proxy (this maps to `config.toml [auth].mode`):
   - The user's agent is logged in and already sends its own auth → **`passthrough`** (proxy
     forwards the caller's auth, injects nothing). This is the default and the common case.
   - The user wants the proxy itself to hold and inject a token → **`inject`**.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neteroster/CodexCont](https://github.com/neteroster/CodexCont) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
