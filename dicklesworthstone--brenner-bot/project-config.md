---
trigger: always_on
description: You may NOT delete any file or directory unless I explicitly give the exact command **in this session**.
---

# AGENTS.md — BrennerBot Project

## RULE 1 – ABSOLUTE (DO NOT EVER VIOLATE THIS)

You may NOT delete any file or directory unless I explicitly give the exact command **in this session**.

- This includes files you just created (tests, tmp files, scripts, etc.).
- You do not get to decide that something is "safe" to remove.
- If you think something should be removed, stop and ask. You must receive clear written approval **before** any deletion command is even proposed.

Treat "never delete files without permission" as a hard invariant.

---

## IRREVERSIBLE GIT & FILESYSTEM ACTIONS

Absolutely forbidden unless I give the **exact command and explicit approval** in the same message:

- `git reset --hard`
- `git clean -fd`
- `rm -rf`
- Any command that can delete or overwrite code/data

Rules:

1. If you are not 100% sure what a command will delete, do not propose or run it. Ask first.
2. Prefer safe tools: `git status`, `git diff`, `git stash`, copying to backups, etc.
3. After approval, restate the command verbatim, list what it will affect, and wait for confirmation.
4. When a destructive command is run, record in your response:
   - The exact user text authorizing it
   - The command run
   - When you ran it

If that audit trail is missing, then you must act as if the operation never happened.

---

## Node / JS Toolchain

- Use **bun** for everything JS/TS.
- ❌ Never use `npm`, `yarn`, or `pnpm`.
- Lockfiles: only `bun.lock`. Do not introduce any other lockfile.
- Target **latest Node.js**. No need to support old Node versions.
- **Note:** `bun install -g <pkg>` is valid syntax (alias for `bun add -g`). Do not "fix" it.

### Bun Standalone Executables (`bun build --compile`)

Bun’s “single‑file executable” means the output is one native binary file, **not** that your CLI must live in a single `.ts` source file.

From Bun’s docs, `bun build --compile` bundles your entire dependency graph (imported files + used packages) plus a copy of the Bun runtime into one executable:
- `https://bun.sh/docs/bundler/executables`

```bash
# Point at the entrypoint (example uses `./src/cli.ts`; in this repo the current entrypoint is `./brenner.ts`).

# build a self-contained executable for the current platform
bun build --compile ./src/cli.ts --outfile brenner

# cross-compile for a target platform/arch (examples)
bun build --compile --target=bun-linux-x64 ./src/cli.ts --outfile brenner
bun build --compile --target=bun-windows-x64 ./src/cli.ts --outfile brenner.exe
bun build --compile --target=bun-darwin-arm64 ./src/cli.ts --outfile brenner

# current repo entrypoint
bun build --compile ./brenner.ts --outfile brenner
```

Targets include libc + CPU variants (e.g. `bun-linux-x64-musl`, `bun-linux-x64-baseline`, `bun-linux-x64-modern`) — use `baseline` if you need compatibility with older x64 CPUs (avoids “Illegal instruction”).

---

## Project Architecture

BrennerBot is a **documents-first research project** that will grow into a web-based multi-agent “research lab”
coordinated via Agent Mail.

### A) Corpus + syntheses (current)
- **Purpose:** Canonical primary source + model writeups for extracting and operationalizing the “Brenner approach”.
- **Key artifacts:** `complete_brenner_transcript.md`, metaprompts, and the model response folders.

### B) Web app (`apps/web/`)
- **Framework:** Next.js **16.x** (App Router) + React **19**
- **Orchestration:** **no vendor AI APIs**; coordinate **Codex CLI / Claude Code / Gemini CLI** via **Agent Mail**
- **Runtime/Tooling:** Bun
- **Hosting:** Vercel
- **Purpose:** A UI to:
  - browse/search the corpus and curated quote banks
  - run structured “Brenner Loop” research sessions
  - orchestrate multi-model collaboration (Claude/Codex/Gemini)
  - produce durable artifacts (hypothesis slates, discriminative tests, assumption ledgers, memos)

### C) Prompt templates + protocols (planned; keep file sprawl minimal)
- **Purpose:** Versioned prompt templates and rubrics that implement Brenner-style workflows.

### D) Agent Mail integration (external)
- **Purpose:** Coordination substrate (threads, inbox/outbox, acknowledgements, file reservations).
- **Repo:** `https://github.com/Dicklesworthstone/mcp_agent_mail`

---

## Repo Layout

```
brenner_bot/
├── README.md
├── AGENTS.md
├── brenner.ts                     # CLI entrypoint (7500+ lines)
├── install.sh / install.ps1      # Installer scripts
├── complete_brenner_transcript.md
├── initial_metaprompt.md
├── metaprompt_by_gpt_52.md
├── gpt_pro_extended_reasoning_responses/
├── gemini_3_deep_think_responses/
├── opus_45_responses/
├── specs/                         # Protocol specifications
├── apps/
│   └── web/                       # Next.js 16.x (App Router) + React 19
│       ├── src/app/               # App Router pages
│       ├── src/components/        # UI components
│       ├── src/lib/               # Shared libraries
│       └── package.json
└── .beads/                        # Issue tracking (br)
```

---

## Generated Files — NEVER Edit Manually

**Current state:** There are **no checked-in generated source files** in this repo.

If/when we add generated artifacts (e.g., corpus indexes, derived quote banks, compiled prompt catalogs):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/brenner_bot](https://github.com/Dicklesworthstone/brenner_bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
