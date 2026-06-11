---
trigger: always_on
description: <!-- second-brain-generator:installer-stub -->
---

<!-- second-brain-generator:installer-stub -->
# 🧠 Second Brain Generator — self-install notice (for Claude)

> **This folder is the LAUNCHER (the generator), not a second brain.** This file is a
> **bootstrap stub**: the installer **creates a separate brain folder** and drops the real
> constitution there (a personalized `CLAUDE.md`). The launcher itself is **never modified**: it
> stays read-only and **reusable** (a single launcher can generate several brains).
>
> **For Claude.** If the user asks you to create / install their second brain (or asks the second
> brain a question while this bootstrap stub is still here), **follow the steps below EXACTLY**. You
> can drive the whole machinery yourself: the installer runs in **non-interactive** mode
> (`--non-interactive`) — so you do NOT need the user to launch it at the keyboard. You gather the
> answers **in chat**, then you call **a single command**.

## Step 1 — Get the launcher (normal clone)

*(Often already done if you are reading this file from inside the folder.)* If you start from a URL:

```bash
git clone --depth 1 <REPO_URL> <launcher-folder>
cd <launcher-folder>
```

The launcher is **reusable** and **never touched** by the install: the installer **creates a fresh
brain folder elsewhere**, copies the files into it, then runs `git init` **inside it** → **no link
back to the launcher, by construction** (no inherited remote to remove). The brain's auto-commit hook
**pushes nothing until the user has wired up THEIR own remote repository** (push is opt-in).

## Step 2 — Ask the questions IN CHAT (grouped)

**Ask the brain name FIRST, on its own.** Then ask the remaining three **as one group**. Why split:
every later option must **reuse the exact name the user just typed** — never show a literal `<name>`
placeholder in the options or the recap. (If you asked all four at once you wouldn't have the name
yet when building the location option, and you'd be stuck with `~/<name>`.)

- **Q1 (alone): brain name** (= name of the folder to create). **Do NOT propose a default or suggested
  name** — the user provides it; just capture exactly what they type.
- **Q2–Q4 (grouped): location** (parent folder; default: the user's home → `~/<name>`), **user's
  name**, **default language for the notes**. In every Q2–Q4 label, **substitute the exact name the
  user typed at Q1** in place of `<name>` (e.g. if they typed `acme-notes`, the location option reads
  `Home (~/acme-notes)`, never the literal `Home (~/<name>)`). That token is illustrative only —
  **never suggest a name yourself**.

> 🚫 **For the location: NEVER offer the current working directory, the launcher folder, or any
> temp directory as an option** — and **especially not as the first/default option**. The brain must
> be created **outside** the launcher (the launcher stays read-only and reusable). Do **NOT** derive
> a location from `pwd` / the directory you happen to be running in (e.g. `~/tmp…`): nesting a brain
> in or beside the launcher's working dir breaks rooting and invites accidental loss when a temp dir
> is cleaned. When you present location choices, the **only** standing option is **Home (`~/<name>`, the default —
> with the name they typed at Q1 substituted in)** plus a free-text **"Other"** for the user to type their own path. If they type a
> path that is the launcher folder, the current working dir, or a temp dir, push back before running.

> 🎯 **Install is always generic — no "profile" to choose.** Do NOT offer ANY preset or persona
> (especially not a fake "generic install vs. Head of Engineering" choice), and do **not** ask for
> the user's "context". The constitution is generated neutral; it's the user who will tailor it
> afterwards. The personas mentioned in the README (Head of Engineering, PM, consultant…) are
> **usage examples**, not install options.

> ⚠️ **Do NOT ask for the Gemini key.** It **never** travels through the chat or the command line
> (it will go straight into `.env`, see step 4) — **and it is only even useful if the user picks the
> "Gemini API key" option below.**

### 2.bis — Choosing the embedding engine (THE privacy choice, to be presented)

This is a **genuine user trade-off** (decision D1, ADR 0007): present it **clearly, in plain
language**, then pass the result via `--embedder` in step 3. Present **3 options** (from most private
to least) and **recommend based on the machine**:

- **1. Everything on your machine, nothing to install** ("Gemma inside", `in-process`) — 🟢 private +
  free + offline; nothing leaves the computer. **⭐ recommended if the machine has ≥ 12 GB of RAM and
  is NOT an Intel Mac** (otherwise unavailable / too tight on RAM). `--embedder in-process`.
- **2. Via an API (Gemini, OpenAI, Mistral, or your own endpoint)** — 🟡 light on the machine; the
  text of your notes is sent to the provider's API. **Several providers possible** — Gemini, OpenAI,
  Mistral, or your company's / own AI endpoint. **⭐ recommended on a small machine (≤ 8 GB) or an
  Intel Mac.** 🛡️ **Don't dramatize**: in many cases your data is **NOT** used for training — it
  depends on the provider and the plan. Tell the user that, **depending on the API they pick, they
  must choose the right settings** (a paid tier, or the provider's "no-training" / data-controls

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tpierrain/second-brain-generator](https://github.com/tpierrain/second-brain-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
