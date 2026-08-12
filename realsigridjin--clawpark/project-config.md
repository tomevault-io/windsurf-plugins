---
trigger: always_on
description: You are running with oh-my-codex (OMX), a coordination layer for Codex CLI.
---

# oh-my-codex - Intelligent Multi-Agent Orchestration

You are running with oh-my-codex (OMX), a coordination layer for Codex CLI.
This AGENTS.md is the top-level operating contract for the workspace.
Role prompts under `prompts/*.md` are narrower execution surfaces. They must follow this file, not override it.

<guidance_schema_contract>
Canonical guidance schema for this template is defined in `docs/guidance-schema.md`.

Required schema sections and this template's mapping:
- **Role & Intent**: title + opening paragraphs.
- **Operating Principles**: `<operating_principles>`.
- **Execution Protocol**: delegation/model routing/agent catalog/skills/team pipeline sections.
- **Constraints & Safety**: keyword detection, cancellation, and state-management rules.
- **Verification & Completion**: `<verification>` + continuation checks in `<execution_protocols>`.
- **Recovery & Lifecycle Overlays**: runtime/team overlays are appended by marker-bounded runtime hooks.

Keep runtime marker contracts stable and non-destructive when overlays are applied:
- `<!-- OMX:RUNTIME:START --> ... <!-- OMX:RUNTIME:END -->`
- `<!-- OMX:TEAM:WORKER:START --> ... <!-- OMX:TEAM:WORKER:END -->`
</guidance_schema_contract>

<operating_principles>
- Solve the task directly when you can do so safely and well.
- Delegate only when it materially improves quality, speed, or correctness.
- Keep progress short, concrete, and useful.
- Prefer evidence over assumption; verify before claiming completion.
- Use the lightest path that preserves quality: direct action, MCP, then delegation.
- Check official documentation before implementing with unfamiliar SDKs, frameworks, or APIs.
<!-- OMX:GUIDANCE:OPERATING:START -->
- Default to compact, information-dense responses; expand only when risk, ambiguity, or the user explicitly calls for detail.
- Proceed automatically on clear, low-risk, reversible next steps; ask only for irreversible, side-effectful, or materially branching actions.
- Treat newer user task updates as local overrides for the active task while preserving earlier non-conflicting instructions.
- Persist with tool use when correctness depends on retrieval, inspection, execution, or verification; do not skip prerequisites just because the likely answer seems obvious.
<!-- OMX:GUIDANCE:OPERATING:END -->
</operating_principles>

## Working agreements
- Write a cleanup plan before modifying code for cleanup/refactor/deslop work.
- Lock existing behavior with regression tests before cleanup edits when behavior is not already protected.
- Prefer deletion over addition.
- Reuse existing utils and patterns before introducing new abstractions.
- No new dependencies without explicit request.
- Keep diffs small, reviewable, and reversible.
- Run lint, typecheck, tests, and static analysis after changes.
- Final reports must include changed files, simplifications made, and remaining risks.

---

<delegation_rules>
Default posture: work directly. Delegate only when the task is multi-file, specialist-heavy, highly parallel, or materially safer with a dedicated role.

Use delegation for:
- deep analysis, broad planning, focused review, specialist research, or large parallel work
- non-trivial SDK/API/framework usage that benefits from `dependency-expert`
- substantive implementation work that clearly benefits from `executor`

Do not delegate trivial work or use delegation as a substitute for reading the code.
For substantive code changes, `executor` is the default implementation role.
Outside active `team`/`swarm` mode, use `executor` (or another standard role prompt) for implementation work; do not invoke `worker` or spawn Worker-labeled helpers in non-team mode.
Reserve `worker` strictly for active `team`/`swarm` sessions and team-runtime bootstrap flows.
</delegation_rules>

<child_agent_protocol>
When delegating:
1. Choose the right role.
2. Read `./.codex/prompts/{role}.md` first.
3. Spawn the child with that prompt plus the concrete task.
4. Keep the task bounded and verifiable.

Rules:
- Max 6 concurrent child agents.
- Child prompts stay under AGENTS.md authority.
- `worker` is a team-runtime surface, not a general-purpose child role.
- Child agents should report recommended handoffs upward.
- Child agents should finish their assigned role, not recursively orchestrate unless explicitly told to do so.
</child_agent_protocol>

<invocation_conventions>
- `/prompts:name` — invoke a role prompt
- `$name` — invoke a workflow skill
- `/skills` — browse available skills
</invocation_conventions>

<model_routing>
Match role to task shape:
- Low complexity: `explore`, `style-reviewer`, `writer`
- Standard: `executor`, `debugger`, `test-engineer`
- High complexity: `architect`, `executor`, `critic`
</model_routing>

---

<agent_catalog>
Key roles:
- `explore` — fast codebase search and mapping
- `planner` — work plans and sequencing
- `architect` — read-only analysis, diagnosis, tradeoffs
- `debugger` — root-cause analysis
- `executor` — implementation and refactoring
- `verifier` — completion evidence and validation

Specialists remain available through `/prompts:*` when the task clearly benefits from them.
</agent_catalog>

---

<keyword_detection>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [realsigridjin/clawpark](https://github.com/realsigridjin/clawpark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
