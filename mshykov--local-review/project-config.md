---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

local-review is a local, BYOK (Bring Your Own Key) AI code reviewer. It's a single Go binary that:
- Runs against git diffs (staged changes, commits, or branches)
- **v0.1+**: Supports multi-LLM parallel reviews (Claude, Gemini, Codex)
- **v0**: Single-LLM via OpenAI-compatible API endpoints
- Saves reviews to local storage and merges findings intelligently
- Ships with language-aware prompt packs for TypeScript, Go, Python, Rust, and more

Key constraints:
- **No vendor SDK dependencies** — keeps the binary small and portable
- **No telemetry** — privacy first
- **Git CLI integration only** — no go-git to avoid binary bloat
- **Hybrid CLI + API mode** — prefer free CLI tools, fallback to API if configured

## Multi-LLM Architecture (v0.5+)

### Design Decisions

**Multi-LLM is the default, single-LLM is the fallback**
- Default: every authenticated LLM CLI (claude, gemini, codex) runs in
  parallel; findings are merged into one consolidated report.
- Fallback: when *no* LLM CLI is authenticated, hit the configured
  `provider:` (any OpenAI-compat endpoint, including local Ollama).
- The v0.1 plan included an "API fallback when CLI auth fails" inside
  the multi-LLM path; that idea was abandoned with v0.5.x — `mode:
  cli|api` was removed from the schema, see do-not-merge/v06-fully-
  local-ollama-preset.md for the shape it would take if revived.

**Supported LLM CLIs**
1. **Claude CLI** — `npm install -g @anthropic-ai/claude-code` (auth: `claude login`)
2. **Gemini CLI** — `npm install -g @google/gemini-cli` (requires Node.js 20+)
3. **OpenAI Codex CLI** — `npm install -g @openai/codex` (auth via `codex login` with ChatGPT Plus, or `OPENAI_API_KEY` env var pay-per-token — usually cheaper for occasional use)

**CLI Invocation Patterns**
- Claude: stdin diff piped to `claude` with the review prompt
- Gemini: short instruction prompt via `-p`, prompt body / diff via stdin (gemini's `-p` content is appended to stdin in headless mode, sidestepping argv-size limits)
- Codex: `codex exec --output-last-message <tempfile>` with the prompt on stdin. We deliberately do NOT use `codex review --commit <sha>` (the dedicated subcommand) because it re-extracts the diff itself with codex's own settings, conflicting with the orchestrator's "extract once, fan out the same diff string to all agents" contract — that would mean codex sees a different diff than the others and consensus tags become apples-to-oranges.

**Version-probe failures**
`internal/cli/detector.go` reports an LLM as `Available=false` when the `--version` probe fails (e.g., a CLI changes its banner format and our regex misses). `local-review doctor` surfaces this as the "install broken" state with the resolved binary path, but the runner currently filters silently — an upstream banner change can shrink the active agent set without an error message. Loosen the regex if a real CLI breaks this; for now the doctor row is the diagnostic.

**Storage Structure**
Reviews saved to `.local-review/reviews/<sanitized-branch>/<commit>_<llm>_<version>.md`:
```
.local-review/
  reviews/
    feature-auth-fix/              # branch name sanitized (/ → -)
      abc123_claude_3.5.md
      abc123_gemini_0.40.md
      abc123_codex_0.128.md
      abc123_merged.md             # LLM-powered merge
      abc123_metadata.json         # run details: timestamps, exit codes
```

**Merge Strategy**
- Use LLM to deduplicate and consolidate findings from all reviews
- Merge LLM selection (in priority order):
  1. User's `--merge-with <llm>` flag
  2. User's config `merge.preferred_llm: claude`
  3. Automatic best-available: Claude > Codex > Gemini
- Default: automatic mode

**Error Handling**
- Parallel execution: continue if one LLM fails
- Log failures to metadata.json
- Include failure notes in merged.md
- Skip not-installed LLMs silently (only log installed ones)

**Configuration Schema (v0.5+)**
```yaml
# .local-review.yml — every field is optional; defaults are sensible.
llms:
  claude:
    # enabled: true              # default; omit to use auto-detect
    cli_path: claude             # path override; defaults to PATH lookup
    model: claude-opus-4-7       # passed via --model
    api_key_env: ANTHROPIC_API_KEY  # custom env var for the key

  gemini:
    cli_path: gemini
    model: gemini-2.0-flash
    api_key_env: GEMINI_API_KEY

  codex:
    # codex runs by default when authenticated. Set enabled: false to
    # opt out (e.g., to keep paid-tier billing predictable).
    cli_path: codex
    model: gpt-4
    api_key_env: OPENAI_API_KEY

merge:
  preferred_llm: auto            # 'auto' or specific LLM name
  deduplicate: true

storage:
  base_path: .local-review/reviews
```

The v0.1 schema had a `mode: cli|api` field per LLM. It was never
wired through to the runtime — the orchestrator always invoked CLIs —
and was removed in v0.5.x. Existing configs with stray `mode:` lines
still load (yaml.v3 silently ignores unknown fields).

**Command Structure (v0.5+)**
```sh
# Review — multi-LLM by default (every authenticated CLI runs in parallel and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mshykov/local-review](https://github.com/mshykov/local-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
