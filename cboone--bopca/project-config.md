---
trigger: always_on
description: <!-- KEEP THIS SECTION UNDER 4000 CHARS - Copilot only reads first ~4000 -->
---

# GitHub Copilot Instructions

## PR Review Checklist (CRITICAL)

<!-- KEEP THIS SECTION UNDER 4000 CHARS - Copilot only reads first ~4000 -->

### Known Patterns - Do Not Flag

**Verify source before flagging**: Line numbers change; check actual code.

- **Agent registration**: All agents registered in `builtin.go:init()` - verify before claiming unregistered
- **Interface signatures**: Verify interfaces in source; plan docs show design history, not final code
- **Containerfile markers**: `AGENT_INSTALL_MARKER` matches exactly - verify before claiming mismatch
- **output.Name()**: Styles identifiers including display names per output.go docs
- **macOS 26 (Tahoe)**: Valid future version (today is 2026)
- **Go 1.23.0 format**: Patch version in go.mod is valid
- **install.sh checks**: Writability checked BEFORE choosing directories
- **macOS-only testing**: Tests designed for macOS, not cross-platform CI; `sh` is always available, `TestHelperProcess` pattern unnecessary
- **Container TTY scripts**: Scripts in `config/` like `xclip-osc52.sh` run inside interactive container sessions where `/dev/tty` is always available; do not flag unconditional `/dev/tty` writes
- **EnhancedMockRunner setup**: `SetupContainerServiceRunning()` etc. configure mock state
- **Cobra help output**: `bopca help` vs `bopca --help` differ intentionally
- **Dry-run skips CheckRequirements**: Commands that use `skipRequirements` annotation and call `CheckRequirements()` in their own `RunE` intentionally skip it during dry-run because `CheckRequirements` uses real system checks (`lookupPath`) not mediated by the Runner interface, so `DryRunRunner` cannot intercept them
- **Containerfile sudo**: The `dev` user has passwordless sudo configured (`NOPASSWD:ALL` in `/etc/sudoers.d/dev`) with `Defaults:dev !env_reset` so `sudo` preserves environment variables for the dev user. The `!env_reset` is intentionally scoped to `dev` (not global) and must remain `!env_reset` (not `setenv`), because dev's sudo calls inside the container need automatic env preservation without requiring `-E`. Do not flag `sudo` commands in `RUN` steps under `USER dev` as potentially failing; sudo works without a TTY or password prompt in this container. Do not suggest further scoping or changing `!env_reset` to `setenv`
- **YOLO prompt default**: `defaultPromptYOLO` intentionally defaults to yes (enabled) for both empty input and read errors. This is a deliberate design choice; do not suggest defaulting to no for non-interactive stdin or adding TTY checks that change the default behavior
- **eval BOPCA_YOLO_INIT**: The `BOPCA_YOLO_INIT` env var is set exclusively by bopca's own Go code (`internal/container/args.go`) from `agent.YOLOShellInit()`, not from untrusted input. It injects shell aliases for YOLO mode in a controlled container environment. Do not flag this eval as a security risk or suggest file-based alternatives
- **Apple build flags**: `--memory`/`--cpus` valid for `container build`
- **Shell script naming**: `log_*` vs `print_*` intentional per-script
- **Code blocks in docs**: `||`, `|` in fenced code blocks are operators, not table syntax
- **Markdown tables in docs**: Tables in documentation files use standard single-pipe syntax (`| Column | ... |`). Do not flag these as having double leading pipes (`||`); verify the raw file content before reporting table syntax issues
- **Cobra command sorting**: `rootCmd.Commands()` sorts alphabetically by default via `commandSorterByName`; do not suggest explicit sorting of command lists obtained from `Commands()`
- **Status line test patterns**: In `statusline_test.go`, `ReplaceStdout(&buf)` prevents Bubble Tea from writing to real stdout; it is not meant to capture logging output. Tests assert on `isStatusLineActive()` to verify behavioral contracts, not on buffer contents. `stopStatusLine()` blocks on the `done` channel (with timeout) before returning, so assertions after it are safe. The `defer restore()` before `t.Cleanup` ordering is an intentional, consistent pattern across all status line tests
- **Pre-v1 breaking CLI changes**: This project does not preserve backwards compatibility until v1. Removing, renaming, or changing CLI flags is acceptable and intentional. Do not flag flag removal as a breaking change or suggest reintroducing removed flags for backwards compatibility
- **Commit messages are historical records**: PR descriptions list commit messages reflecting the code state when each commit was made. Later commits may change the implementation (e.g., removing a flag that an earlier commit tested). Do not flag commit messages or PR descriptions as inconsistent with the final code state; they document the development history, not the final result
- **Plan documents in docs/plans/**: Documents under `docs/plans/todo/` are proposed plans not yet implemented. The directory path communicates the document's status. Do not suggest adding "Not Yet Implemented" labels or status banners to plan documents
- **Scrut test expected output**: Scrut tests match actual command output verbatim. When expected output contains unusual syntax (e.g., `./"` vs `.\"`), verify against real command output before flagging as a typo; the test reflects what the tool actually produces

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cboone/bopca](https://github.com/cboone/bopca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
