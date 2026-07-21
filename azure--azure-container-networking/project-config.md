---
trigger: always_on
description: Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.
---

# agents.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 0. Use Repo Skills

Before implementing, review relevant repo skills in `.github/skills/` and apply them when the task touches Go, APIs, CRDs, controllers, or platform code.

- Skill use is required when applicable, not optional decoration.
- Multiple skills may apply; combine them when the change crosses concerns.
- For Go work, start with the relevant `acn-go-*` skills and follow them.

## 0.5 Use Go Code Intelligence (LSP / gopls MCP)

This repo ships two Go code-intelligence integrations for Copilot CLI / IDE / cloud agents:

- `.github/lsp.json` — `gopls` as a Language Server (powers `/lsp`).
- `.mcp.json` — the official `gopls mcp` server (gopls v0.20+) exposing `go_search`, `go_symbol_references`, `go_package_api`, `go_file_context`, `go_workspace`, `go_diagnostics`, `go_vulncheck`, `go_rename_symbol`.
- `.github/gopls-mcp-instructions.md` — gopls' own model-instructions file describing the read/edit workflow. Treat it as required reading when working with Go in this repo.

**One-time setup per developer:** run `./.github/scripts/setup-go-tooling.sh` (installs `gopls`; it must end up on `PATH`). Then `/exit` and re-launch Copilot CLI.

When working with Go code you **must** prefer code-intelligence tools over plain text search:

- Symbol lookup (definition, references, implementations, type info, hover/docs, workspace/document symbols) → use the LSP or gopls MCP tools. Do **not** grep for these.
- Call sites of a function, method, or interface → LSP `references` or `go_symbol_references` (and `implementations` for interfaces). Not grep.
- "Where is this defined?" → LSP `definition` or `go_search`. Not grep + view.
- Package surface area → `go_package_api` or LSP document symbols. Not view + scan.
- Build / type errors → `go_diagnostics`. Not `go build` parsing.
- Run `/lsp` and `/mcp` in Copilot CLI to confirm the servers are healthy. If the relevant server is unavailable, fix it before falling back to text search.

`grep` and `view` remain appropriate for: raw string/comment search, non-Go files (YAML, Markdown, shell, manifests), build/CI configs, log lines, and any case the LSP/MCP demonstrably cannot answer. State the reason briefly when falling back.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 5. PR Workflow

**All local agent work MUST happen in a dedicated git worktree, never in the shared repo root.** The repo root is shared across concurrent agent sessions; mutating it causes parallel branches, working trees, and build artifacts (`output/`, `bin/`) to collide. From the repo root, create a worktree under the active session folder before touching files:

```bash
SLUG=<short-task-slug>                  # e.g. cns-shutdown
BRANCH=<branch-name>                    # e.g. fix/cns-shutdown
WT="$HOME/.copilot/session-state/$COPILOT_AGENT_SESSION_ID/files/worktrees/$SLUG"
git fetch origin master --quiet
git worktree add -b "$BRANCH" "$WT" origin/master
cd "$WT"                                # then run all subsequent commands here
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/azure-container-networking](https://github.com/Azure/azure-container-networking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
