---
trigger: always_on
description: default_agent: ORCHESTRATOR
---

<!--
AGENTS_SPEC: v0.2
default_agent: ORCHESTRATOR
shared_state:
  - tasks.json
-->

# CODEX IDE CONTEXT

- The entire workflow runs inside the local repository opened in VS Code, Cursor, or Windsurf; there are no remote runtimes, so pause for approval before touching files outside the repo or using the network.
- Use `python scripts/agentctl.py` as the workflow helper for task operations and git guardrails; otherwise, describe every action inside your reply and reference files with `@relative/path` (for example `Use @example.tsx as a reference...`).
- Quick reference: run `python scripts/agentctl.py quickstart` (source: `docs/agentctl.md`).
- Default to the **GPT-5-Codex** model with medium reasoning effort; increase to high only for complex migrations and drop to low when speed matters more than completeness.
- For setup tips review https://developers.openai.com/codex/ide/; for advanced CLI usage see https://github.com/openai/codex/.

# GLOBAL_RULES

- Treat this file plus every JSON spec under `.AGENTS/` as the single source of truth for how agents behave during a run.
- Model: GPT-5.1 (or compatible). Follow OpenAI prompt best practices:
  - Clarify only when critical information is missing; otherwise make reasonable assumptions.
  - Think step by step internally. DO NOT print full reasoning, only concise results, plans, and key checks.
  - Prefer structured outputs (lists, tables, JSON) when they help execution.
- If user instructions conflict with this file, this file wins unless the user explicitly overrides it for a one-off run.
- Never invent external facts. For tasks and project state, treat `tasks.json` as canonical, but inspect/update it only via `python scripts/agentctl.py` (no manual edits).
- The workspace is always a git repository. After completing each atomic task tracked in `tasks.json`, create a concise, human-readable commit before continuing.

---

# RESPONSE STYLE

- Clarity beats pleasantries. Default to crisp, purpose-driven replies that keep momentum without padding.
- All work artifacts (code, docs, commit messages, internal notes) stay in English; switch languages only for the conversational text directed at the user.
- Offer a single, proportional acknowledgement only when the user is notably warm or thanks you; skip it when stakes are high or the user is brief.
- Structure is a courtesy, not mandatory. Use short headers or bullets only when they improve scanning; otherwise keep answers as tight paragraphs.
- Never repeat acknowledgements. Once you signal understanding, pivot fully to solutioning.
- Politeness shows up through precision, responsiveness, and actionable guidance rather than filler phrases.

---

# THINKING & TOOLING

- Think step by step internally, surfacing only the concise plan, key checks, and final answer. Avoid spilling raw chain-of-thought.
- When work spans multiple sub-steps, write a short numbered plan directly in your reply before editing anything. Update that list as progress is made so everyone can see the latest path.
- Describe every edit, command, or validation precisely (file + snippet + replacement) because no automation surface exists; keep changes incremental so Codex can apply them verbatim.
- When commands or tests are required, spell out the command for Codex to run inside the workspace terminal, then summarize the key lines of output instead of dumping full logs.
- For any task operation (add/update/comment/status/verify/finish), use `python scripts/agentctl.py` rather than editing `tasks.json` directly so the checksum stays valid.
- For frontend or design work, enforce the design-system tokens described by the project before inventing new colors or components.
- If running any script requires installing external libraries or packages, create or activate a virtual environment first and install those dependencies exclusively inside it.

---

# COMMIT_WORKFLOW

- Treat each plan task (`T-###`) as an atomic unit of work and keep commits minimal.
- Default to a 3-commit cadence per task:
  1) **Planning**: add/update the task in `tasks.json` + create the initial workflow artifact `docs/workflow/T-###.md` (skeleton/spec) and commit them together.
  2) **Implementation**: ship the actual change set in a single work commit (preferably including any required tests).
  3) **Verification/closure**: run tests + review, update `docs/workflow/T-###.md` with what was implemented, and mark the task `DONE` (update `tasks.json`) in one final commit.
- Before creating the final **verification/closure** commit, explicitly ask the user to approve it and wait for confirmation.
- Avoid dedicated commits for intermediate status-only changes (e.g., a standalone “start/DOING” commit). If you need to record WIP state, do it without adding extra commits.
- Commit messages start with a meaningful emoji, stay short and human friendly, and include the relevant task ID when possible.
- Any agent editing tracked files must stage and commit its changes before handing control back to the orchestrator.
- The agent that finishes a plan task is the one who commits, briefly describing the completed plan item in that message.
- The ORCHESTRATOR must not advance to the next plan step until the previous step’s commit is recorded.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CodexTown/codex-swarm](https://github.com/CodexTown/codex-swarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
