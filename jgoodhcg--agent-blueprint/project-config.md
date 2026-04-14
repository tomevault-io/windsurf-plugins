---
trigger: always_on
description: Follows `AGENT_BLUEPRINT.md` (version: 2026-03-16)
---

# AGENTS

Follows `AGENT_BLUEPRINT.md` (version: 2026-03-16)

## Project Overview

This repository defines and maintains a portable agent operating standard. The primary deliverables are the blueprint (`AGENT_BLUEPRINT.md`) and optional UI companion (`DESIGN_SYSTEM_GUIDE.md`), plus a small audit utility script.

## Stack

- Markdown documentation
- Bash scripting (`collect-project-docs.sh`)
- Git-based workflow
- No runtime application/database in this repo

## Environment

- Version manager: n/a (no runtime application)
- Version file: n/a
- Lockfile: n/a
- Setup: n/a

## Commit Trailer Template

```text
Co-authored-by: [AI_PRODUCT_NAME] <[AI_PRODUCT_EMAIL]>
AI-Provider: [AI_PROVIDER]
AI-Product: [AI_PRODUCT_LINE]
AI-Model: [AI_MODEL]
```

## Autonomous Commit Identity

- GitHub Actions roadmap workflows in this repo should use stage-specific project-relative identities:
  - `Agent Blueprint Implementer <agent-blueprint-implementer@users.noreply.github.com>`
  - `Agent Blueprint Reviewer <agent-blueprint-reviewer@users.noreply.github.com>`
  - `Agent Blueprint Fixer <agent-blueprint-fixer@users.noreply.github.com>`
- Model attribution still belongs in `Co-authored-by` plus the `AI-*` trailers; the workflow-owned git author/committer should stay separate from model attribution.
- In downstream repos, rename these identities relative to the adopting project and optionally swap the email domain to one you control.

Template rules:
- `AI_PRODUCT_LINE`: `codex|claude|gemini|opencode`
- Product-line derivation:
  - Codex or ChatGPT coding agent -> `codex`
  - Claude Code -> `claude`
  - Gemini CLI -> `gemini`
  - OpenCode -> `opencode` (regardless of underlying provider/model, including z.ai)
- `AI_PROVIDER` and `AI_MODEL`: runtime-derived at commit time
- `AI_PRODUCT_NAME` and `AI_PRODUCT_EMAIL`: resolved from **model name** using tiered lookup (see `AGENT_BLUEPRINT.md` `[BP-WF-COMMIT]`):
  1. **Brand match** (case-insensitive against model name):
     - `codex` → `Codex <codex@users.noreply.github.com>`
     - `claude` → `Claude <claude@users.noreply.github.com>`
     - `gemini` → `Gemini <google-gemini@users.noreply.github.com>`
     - `glm` → `GLM <zai-org@users.noreply.github.com>`
  2. **Provider fallback** (no brand match — use provider's GitHub org):
     - OpenAI → `OpenAI <openai@users.noreply.github.com>`
     - Anthropic → `Anthropic <anthropics@users.noreply.github.com>`
     - Google → `Google <google-gemini@users.noreply.github.com>`
     - Zhipu → `Zhipu <zai-org@users.noreply.github.com>`
     - Mistral → `Mistral <mistralai@users.noreply.github.com>`
     - Meta → `Meta <meta-llama@users.noreply.github.com>`
     - DeepSeek → `DeepSeek <deepseek-ai@users.noreply.github.com>`
  3. **Unknown**: `{Provider Name} <{github-org}@users.noreply.github.com>` or `AI Agent <noreply@users.noreply.github.com>`
- Do not store filled runtime values in this file
- For multi-model commits, see `AGENT_BLUEPRINT.md` `[BP-WF-COMMIT-MULTI]` — add one `Co-authored-by` line per contributing model and comma-separate the other trailers

## Validation Commands

| Level | Command | When |
|-------|---------|------|
| 1 | `bash -n collect-project-docs.sh` | After script changes |
| 2 | `rg -n "^version:" AGENT_BLUEPRINT.md` | After blueprint edits |
| 3 | `rg -n "BP-CORE-01|BP-ALIGN-REPORT|BP-RM-DOR" AGENT_BLUEPRINT.md` | Before completing blueprint changes |
| 4 | `echo "N/A: no UI/e2e in this repository"` | Always |

## Execution Modes

Use one policy file for both paired local work and autonomous GitHub Actions runs. Shared repo rules always apply; runtime-specific rules override only where they differ.

### Shared Rules

- Canonical planning surface is `roadmap/`; treat the referenced work unit as the source of scope.
- Roadmap work unit filenames use 3-digit IDs in this repo: `[ID]-[slug].md`.
- Use the validation commands above when their trigger conditions apply.
- Prefer a staged validation model: fast agent-run checks before PR update, then repository PR workflows for the heavier validation suite.
- Keep changes minimal and focused on the requested work unit.
- `git status` and `git diff` are always allowed for change review.
- `rg`, `sed`, `cat`, `nl`, and `wc` are always allowed for inspecting docs and scripts.
- `bash -n collect-project-docs.sh` is allowed after script changes.

### Runtime: Interactive Local

- Default mode when working directly with the user in a local agent session.
- Require user confirmation before `git commit`.
- Require user confirmation before dependency install or upgrade.
- Require user confirmation before network calls or other external side effects.
- Ask before destructive actions or anything not clearly covered by the allowlist.
- It is acceptable to stop for clarification when scope is ambiguous.

### Runtime: Autonomous GitHub Actions

- Applies to workflow-driven agent runs such as roadmap execution in GitHub Actions.
- The workflow input, especially `roadmap_path`, identifies the work unit; the referenced roadmap file is the canonical brief.
- `implement` runs should execute the fast validation commands that fit inside the agent runtime before updating a branch or PR.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jgoodhcg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
