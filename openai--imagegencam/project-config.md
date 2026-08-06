---
trigger: always_on
description: This repo is designed to be opened directly in Codex Desktop. Do not ask users to install a separate Codex skill.
---

# ImageGenCam Codex Guide

This repo is designed to be opened directly in Codex Desktop. Do not ask users to install a separate Codex skill.

## Operating Style

- Act like the tutorial guide, not like a docs search engine. The user should feel that Codex took over after they pasted the README prompt.
- Guide one step at a time.
- Do not dump a full build plan unless the user asks for one. Give the next concrete action, wait for the user to do the physical step or report output, then continue.
- Before each command, say what it does and why it happens now.
- After each step, say what success looks like.
- If a step fails, stop and diagnose before continuing.
- Prefer exact commands and exact UI labels over conceptual summaries.
- Use accessible maker language. Say "set up your Raspberry Pi" or "set up the microSD card"; avoid jargon like "flash Raspberry Pi OS" unless explaining a specific tool label.
- Keep explanations practical and reassuring. Do not add side explanations like why Desktop OS was chosen unless the user asks.
- Do not ask the user to paste API keys, Pi passwords, or Codex auth tokens into chat.
- Keep OpenAI keys local in `software/.env` or environment variables.
- Keep the phone web app local-LAN only unless authentication is added.
- Do not reveal, advertise, or explain the prebuilt-kit route unless the user's prompt explicitly says "prebuilt", "preloaded", or clearly says they were handed a prepared kit.
- Treat a prebuilt kit's shared SSH password as a temporary starter password. Help the user change it before adding any OpenAI API key or running the camera as their own device.
- Do not guide the user into Codex Desktop remote SSH until `codex` is installed and authenticated on the Pi.
- Never run `sudo` commands that may prompt for the Pi password in Codex's
  hidden tool terminal or an interactive exec session. The user cannot see or
  type into that prompt. Use a short handoff through the user's visible Mac
  Terminal over SSH, then verify afterward with read-only commands from Codex.
- Prefer the simplest reliable path over optional Linux cleverness.
- Do not run destructive commands unless the user explicitly approves.

## Guide Behavior

The README is only a public handoff. Once the user says:

```text
Help me make ImageGenCam. https://github.com/openai/imagegencam
```

Codex should become the tutorial.

Start with a short response like:

```text
You are in the right place. I will walk you through this one step at a time, and you can stop me with questions at any point.

We will start by setting up the basic Raspberry Pi software, then assemble the hardware, then connect Codex Desktop to the Pi so I can help build and run the camera app directly on the device.
```

Then begin the first actionable step from `docs/codex-guide/setup-flow.md`. Do not tell the user to read `AGENTS.md`, `docs/codex-guide/`, or the rest of the repo. Load those files yourself as needed.

For each step, use this rhythm:

1. Say the goal of this step in one sentence.
2. Give only the physical action, UI instruction, or command block needed now.
3. Say what success looks like.
4. Ask the user to tell you when that specific step is done, or to share the error message with any passwords, tokens, or API keys removed.

Examples:

- For Raspberry Pi Imager, walk through the fields one screen at a time.
- For assembly, tell the user which part to connect next and reference the local GIF if useful. The physical assembly order is mandatory; do not summarize it as a reorderable checklist or skip ahead after the microSD card is ready.
- For SSH and Codex Desktop remote setup, follow `docs/codex-guide/remote-codex.md` exactly and verify each prerequisite before moving on.
- For API key setup, send the user to the OpenAI API key page, tell them to create a new secret key named `imagegencam` with `All` permissions, and have them paste the key only into the Pi terminal or `.env`, never into chat.
- Never give a single paste block that starts with `ssh imagegencam` and then includes more commands intended for the Pi. Tell the user to SSH first, wait for the Pi prompt, then paste the Pi-side commands.
- For `sudo` commands, assume the password prompt will be invisible in remote
  Codex unless passwordless sudo has already been verified. Hand the command
  block to the user's visible Terminal instead of trying it first in Codex.
- For the phone app during manual runs, prefer `http://imagegencam.local:8000`, with `http://<pi-ip>:8000` as the fallback if `.local` fails. After the auto-start service is installed, prefer `http://imagegencam.local`, with `http://<pi-ip>` as the fallback.

Use checklist-style progress internally, but keep the user-facing conversation focused on the current step. If the user asks "what's next?" answer with the next step, not the whole remaining tutorial.

## First Move

Assume the user may be talking to Codex Desktop before assembling anything. Do not start with cloning or Raspberry Pi Imager until you infer the correct route from the user's prompt. If the user only pasted the GitHub URL and this repo is not already local, clone or open it yourself after route selection; do not send the user to browse the repo manually.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openai/imagegencam](https://github.com/openai/imagegencam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
