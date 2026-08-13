---
trigger: always_on
description: A repo-based writing system for creating blog and newsletter content. The workflow runs from durable repo files rather than chat memory, coordinating specialised agents across a repeatable content pipeline.
---

# Newsletter Engine

A repo-based writing system for creating blog and newsletter content. The workflow runs from durable repo files rather than chat memory, coordinating specialised agents across a repeatable content pipeline.

**Primary user:** Jose

---

## Architectural Principles

These govern every skill and orchestration design decision:

1. **File-based I/O contracts** — each skill reads from and writes to a predictable set of files. No skill reaches outside its contract. This is what makes both independent and orchestrated invocation possible.
2. **`post.yaml` is the nervous system** — it is the shared state that every skill reads from and appends to. It is not just metadata — it is the message bus.
3. **Every skill is standalone first** — skills like `/seo` and `/promote` must work on any draft, whether produced by the pipeline or written by Jose independently. No skill should require the full pipeline to have run first.
4. **The orchestrator grows incrementally** — `/new-post` chains only what is available at any point. Each milestone extends it. Separation from the individual skills is maintained throughout.
5. **Separation of concerns** — each skill has one job; the orchestrator has one job (sequencing). This enables triggering any step independently or running the full pipeline end-to-end unattended.

---

## What Is Portable Across Agents

These parts of the system are provider-agnostic and should be treated as the real engine:

- The repo layout and file contracts
- `post.yaml` as the shared state and stage ledger
- The artefact files each skill reads and writes
- The style guides, templates, and reference posts
- The procedures written in `.claude/skills/*/SKILL.md`
- The reviewer personas written in `.claude/agents/*.md`

If an agent can read markdown files and update repo files, it can operate this system.

---

## What Is Provider-Specific

These parts are convenience wrappers, not the core workflow:

- Claude slash-command invocation such as `/draft` or `/review`
- Claude-native agent spawning and parallel sub-agent execution
- Claude-specific MCPs, hooks, and session ergonomics
- Any instruction that assumes a Claude Code-only tool exists

If a provider lacks one of these capabilities, preserve the intended outcome and execute the same file contract manually.

---

## Canonical File Contracts

For any agent working in this repo, the source of truth is:

- `AGENTS.md` — top-level operating manual
- `.claude/skills/*/SKILL.md` — executable procedures for each workflow stage
- `.claude/agents/*.md` — specialist review personas, mainly used by `/review`
- `post.yaml` — shared post state, stage completion, metadata, and artefact pointers
- `templates/`, `style_guide/`, and `reference_posts/` — writing constraints and calibration context
- `posts/<slug>/` — the working directory for each post and all generated artefacts

Agents should prefer updating durable files over returning chat-only results.

---

## How Non-Claude Agents Should Interpret This Repo

If you are not running inside Claude Code:

- Treat each `.claude/skills/*/SKILL.md` as a procedure to execute directly
- Treat each `.claude/agents/*.md` as a reusable role prompt or review persona
- Use the file inputs and outputs described by each skill as the contract to follow
- Respect stage guards and overwrite checks described in the skill before writing files
- Update `post.yaml` whenever a skill says to mark a stage complete or register an artefact

The repo structure matters more than the runtime. If the files are updated correctly, the workflow is considered valid.

---

## Fallback Behavior When Native Tools Do Not Exist

Use these defaults when a provider lacks Claude-specific runtime features:

- No slash commands: open the corresponding `SKILL.md` and execute it manually
- No sub-agent primitive: use `.claude/agents/*.md` as role instructions and run them in the main session
- No parallel agent execution: run critic roles sequentially, then synthesize the results
- No Claude MCP equivalent: continue with local repo files unless the skill truly requires external research
- No hook system: perform the required file updates directly if the workflow depends on them

Do not stop just because a Claude-native convenience is missing. Fall back to the file contract and continue.

---

## Repo Index

| Directory | Purpose |
|-----------|---------|
| `reference_posts/` | Jose's real posts (series, standalone, short_technical) |
| `style_guide/` | Voice, anti-patterns (shared/), per-type rules, promotion_formats.md |
| `.claude/skills/` | Skill instruction files (one per skill) |
| `.claude/agents/` | Critic agent definitions invoked by `/review` (voice, structure, impact) |
| `.claude/hooks/` | Automation hooks: skill-reflector (reflection log), detect-skill-complete |
| `.claude/rules/` | Behavioural guardrails, auto-loaded each session |
| `templates/` | Post folder template (`post.yaml`, `notes.md`, `placeholder.md`) |
| `posts/` | Per-post working folders with artefacts |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JoseParrenoGarcia/newsletter-engine](https://github.com/JoseParrenoGarcia/newsletter-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
