---
trigger: always_on
description: <!-- FOR AI AGENTS - Human readability is a side effect, not a goal -->
---

<!-- FOR AI AGENTS - Human readability is a side effect, not a goal -->
<!-- Managed by agent: keep sections and order; edit content, not structure -->
<!-- Last updated: 2026-03-23 | Last verified: 2026-03-23 -->

# AGENTS.md

**Precedence:** the **closest `AGENTS.md`** to the files you're changing wins. Root holds global defaults only.

## Commands (verified)
> Source: go.mod + CI (.github/workflows/ci.yaml) — CI-sourced commands are most reliable

<!-- AGENTS-GENERATED:START commands -->
| Task | Command | ~Time |
|------|---------|-------|
| Typecheck | `go build -v ./...` | ~15s |
| Format | `gofmt -w .` | ~5s |
| Test (single) | `go test -v -race ./path/to/pkg` | ~2s |
| Test (all) | `go test -v -race -short ./...` | ~30s |
| Build | `go build -v ./cmd/yaah` | ~15s |
| Install locally | `go install ./cmd/yaah` | ~15s |
<!-- AGENTS-GENERATED:END commands -->

> If commands fail, verify against go.mod or ask user to update.

## Workflow
1. **Before coding**: Read nearest `AGENTS.md` + check Golden Samples for the area you're touching
2. **After each change**: Run the smallest relevant check (typecheck → single test)
3. **Before committing**: Run full test suite if changes affect >2 files or touch shared code
4. **Before claiming done**: Run verification and **show output as evidence** — never say "try again" or "should work now" without proof

## File Map
<!-- AGENTS-GENERATED:START filemap -->
```
cmd/yaah/           -> CLI entry point (cobra root command)
pkg/catalog/        -> Skill catalog, bundles, discovery, and validation
pkg/harness/        -> Top-level orchestrator wiring all registries
pkg/hooks/          -> Hook registry + handler interface
pkg/hooks/handlers/ -> Built-in handlers (linter, command-guard, secret-scanner, comment-checker, session-logger)
pkg/mcp/            -> MCP server registry + provider interface
pkg/mcp/providers/  -> Built-in MCP providers (yaah, context7, pulumi)
pkg/lsp/            -> LSP registry + provider interface
pkg/lsp/providers/  -> Built-in LSP providers (gopls, pyright, typescript, csharp)
pkg/skills/         -> Skill registry + loader
pkg/skills/builtins/-> Built-in skill definitions (commit, pr, review)
pkg/agents/         -> Agent registry + loader
pkg/commands/       -> Custom CLI command registry
pkg/plugins/        -> Plugin registry
pkg/generator/      -> Per-agent config generators (claude, opencode, codex, copilot)
pkg/schema/         -> Settings types + defaults
pkg/session/        -> Session audit log store
pkg/gitcache/       -> Git-based caching for remote skills/agents
pkg/mcpserver/      -> Built-in MCP server runtime (stdio transport)
internal/cli/       -> CLI wiring (cobra commands, flag parsing)
docs/               -> Component deep-dive + configuration reference
website/            -> Static marketing site (auto-deployed to GitHub Pages)
.claude/skills/     -> Generated + remote skill definitions
.claude/agents/     -> Generated + remote agent definitions
.github/workflows/  -> CI (ci.yaml) + Pages deploy (deploy-pages.yaml)
```
<!-- AGENTS-GENERATED:END filemap -->

## Golden Samples (follow these patterns)
<!-- AGENTS-GENERATED:START golden-samples -->
| For | Reference | Key patterns |
|-----|-----------|--------------|
| Hook handler | `pkg/hooks/handlers/linter.go` | Implement `hooks.Handler` interface, register in `defaultHarness()` |
| MCP provider | `pkg/mcp/providers/yaah.go` | Implement `mcp.Provider` interface, return server config |
| LSP provider | `pkg/lsp/providers/gopls.go` | Implement `lsp.Provider` interface, return LSP config |
| Generator | `pkg/generator/claude.go` | Implement `Generator` interface, emit agent-native config |
| CLI command | `internal/cli/generate.go` | Cobra command wired in `internal/cli/root.go` |
| Skill definition | `pkg/skills/builtins/commit.go` | Skill struct with Name, Description, Source fields |
| Catalog entry | `pkg/catalog/entries.go` | CatalogEntry struct with ID, Category, Tags, Risk |
| Plugin | `pkg/plugins/codex.go` | Implement `plugins.Plugin` + `MarketplacePlugin` interfaces |
<!-- AGENTS-GENERATED:END golden-samples -->

## Utilities (check before creating new)
<!-- AGENTS-GENERATED:START utilities -->
| Need | Use | Location |
|------|-----|----------|
| Git clone/cache remote repo | `gitcache.CloneOrUpdate()` | `pkg/gitcache/` |
| Session audit logging | `session.Store` | `pkg/session/` |
| Settings defaults | `schema.DefaultSettings()` | `pkg/schema/` |
| Hook event dispatch | `harness.HandleHookEvent()` | `pkg/harness/` |
| MCP server stdio | `mcpserver.Serve()` | `pkg/mcpserver/` |
<!-- AGENTS-GENERATED:END utilities -->

## Heuristics (quick decisions)
<!-- AGENTS-GENERATED:START heuristics -->
| When | Do |
|------|-----|
| Adding a new hook handler | Create in `pkg/hooks/handlers/`, register in `internal/cli/generate.go` default harness |
| Adding a new MCP provider | Create in `pkg/mcp/providers/`, register in default harness |
| Adding a new plugin | Create in `pkg/plugins/`, implement `Plugin` interface, register in default harness |
| Adding a new agent target | Create generator in `pkg/generator/`, add to generate command |
| Adding a CLI command | Create cobra command in `internal/cli/`, wire in `root.go` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dirien/yet-another-agent-harness](https://github.com/dirien/yet-another-agent-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
