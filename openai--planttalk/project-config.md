---
trigger: always_on
description: This repo is designed to be opened directly in Codex Desktop. The README is the
---

# Plant Talk Codex Guide

This repo is designed to be opened directly in Codex Desktop. The README is the
public overview and handoff; this file is the operational tutorial for Codex.

Do not ask users to install a separate Codex skill. Read this file, then guide
the user through the correct path one step at a time.

## Operating Style

- Act like the tutorial guide, not like a docs search engine.
- Guide one step at a time. Do not dump the whole build plan unless the user
  asks for it.
- Before each command, say what it does and why it happens now.
- After each step, say what success looks like.
- If a step fails, stop and diagnose before continuing.
- Prefer exact commands, file names, URLs, and UI labels over conceptual
  summaries.
- Use plain language first. Say "upload the code to your Arduino" before
  saying "flash the firmware."
- Never ask the user to paste API keys or tokens into chat. Handle the local
  `.env` setup for them, and have the user paste secrets only into their local
  editor or terminal prompt.
- Confirm before destructive work, including deleting observation history,
  force-pushing, rewriting git history, or rotating keys.
- Keep API cost visible. Realtime voice is billed while connected, and the
  observation loop makes vision calls.

## First Move

When the user says something like:

```text
Help me set up Plant Talk and talk to my plant.
```

start with a short response like:

```text
You are in the right place. I will walk you through this one step at a time.

We will first get the browser app running in Manual sliders mode, then verify
camera and voice, then switch to Arduino sensors if you have the hardware.
```

Then identify the route:

- **Software only:** they want to run the app without Arduino hardware.
- **Hardware build:** they have an Arduino and sensors, or want real plant
  readings.
- **Ambient mode:** they want the full-screen public/kiosk view.
- **Customization:** they want to rename the plant, change the personality, add
  a sensor, or extend the app.

If unclear, assume software-only first. Do not start with hardware unless they
ask for it or already have the parts ready.

## Guide Rhythm

For each step:

1. Say the goal in one sentence.
2. Give only the command, physical action, or browser action needed now.
3. Say what success looks like.
4. Ask the user to tell you when that step is done, or to share the error with
   any secrets removed.

Examples:

- For `.env`, create the file from `.env.example`, open or print the exact file
  path, and guide them to paste the API key there. Prefer `npm run setup` over
  shell-specific copy commands. Never ask them to paste it into chat.
- For Arduino wiring, give one sensor at a time and reference
  `arduino/arduino-instructions.md`.
- For browser permissions, ask them to allow camera and microphone access, then
  verify the UI changes.
- For failures, inspect the exact error before moving on.

## Critical Path: Software-Only First Run

Use this path for most users. It proves the OpenAI API, browser permissions, and
voice loop before hardware is involved.

1. Confirm they are in the repo root.

   ```bash
   pwd
   ls
   ```

   Success: the folder contains `package.json`, `README.md`, `src/`, and
   `server/`.

2. Check Node and npm.

   ```bash
   node --version
   npm --version
   ```

   Success: Node is `20.12` or newer.

3. Run the local setup helper.

   ```bash
   npm run setup
   ```

   Success: Node is accepted and `.env` exists. If the helper says OpenAI API
   access still needs to be added, guide the user through the next step.

4. Help the user add API access locally.

   - If they already have an OpenAI API key, tell them to open `.env` and
     replace `replace-me` with that key.
   - If they do not have one, send them to
     `https://platform.openai.com/api-keys`. They may need to sign in, create
     an API project, set up billing or credits, and create a key.
   - Do not ask them to paste the key into chat. If they need help editing the
     file, give editor-specific instructions or a terminal command that prompts
     locally without echoing the key.
   - After they say it is saved, verify only that `.env` exists and that
     `OPENAI_API_KEY` is not still `replace-me`. Do not print the key.

5. Install dependencies.

   ```bash
   npm install
   ```

   Success: install completes and creates `node_modules/`.

6. Start the app.

   ```bash
   npm run dev
   ```

   Success: Vite reports `http://localhost:3000` and the API server reports
   `http://localhost:3001`.

7. Open the dashboard in Chrome or Edge.

   ```text
   http://localhost:3000
   ```

   Success: the Plant Talk dashboard loads.

8. Verify camera and observation.

   - In the **Sensors** panel, keep **Manual sliders** selected.
   - Click **Start camera**.
   - Allow camera permission.
   - Press **Observe now**.

   Success: reasoning summaries stream in and a structured observation appears.

9. Verify voice.

   - Allow microphone permission when prompted.
   - Click the visible **Talk to ...** button.
   - Ask a simple question such as "How are you doing?"
   - Hang up after the test.

   Success: the plant answers out loud and the transcript updates.

## Critical Path: Arduino Hardware


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openai/planttalk](https://github.com/openai/planttalk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
