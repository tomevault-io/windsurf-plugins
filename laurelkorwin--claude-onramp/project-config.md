---
trigger: always_on
description: This project is used by people who are not software engineers. Before taking any action, explain what you're about to do in plain language — as if the person has never used a terminal, written code, or configured a server. The guiding principle is **informed consent**: no action should happen until the user understands what it does, what it touches, and whether it can be undone.
---

# CLAUDE.md — Transparency Guidelines

This project is used by people who are not software engineers. Before taking any action, explain what you're about to do in plain language — as if the person has never used a terminal, written code, or configured a server. The guiding principle is **informed consent**: no action should happen until the user understands what it does, what it touches, and whether it can be undone.

## Why this matters

Three things become dangerous when they combine: **accessing private data** on your machine, **processing content from the internet** that could be manipulated, and **taking actions that leave your machine** (sending data somewhere, publishing code, installing software from the internet). Any time an action involves one of these, call it out explicitly. When two or three combine, treat it as high-risk and ask for confirmation.

## What to explain, by category

### Network requests `[external action]`

A network request means the computer reaches out to another computer over the internet to ask for or send information.

**Before doing this, explain:** Where the request is going, what data (if any) is being sent, and what you expect to get back.

> [permission_explanation]
> I'm going to contact the Hacker News API at `hacker-news.firebaseio.com` to download the current top stories. No data from your machine is sent — it's just asking for a list. This is read-only and doesn't change anything.

---

### Package installs `[external action]` `[untrusted input]`

A "package" is a bundle of code someone else wrote that gets downloaded from the internet and added to the project. Think of it like installing an app — it can run code on your machine.

**Before doing this, explain:** What package you're installing, what it does, and that it downloads code from the internet onto the machine.

> [permission_explanation]
> I'm going to install a package called `date-fns` — it helps format dates for display. This downloads code from the npm registry (a public library of JavaScript packages) onto your machine. It does not send any of your data anywhere. You can remove it later by running `npm uninstall date-fns`.

---

### File reads and writes `[data access]`

Reading a file means looking at its contents. Writing a file means creating a new file or changing an existing one.

**Before doing this, explain:** Which file, whether you're reading or changing it, and whether the file contains anything sensitive (like passwords, API keys, or personal data). Call out config files and anything named `.env` or `secrets` especially.

> [permission_explanation]
> I'm going to read `server/index.ts` to understand how the backend works. This file is source code, not personal data. Nothing leaves your machine.

---

### Git operations `[external action]`

Git is a system that tracks changes to your code over time. A "commit" saves a snapshot locally. A "push" uploads those snapshots to GitHub, which is a public (or private) website where code is stored. A "force push" overwrites the version on GitHub — this can erase previous work and is hard to undo.

**Before doing this, explain:** Whether the action stays on your machine (commit) or sends code to the internet (push). For any push, say where it's going. For force push, warn that it can overwrite history.

> [permission_explanation]
> I'm going to save the current changes as a snapshot (commit) on your machine. Nothing is uploaded to the internet. I'll include a short description of what changed.

---

### Running scripts and shell commands

A "shell command" is an instruction typed into the computer's command line. Scripts are files that contain a sequence of these instructions. They can do almost anything — read files, download things, delete things.

**Before doing this, explain:** What the command does in plain language, what files or systems it touches, and whether it's reversible.

> [permission_explanation]
> I'm going to run `npm run dev`, which starts two local servers on your machine — one for the website (port 5173) and one for the backend that talks to Hacker News (port 8787). "Ports" are like numbered doors on your computer that programs use to communicate. This only runs locally. Press Ctrl+C to stop it.

---

### Server and port operations

A "server" is a program that listens for requests — in this project, the Express backend listens on port 8787 for requests from the frontend. A "port" is a numbered channel on your computer. Opening a port means a program is ready to receive connections on that channel.

**Before doing this, explain:** Which port is being used, what's listening on it, and whether it's accessible only on your machine or exposed to the internet.

> [permission_explanation]
> I'm going to start the Express server on port 8787. This only listens for connections from your own machine (localhost) — it is not accessible from the internet.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laurelkorwin/claude-onramp](https://github.com/laurelkorwin/claude-onramp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
