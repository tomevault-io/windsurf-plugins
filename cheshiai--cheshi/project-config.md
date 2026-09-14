---
trigger: always_on
description: These instructions apply to the entire repository. Model selection and reasoning
---

# Repository Instructions

These instructions apply to the entire repository. Model selection and reasoning
effort belong to the active runtime; this file defines the repository's working
agreements.

## Local preferences

If `AGENTS.local.md` exists in the repository root, read it for local language
preferences and machine-specific tooling instructions. It is optional and
Git-ignored; contributors do not need to create it. Keep shared development
rules in this file and personal settings in the local file. Local preferences
do not override explicit user requests or shared repository requirements.

## Working agreements

- Complete the requested work, including relevant validation, within the agreed
  scope. Use conversation context to resolve routine implementation choices.
- Decide file organization, naming, internal design, work order, and validation
  methods autonomously within the requested scope and repository conventions.
  Ask about choices that materially change the intended result or scope.
- Fix adjacent issues when necessary to complete or validate the requested work,
  and report those changes. Propose independent features or large structural
  changes separately before expanding the scope.
- Respect current user directions over repository defaults and skill guidance,
  within the runtime's instruction hierarchy. Keep unrelated changes intact.
- When actual work shows that a repository rule or user direction limits a
  better solution, proactively explain the specific constraint and a concrete
  example, then propose an alternative and a focused adjustment with its
  benefits and tradeoffs. Continue authorized work where possible and follow
  the current instruction until the user approves changing it.
- Ask a focused question only when missing information materially changes the
  scope or blocks correct execution. Continue independent, authorized work while
  waiting. Do not request the same authorization twice.
- If a rule or tool approval blocks work, identify the exact instruction or
  rejection and its source. Prepare the reviewable result as far as authorized.
- Treat corrections and status questions as part of the ongoing task. Preserve
  decisions, completed checks, and remaining work across context compaction.
- Before saving project information, learned findings, or other content to
  persistent memory, tell the user what will be saved and where, ask for
  permission, and wait for explicit approval. Apply this to both new memory
  entries and updates to existing memories.
- Use judgment to decide when and how often to propose memory updates based on
  lasting usefulness, novelty, and relevance to future work. Group related
  findings into one proposal when practical to avoid unnecessary interruptions;
  this discretion does not authorize saving without the user's approval.
- Use parallel tool calls for independent reads and checks. Keep dependent
  operations and edits sequential. Autonomously delegate independent tasks to
  subagents when this can save time or improve quality. Assign bounded tasks
  with clear file ownership, coordinate shared dependencies, and personally
  review the combined result and relevant validation. Delegation follows the
  same scope and approval requirements as work performed directly.
- Use the user's requested language. Lead with the result, give brief
  progress updates during sustained work, and report concrete changes, checks,
  and unresolved issues without repeating the work log.

## Repository boundaries and tooling

- `cli/cheshi-cli.ts`: public CLI entry point.
- `codegraph/`: indexing, extraction, search, graph, and MCP engine.
- `desktop/main.mts`, `desktop/preload.cts`, `desktop/lib/`: Electron lifecycle,
  IPC bridge, and desktop services. Keep orchestration separate from service logic.
- `desktop/backend/codegraph-server/`: CodeGraph HTTP API.
- `desktop/frontend/src/features/`: feature UI and state; shared controls belong
  in `desktop/frontend/src/shared/ui/`.
- `desktop/native/`, `config/`, `scripts/`, `forge.config.mts`: native integration,
  configuration, development tooling, and packaging.
- Prefer feature and service boundaries compatible with an MSA monorepo. Split
  modules by responsibility, make dependency direction explicit, and preserve
  public contracts and behavior. Introduce independently deployed services or
  new communication layers only when included in the requested scope.
- Author JavaScript-family code in `.ts`, `.mts`, `.tsx`, or `.cts` as appropriate.
  Edit source files rather than generated `.js`, `.mjs`, or `.cjs` artifacts.
- Use Bun for dependency management and routine scripts. Check `package.json`
  for current command names; do not substitute npm or pnpm without a task reason.
- Use `cheshi-cli codegraph ...` publicly, or `bun run cheshi-cli codegraph ...`
  from an unlinked checkout. Use `bun run desktop:dev` for local development.

## CodeGraph workflow

Cheshi stores CodeGraph indexes outside source repositories under its user-data
directory. Do not use a repository-local `.codegraph/` directory as the index
status signal. Check the current Workspace with
`cheshi-cli codegraph status /absolute/path/to/workspace --json`; when it reports

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CheshiAI/Cheshi](https://github.com/CheshiAI/Cheshi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
