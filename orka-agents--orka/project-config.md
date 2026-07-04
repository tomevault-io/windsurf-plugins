---
trigger: always_on
description: Orka is a Kubernetes-native task execution platform that manages Jobs and Pods for container tasks and AI agent tasks.
---

# AGENTS.md

Orka is a Kubernetes-native task execution platform that manages Jobs and Pods for container tasks and AI agent tasks.

## Constraints

- **No secrets** — never commit, log, or print API keys, tokens, or credentials. Use Kubernetes Secrets or env vars.
- **No binaries in repo** — build artifacts go in `bin/` (gitignored) or CI release pipelines.
- **Scope discipline** — implement exactly what's asked, nothing more.
- **Pre-land/pre-commit code changes**: use `$autoreview` until no accepted/actionable findings remain, unless equivalent manual review already done, trivial/docs-only, or user opts out.
- **Git push discipline** — after making a change, push to the current branch when it is not `main`; never push directly to `main`.
- **Post-PR closeout** — after creating or updating an agent-authored PR, use `$pr-closeout` until current CI/review blockers are resolved, unless the human opts out or the PR is intentionally draft/WIP.

## Continuous Review

For non-trivial code changes, run `$autoreview` (`.agents/skills/autoreview/SKILL.md`) before final/commit/ship and keep going until there are no accepted/actionable findings, unless the change is trivial/docs-only, equivalent manual review already happened, or the human opts out.

- Treat review output as advisory: verify every finding against the real code path before changing code.
- If review-triggered fixes change code, rerun focused tests and rerun `$autoreview`.
- Format before review when formatting can move line locations; focused tests and review may run in parallel only after formatting is stable.

## GitHub PRs & Issues

Before creating or updating a GitHub PR or issue body for agent-authored work, read `$agent-transcript` (`.agents/skills/agent-transcript/SKILL.md`) if available and check for a safe local session transcript.

- If a high-confidence transcript is found, ask exactly: "Include a redacted agent transcript? It helps reviewers and can make the PR easier to prioritize. I can open a local preview first."
- Never upload raw logs. Include an `## Agent Transcript` section only after human approval and only with a sanitized, scoped transcript.
- Drop system/developer prompts, reasoning, raw tool outputs, env, cookies, tokens, auth URLs, secrets, broad local paths, and unrelated session turns.
- If no safe transcript exists or the human declines, continue without a transcript and do not add a placeholder section.

## PR Closeout

After creating or updating an agent-authored PR, use `$pr-closeout` (`.agents/skills/pr-closeout/SKILL.md`) by default, like `$autoreview` is used before landing. Resolve merge conflicts, fix failing CI, address or push back on unresolved review threads, reply on GitHub and resolve addressed comments, push the non-main PR branch, and repeat until current CI is green and no unresolved actionable review threads remain. Skip only when the human opts out, the PR is intentionally draft/WIP, or the remaining blocker is external/human-only. Do not merge or enable auto-merge unless explicitly asked.

## Build & Test

```bash
make manifests          # Regenerate CRDs (after editing *_types.go or markers)
make generate           # Regenerate Go types
make build              # Build (includes UI)
make test               # Run tests
make lint-fix           # Lint and fix
make docker-build-all   # Controller, AI/general workers, harness wrapper image
make deploy IMG=<registry>/orka:tag HARNESS_WRAPPER_IMG=<registry>/agent-harness-wrapper:tag
```

UI: `cd ui && bun install && bun run dev` (dev server on :5173). See @website/docs/development/development.md for full commands.

For testing against a local Kubernetes cluster, use the `$kindctl` skill to manage repo/worktree-scoped kind clusters without touching the global kubeconfig.

To stand up a reverse proxy for Anthropic/Gemini/OpenAI-compatible clients, use the `$vekil-reverse-proxy-deploy` skill. When it falls back to GitHub Copilot device-code login, surface the login code and URL to the user and wait for their confirmation before continuing — never complete the login on their behalf.

To stand up an execution-workspace provider on a local kind cluster for evaluation, use the `$agent-sandbox-deploy` skill (kubernetes-sigs agent-sandbox; pairs with `$kindctl` for the cluster and `$orka-kind-deploy` for the controller) or the `$agent-substrate-deploy` skill (Agent Substrate; owns its own gVisor kind cluster, so it is not hosted on a `$kindctl` cluster). Both are local/kind eval only — Orka does not install or manage these providers in production — and both surface the `$vekil-reverse-proxy-deploy` device-code login to the user for confirmation rather than completing it.

## Verification

Run after every change:

```bash
make manifests generate          # After *_types.go or marker edits
make lint-fix && make test       # After any *.go edits
cd ui && bun run lint && bun run test  # After UI edits
bash -n scripts/*.sh                  # After shell script edits
go run github.com/rhysd/actionlint/cmd/actionlint@latest .github/workflows/<workflow>.yml  # After workflow edits
```

Single test: `go test ./internal/api/ -run TestHandlerName -v`

## Auto-Generated — Do NOT Edit


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orka-agents/orka](https://github.com/orka-agents/orka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
