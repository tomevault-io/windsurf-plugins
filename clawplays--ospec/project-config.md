---
trigger: always_on
description: Document-driven OSpec workflow for AI-assisted development with change-ready initialization, execution, validation, and archive readiness.
---


# OSpec CLI

Document-driven OSpec workflow for AI-assisted development with change-ready initialization, execution, validation, archiving, and docs maintenance.

## Default Entry

When the user says something short like:

- `使用 ospec 初始化项目`
- `使用 ospec 初始化这个目录`
- `use ospec to initialize this directory`
- `use ospec to initialize this repo`

expand it internally as:

1. initialize the repository with `ospec init` so it ends in a change-ready state
2. if project context is missing and the AI can ask follow-up questions, ask one concise question for project summary or tech stack
3. if the user declines or the flow is CLI-only, continue with placeholder project docs
4. create the first change only when explicitly requested

Do not force the user to repeat those steps manually when the request is already clear.

Treat plain project-init intent as enough to trigger this flow. Do not require the user to restate the guardrails in a longer prompt.

## Mandatory Init Execution

When the user asks to initialize a directory, do not freehand the initialization flow.

If the user intent is simply to initialize the project or current directory, treat that as a request for this mandatory flow.

Use this exact behavior:

1. run `ospec init [path]` when the directory is uninitialized or not yet change-ready
2. in AI-assisted init, map an explicit language request or the current conversation language to `--document-language`; do not assume a brand-new repo will infer it
3. if AI assistance is available and the repository lacks usable project context, ask one concise follow-up for summary or tech stack before init when helpful
4. verify the actual filesystem result before claiming initialization is complete
5. stop before `ospec new` unless the user explicitly asks to create a change

Never replace `ospec init` with manual directory creation or a hand-written approximation.

Do not say initialization is complete unless the managed protocol-shell assets and baseline project knowledge docs actually exist on disk.

Required checks after `ospec init`:

- `.skillrc`
- `.ospec/`
- `changes/active/`
- `changes/archived/`
- `SKILL.md`
- `SKILL.index.json`
- `.ospec/tools/build-index-auto.cjs`
- `for-ai/ai-guide.md`
- `for-ai/execution-protocol.md`
- `for-ai/naming-conventions.md`
- `for-ai/skill-conventions.md`
- `for-ai/workflow-conventions.md`
- `for-ai/development-guide.md`
- `docs/project/overview.md`
- `docs/project/tech-stack.md`
- `docs/project/architecture.md`
- `docs/project/module-map.md`
- `docs/project/api-overview.md`

During plain init, do not report `docs/SKILL.md`, optional knowledge maps such as `knowledge/src/SKILL.md` / `knowledge/tests/SKILL.md`, or business scaffold as if they were part of change-ready completion.

## Prompt Profiles

Use these prompt styles as the preferred mental model.

### 1. Minimal Prompt

Use when the user already trusts OSpec defaults.

```text
Use ospec to initialize this project.
```

### 2. Standard Prompt

Use when you want short prompts and still want OSpec to finish initialization properly.

```text
Use ospec to initialize this project according to current OSpec rules.
```

### 3. Guided Init Prompt

Use when you want the AI to gather missing context before initialization if needed.

```text
Use ospec to initialize this project. If project context is missing, ask me for a short summary or tech stack first. If I skip it, continue with placeholder docs.
```

### 4. Docs Maintenance Prompt

Use when the repository is already initialized and the project knowledge layer needs a refresh or repair pass.

```text
Use ospec to refresh or repair the project knowledge layer for this directory. Do not create a change yet.
```

### 5. Change-Creation Prompt

Use when the user is explicitly ready to move into a small or routine execution flow.

```text
Use ospec to create and advance a change for this requirement. Respect the current project state and do not create queue work unless I ask for it.
```

### 5b. Goal-Creation Prompt

Use when the user explicitly wants the full 1.2 workflow for complex, cross-cutting, high-risk, multi-worker, or evidence-heavy work.

```text
Use ospec goal to create and advance a full goal for this requirement. Use design, implementation-plan, task graph, review, worker-status, and evidence gates.
```

### 6. Queue Prompt

Use when the user explicitly wants multiple changes queued instead of one normal active change.

```text
Use ospec to break this TODO into multiple changes, create a queue, show the queue first, and do not run it yet.
```

### 7. Queue-Run Prompt

Use when the user explicitly wants queue execution, not the normal single-change flow.

```text
Use ospec to create a change queue and execute it explicitly with ospec run manual-safe.
```

## Anti-Drift Rules

Always keep these rules:

- `ospec init` should leave the repository in a change-ready state
- in AI-assisted init, pass `--document-language` from the explicit language request or current conversation language when the project language is already apparent
- AI-assisted init may ask one concise follow-up question for missing summary or tech stack; if the user declines, continue with placeholder docs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clawplays/ospec](https://github.com/clawplays/ospec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
