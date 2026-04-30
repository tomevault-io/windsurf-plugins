---
trigger: always_on
description: **YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION FROM ME OR A DIRECT COMMAND FROM ME.**
---

# AGENTS.md — Jeffrey Emanuel Personal Site

## RULE NUMBER 1 (NEVER EVER EVER FORGET THIS RULE!!!)

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION FROM ME OR A DIRECT COMMAND FROM ME.**

Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work that I then need to pay to reproduce.

As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted. You must **ALWAYS** ask and *receive* clear, written permission from me before ever even thinking of deleting a file or folder of any kind!

---

## IRREVERSIBLE GIT & FILESYSTEM ACTIONS — DO-NOT-EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.

2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.

3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.

4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.

5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

---

## Project Overview

This is Jeffrey Emanuel's personal website — a Next.js 16 site showcasing his work, projects, writing, and the Flywheel ecosystem of AI coding tools.

**Repository:** https://github.com/Dicklesworthstone/jeffrey_emanuel_personal_site

**Live Site:** Deployed on Vercel

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Framework | Next.js 16 (App Router) |
| UI | React 19, TypeScript (strict mode) |
| Styling | Tailwind CSS 4 |
| Animations | framer-motion, GSAP |
| 3D Graphics | Three.js, @react-three/fiber, @react-three/drei |
| Icons | lucide-react |
| Search | Fuse.js (client-side fuzzy search) |
| Markdown | gray-matter, react-markdown, rehype, remark |
| Math | KaTeX for LaTeX rendering |
| Testing | Playwright (E2E) |
| Package Manager | **bun** (NEVER npm, yarn, or pnpm) |
| Deployment | Vercel |

---

## Package Manager — BUN ONLY

We **only** use `bun` in this project. NEVER use `npm`, `yarn`, or `pnpm`.

```bash
# Install dependencies
bun install

# Run dev server
bun dev

# Build for production
bun run build

# Run linting
bun lint

# Run type checking
bun tsc --noEmit
```

Dependencies are managed **exclusively** via `package.json` + `bun.lock`. Do **not** introduce `package-lock.json`, `yarn.lock`, or any other lockfiles.

---

## Code Editing Discipline

**NEVER** run a script that processes/changes code files in this repo. No "code mods" you just invented, no giant regex-based `sed` one-liners, no auto-refactor scripts that touch large parts of the tree.

That sort of brittle, regex-based stuff is always a huge disaster and creates far more problems than it ever solves.

* If many changes are needed but they're **mechanical**, use several subagents in parallel to make the edits, but still apply them **manually** and review diffs.
* If changes are **subtle or complex**, you must methodically do them yourself, carefully, file by file.

---

## Backwards Compatibility & File Sprawl

We do **not** care about backwards compatibility — we want the cleanest possible architecture with **zero tech debt**:

* Do **not** create "compatibility shims".
* Do **not** keep old APIs around just in case. Migrate callers and delete the old API (subject to the no-deletion rule for files; code removal inside a file is fine).

**AVOID** uncontrolled proliferation of code files:

* If you want to change something or add a feature, you MUST revise the **existing** code file in place.
* You may NEVER create files like `componentV2.tsx`, `componentImproved.tsx`, `componentNew.tsx`, etc.
* New code files are reserved for **genuinely new domains** that make no sense to fold into any existing module.
* The bar for adding a new file should be **incredibly high**.

---

## Project Structure

```
├── app/                    # Next.js App Router pages
│   ├── page.tsx           # Homepage
│   ├── about/             # About page
│   ├── projects/          # Projects showcase
│   ├── writing/           # Blog/essays (markdown-driven)
│   ├── consulting/        # Consulting services
│   ├── media/             # Press & media appearances
│   └── contact/           # Contact page

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/mcp_agent_mail_website](https://github.com/Dicklesworthstone/mcp_agent_mail_website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
