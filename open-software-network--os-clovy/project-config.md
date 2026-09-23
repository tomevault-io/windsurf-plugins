---
trigger: always_on
description: <!-- SPECKIT START -->
---

<!-- SPECKIT START -->

For additional context about technologies to be used, project structure,
shell commands, and other important information, read
`specs/003-conversation-turns/plan.md`.

<!-- SPECKIT END -->

# Clovy — Agent Instructions

## Project

Clovy is a private-by-architecture **Tauri desktop app** for meeting notes: it
records a meeting or dictation, transcribes the audio, turns the transcript
into a structured note, and hosts an AI agent you can chat with over your
notes. The frontend is **React** (`src/`), the native shell is **Rust**
(`src-tauri/`), and a confidential **Rust backend, Clovy API** (`clovy-api/`),
proxies all upstream AI and runs metered billing. Identity and credits come
from **OS Accounts**; the agent harness is a Clovy-owned TypeScript service
built on the **OpenAI Agents SDK**; AI models are served through Clovy's model
routing. Clovy API runs
inside a TEE (Phala) so prompt data is not readable by its own infra.

> Read **[CONTEXT.md](CONTEXT.md)** before naming anything, and
> **[docs/index.md](docs/index.md)** to find the doc for the area you touch.

## OS Platform (shared brain)

Platform-enabled repo — Product `june`, Team `os-core`, Issue prefix `JUN`.
The product handle and issue prefix are retained June-era technical identities;
the current product name is Clovy (see
[ADR-0055](docs/adr/0055-clovy-technical-identity-migrates-through-a-compatibility-bridge.md)).
Use the `os_platform_*` MCP tools (https://platform-api.opensoftware.co/mcp);
REST fallback `https://app.opensoftware.co/api` + `Authorization: Bearer
$OS_PLATFORM_API_KEY`. Never print or store credentials.

**Before work that will produce a branch** (skip for Q&A, typos, exploration, CI):
1. `os_platform_get_product{handle:"june"}` — response embeds the Product
   Memory index; `os_platform_get_memory` only entries whose description touches
   your task.
2. Find-or-create the Issue: `os_platform_search_product_issues` first; create
   only if no open Issue matches the outcome. One Issue per independently
   reviewable outcome; reuse it across sessions.
3. Set the Issue `in_progress` (`os_platform_set_issue_status` with the Product
   handle, Issue number, and status), branch `JUN-<number>-<slug>`, and
   put `JUN-<number>` in commit subjects and the PR title/body. PR-Links
   advances in_review/completed automatically where installed; if it doesn't,
   set them yourself. Leave unfinished work in its true status.

**After the work lands**: for each durable fact (decision, convention, gotcha
that cost >10 min) — check the memory index, then `os_platform_create_memory` or
`os_platform_update_memory` the existing slug; never a near-duplicate, never
secrets, never a local notes file.

**Posts** (`os_platform_create_post{team:"os-core"}`): only when a teammate
would act differently for reading it — blocked and stopping, a decision that
changes someone's work, a shipped result the derived events don't convey, or a
start of cross-person/cross-session work. Normally ≤1 per Issue per day. Read
`os_platform_get_team_timeline` before asking anyone "what's the status?".

**No access** (no MCP tools, no key): say once — "OS Platform not configured;
see README → OS Platform" — then work normally, fully offline. No local memory
substitute; put durable learnings in the PR description. At handoff, state
"platform sync skipped"; never imply the platform steps ran.

## Structure

```
os-clovy/
├── src/                     # React frontend
│   ├── app/                 # app shell, routing, update-decision
│   ├── components/          # agent (chat), settings, account, onboarding, note-editor, recorder, sidebar, ...
│   ├── lib/                 # agent runtime contracts, model privacy, Tauri bindings, ...
│   ├── styles/              # app.css + tokens.css (design tokens)
│   └── test/                # vitest suites (all frontend tests live here)
├── src-tauri/               # Rust native shell (Cargo package `clovy`)
│   ├── src/audio/           # recording, source separation, turn detection, live preview
│   ├── src/agent_runtime/   # sidecar protocol, tools, persistence, and migration
│   ├── src/os_accounts.rs   # OS Accounts login (PKCE), keychain token store
│   ├── src/providers/       # model-settings persistence
│   ├── src/commands.rs      # the Tauri command surface
│   └── native/              # macOS system-audio helper (Swift) + dictation helper
├── clovy-api/               # Rust backend (Cargo workspace, crates prefixed `clovy-`)
│   └── crates/              # domain / services / providers / config / api / app  (hexagonal)
├── docs/                    # see docs/index.md — ADRs, subsystem docs, runbooks, PRDs, QA
├── specs/                   # Spec Kit feature specs (001-003)
├── spec/                    # enforceable coding rules (see spec/index.md) — distinct from specs/
├── scripts/                 # build / dev / release tooling
├── CONTEXT.md               # domain glossary — canonical names
├── AGENTS.md                # this file (canonical); CLAUDE.md is a symlink to it
└── .agents/skills/          # vendored agent skills, symlinked into .claude/skills/
```

## Domain & decisions — read before writing code

- **[CONTEXT.md](CONTEXT.md)** — the domain glossary / ubiquitous language.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-software-network/os-clovy](https://github.com/open-software-network/os-clovy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
