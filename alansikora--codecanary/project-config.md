---
trigger: always_on
description: AI-powered code review for GitHub pull requests.
---

# CodeCanary

AI-powered code review for GitHub pull requests.

## Project structure

```
cmd/
  review/          # Main binary — review CLI + setup wizard
    main.go        # Entry point
    cli/           # Cobra commands
      root.go      # Root "codecanary" command
      review.go    # codecanary review <pr>
      findings.go  # codecanary findings <pr> — fetch bot findings for the review skill
      reply.go     # codecanary reply --url <comment-url> --body <text> — post a reply on a review thread
      install_skill.go # codecanary install-skill — write embedded Claude skill to disk
      setup.go     # codecanary setup [local|github]
      auth.go      # codecanary auth [status|delete]
internal/
  review/
    runner.go            # Core review pipeline — single Run() entry point
    config.go            # Config loading, validation, defaults
    # Provider layer (LLM abstraction)
    provider.go          # ModelProvider interface + factory registry
    provider_anthropic.go
    provider_openai.go
    provider_openrouter.go
    provider_claude.go   # Claude CLI wrapper
    provider_compat.go   # Shared types for OpenAI-compatible APIs
    pricing.go           # Token-based cost estimation
    # Platform layer (environment abstraction)
    platform.go          # ReviewPlatform interface
    platform_github.go   # GitHub Actions implementation
    platform_local.go    # Local CLI implementation
    # Supporting modules
    prompt.go            # Prompt building (review, incremental, per-thread)
    findings.go          # Finding parsing, filtering, result structures
    triage.go            # Thread classification + parallel LLM evaluation
    formatter.go         # JSON/Markdown/Terminal output formatting
    usage.go             # Token tracking, budget checking
    github.go            # GitHub API calls (fetch threads, post reviews)
    comments.go          # PR review comment fetch + finding marker parser + review-check watcher
    local.go             # Local diff & git operations
    state.go             # Local state persistence
    docs.go              # Project doc discovery
  credentials/     # Credential storage (keychain with file fallback)
    keyring.go     # Store/Retrieve/Delete — keychain first, ~/.codecanary/credentials.json fallback
  skills/          # Claude Code skills embedded in the binary via //go:embed
    skills.go      # Exports CodecanaryFix() returning the skill body
    codecanary-fix/SKILL.md  # Canonical skill source (duplicated at .claude/skills/codecanary-fix/SKILL.md; parity enforced by skills_test.go)
  setup/           # Setup wizard logic (huh forms)
    forms.go       # Shared huh form components
    validate.go    # API key validation via test calls
    guidance.go    # Token/permissions guidance text
    workflow.go    # GitHub Actions workflow template
    local.go       # RunLocal() — local setup flow
    github.go      # RunGitHub() — GitHub Actions setup flow
  auth/            # OAuth PKCE flow, GitHub App installation
telemetry/         # Telemetry domain (anonymous usage analytics)
  worker/          # Cloudflare Worker — telemetry ingestion (TypeScript)
  dashboard/       # Cloudflare Pages — internal analytics dashboard (vanilla JS + Chart.js)
oidc/              # OIDC domain
  worker/          # Cloudflare Worker — OIDC token exchange proxy (TypeScript)
action.yml         # GitHub Action definition (composite action)
install.sh         # Downloads and installs codecanary binary permanently
.claude/
  skills/
    codecanary-fix/ # Claude Code skill — drives review→fix→push loop using `codecanary findings` + `codecanary reply`
```

## Binary

- **`codecanary`** — single binary for reviews, setup, and credential management. Installed locally via `install.sh`, also used by the GitHub Action.

## Build

```sh
go build ./cmd/review    # builds codecanary
```

Version is set via ldflags: `-X main.version=v{version}`

## Lint

```sh
golangci-lint run ./...
```

All code must pass `golangci-lint` with default linters (errcheck, staticcheck, etc.). Run this before committing.

## Key dependencies

- `spf13/cobra` — CLI framework
- `charmbracelet/huh` — terminal form builder (setup wizard)
- `zalando/go-keyring` — OS keychain (with file-based fallback for systems without one)
- `bmatcuk/doublestar` — glob pattern matching for ignore rules
- `gopkg.in/yaml.v3` — config parsing
- `golang.org/x/term` — terminal detection

## Architecture

### Core principle: adapters keep the engine agnostic

The review engine (`runner.go`) is provider- and platform-agnostic. It depends only on two interfaces — never on concrete GitHub APIs, LLM SDKs, or environment-specific logic. All environment and provider specifics live behind adapters.

### Provider layer — `ModelProvider` interface (`provider.go`)

Abstracts LLM invocations. The core engine calls `provider.Run(ctx, prompt, opts)` and gets back text + usage metadata. It never knows which LLM backend is being used.

**Implementations**: `anthropic`, `openai`, `openrouter`, `claude` (CLI).
**Selection**: factory registry in `provider.go` — `NewProviderForRole(mc, env)` returns the right implementation based on `mc.Provider`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alansikora/codecanary](https://github.com/alansikora/codecanary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
