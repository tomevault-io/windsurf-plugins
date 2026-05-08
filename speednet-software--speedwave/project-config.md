---
trigger: always_on
description: Security-first AI platform connecting Claude Code with external services (Slack, SharePoint, GitLab, Redmine, Mail, Calendar). Claude runs in a hardened, token-free container — all service credentials are isolated per-worker. Additional VM-level isolation on macOS (Lima) and Windows (WSL2); rootless user namespaces on Linux. Ships as a single installable app (.dmg, .exe, .deb) without Docker Desktop. Two interfaces: CLI (terminal) and Desktop (chat UI).
---

# Speedwave

Security-first AI platform connecting Claude Code with external services (Slack, SharePoint, GitLab, Redmine, Mail, Calendar). Claude runs in a hardened, token-free container — all service credentials are isolated per-worker. Additional VM-level isolation on macOS (Lima) and Windows (WSL2); rootless user namespaces on Linux. Ships as a single installable app (.dmg, .exe, .deb) without Docker Desktop. Two interfaces: CLI (terminal) and Desktop (chat UI).

## Key Architecture

- **SSOT: `crates/speedwave-runtime/`** — all Lima/WSL2/nerdctl logic. CLI and Desktop both import it as a Cargo dependency
- **SSOT: `mcp-servers/shared/`** — MCP protocol utilities shared by all servers
- **SSOT: `containers/compose.template.yml`** — container definitions. `render_compose()` generates per-project files
- **SSOT alignment:** `scripts/bundle-build-context.sh` IMAGES list must stay aligned with `crates/speedwave-runtime/src/build.rs` IMAGES constant
- **SSOT alignment:** `scripts/sign-bundled-binaries.sh` SIGN_TARGETS must stay aligned with `desktop/src-tauri/tauri.macos.conf.json` bundle.resources — every bundled Mach-O must be signed, and binaries using restricted platform APIs need entitlements plists in `desktop/src-tauri/entitlements/`
- **Per-project isolation:** `~/.speedwave/tokens/<project>/<service>/` (read-only mount), `speedwave_<project>_network` (isolated network)
- **ContainerRuntime trait:** `Box<dyn ContainerRuntime>` — implementations: `LimaRuntime`, `NerdctlRuntime`, `WslRuntime`
- **MCP Hub:** port 4000, the ONLY MCP server Claude sees. Hub has zero tokens.
- **IDE Bridge:** writes `~/.speedwave/ide-bridge/<port>.lock` on host, mounted as `~/.claude/ide/` in container
- **Config merge:** defaults -> repo `.speedwave.json` -> user `~/.speedwave/config.json` (highest priority). See ADR-011
- **Claude Code:** installed inside container by `entrypoint.sh` at start (Anthropic All Rights Reserved — cannot be bundled)
- If same logic appears in two places — extract it to `speedwave-runtime`

## Commands

**Use `Makefile` for all build/test/check operations.** Do not call cargo/npm directly.

```bash
make setup-dev      # first-time: check prerequisites + install all dependencies
make test           # run all tests (Rust + Angular + MCP + entrypoint + desktop)
make check          # lint + clippy + type-check + format
make check-all      # full quality gate: check + test + coverage + audit
make coverage-html  # generate HTML coverage reports and open in browser
make audit          # check dependencies for known vulnerabilities
make dev            # start desktop in dev mode (Tauri + Angular hot reload)
make build          # build everything
make fmt            # format all code
make status         # quick health check
```

Granular targets:

- **Test:** `test-rust`, `test-cli`, `test-angular`, `test-mcp`, `test-os`, `test-swift`, `test-desktop`, `test-e2e`, `test-entrypoint`, `test-desktop-build`, `test-e2e-desktop`, `test-e2e-all`, `setup-e2e-vms`
- **Build:** `build-runtime`, `build-cli`, `build-desktop`, `build-native-macos`, `build-os-cli`, `build-mcp`, `build-angular`, `build-tauri`
- **Check:** `check-clippy`, `check-desktop-clippy`, `check-fmt`, `check-mcp`, `check-mcp-lint`, `check-angular`, `check-angular-lint`
- **Coverage:** `coverage-rust`, `coverage-mcp`, `coverage-angular`
- **Audit:** `audit-rust`, `audit-mcp`, `audit-desktop`
- **Download:** `download-lima`, `download-nodejs`, `download-nerdctl-full`, `download-wsl-resources` (+ `clean-*` variants)
- **Other:** `lint`, `install-deps`, `install-hooks`, `clean`

## Git Workflow

```bash
git add <files>
git commit -m "..."
git push
```

- **PRs always target `dev`** — never open a PR directly to `main`
- **`dev` -> `main`:** always squash merge in GitHub UI. PR title must be a conventional commit (e.g. `feat(runtime): add logging`). See [RELEASING.md](RELEASING.md#why-squash-merge-matters)
- **`chore(...)` is NOT allowed as a PR title to `main`** — release-please ignores `chore` commits, so a `chore` squash merge would collapse all bundled `feat`/`fix` commits into an invisible release (no version bump). Allowed types for `dev → main`: `feat, fix, perf, refactor, docs, ci, test, build, style, revert`. `chore` remains valid for PRs to `dev`.
- **Backmerge (`main` -> `dev`):** automated via `backmerge.yml` on release publish. Resets dev to main (force-push) to prevent ghost commit accumulation. Falls back to regular merge PR if dev has new commits since the release
- **`merge-strategy-check.yml`** enforces conventional commit PR titles on PRs to `main` (release-please and backmerge PRs are exempt)
- Link commits to GitHub issues when they exist

### Merge strategy table

| PR direction                  | Strategy            | Enforced by                         |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [speednet-software/speedwave](https://github.com/speednet-software/speedwave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
