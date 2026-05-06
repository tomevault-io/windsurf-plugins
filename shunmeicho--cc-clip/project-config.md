---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Does

cc-clip bridges your local Mac/Windows clipboard to a remote Linux server over SSH, so `Ctrl+V` image paste works in remote Claude Code and Codex CLI sessions. It uses an xclip/wl-paste shim that transparently intercepts only Claude Code's clipboard calls, an X11 selection owner bridge for Codex CLI which reads the clipboard via X11 directly, and an SSH notification bridge that forwards Claude Code hook events (stop, permission prompt, idle) back to the local machine as native notifications.

```
Claude Code path:
  Local Mac clipboard → pngpaste → HTTP daemon (127.0.0.1:18339) → SSH RemoteForward → xclip shim → Claude Code

Codex CLI path (--codex):
  Local Mac clipboard → pngpaste → HTTP daemon (127.0.0.1:18339) → SSH RemoteForward → x11-bridge → Xvfb CLIPBOARD → arboard → Codex CLI

Notification path:
  Claude Code hook → cc-clip-hook (stdin JSON) → POST /notify via tunnel → classifier → dedup → DeliveryChain → native notification
```

## Build & Test Commands

```bash
make build                          # Build binary with version from git tags
make test                           # Run all tests (go test ./... -count=1)
make vet                            # Run go vet
go test ./internal/tunnel/ -v -run TestFetchImageRoundTrip  # Single test
make release-local                  # Build bare binaries for all platforms (dist/), local testing only
```

Version is injected via `-X main.version=$(VERSION)` ldflags. The `version` variable in `cmd/cc-clip/main.go` defaults to `"dev"`.

### Release Process

Production releases use **goreleaser** via GitHub Actions (`.github/workflows/release.yml`). Push a version tag to trigger:

```bash
git tag v0.6.0
git push origin v0.6.0
# CI runs: test → contract check → goreleaser → published release with tar.gz + checksums
```

**NEVER release manually with `make release-local` + `gh release create`.** The install script (`scripts/install.sh`) expects goreleaser's naming convention (`cc-clip_{version}_{os}_{arch}.tar.gz`). Bare binaries from `make release-local` use a different naming scheme (`cc-clip-{os}-{arch}`) and will cause install script 404s. `make release-local` is for local cross-compilation testing only.

goreleaser config: `.goreleaser.yaml`. Release is published automatically (not draft).

## Architecture

### Data Flow

1. **daemon** (`internal/daemon/`) — HTTP server on loopback, reads Mac clipboard via `pngpaste`, serves images at `GET /clipboard/type` and `GET /clipboard/image`. Auth via Bearer token + User-Agent whitelist.
2. **tunnel** (`internal/tunnel/`) — Client-side HTTP calls through the SSH-forwarded port. `Probe()` checks TCP connectivity. `Client.FetchImage()` downloads and saves with timestamp+random filename.
3. **shim** (`internal/shim/template.go`) — Bash script templates for xclip and wl-paste. Intercepts two specific invocation patterns Claude Code uses, fetches via curl through tunnel, falls back to real binary on any failure.
4. **connect** (`cmd/cc-clip/main.go:cmdConnect`) — Orchestrates deployment via SSH master session: detect remote arch → incremental binary upload (hash-based skip) → install shim → sync token → verify tunnel. Supports `--force`, `--token-only` flags.
5. **ssh** (`internal/shim/ssh.go`) — `SSHSession` wraps a ControlMaster SSH connection. Single passphrase prompt; all subsequent `Exec()` and `Upload()` calls reuse the master.
6. **deploy** (`internal/shim/deploy.go`) — `DeployState` tracks binary hash, version, shim status on the remote. JSON file at `~/.cache/cc-clip/deploy-state.json`. `NeedsUpload()` / `NeedsShimInstall()` enable incremental deploys.
7. **pathfix** (`internal/shim/pathfix.go`) — Auto-detects remote shell (bash/zsh/fish) and injects `~/.local/bin` PATH marker into rc file with `# cc-clip-managed` guards.
8. **service** (`internal/service/launchd.go`) — macOS launchd integration: `Install()`, `Uninstall()`, `Status()`. Generates plist for auto-start daemon.
9. **xvfb** (`internal/xvfb/`) — Manages Xvfb virtual X server on remote. `StartRemote()` auto-detects display via `-displayfd`, reuses healthy instances, writes PID/display to `~/.cache/cc-clip/codex/`. `StopRemote()` verifies PID+command before killing.
10. **x11bridge** (`internal/x11bridge/`) — Go X11 selection owner using `github.com/jezek/xgb` (pure Go, no CGo). Claims CLIPBOARD ownership on Xvfb, responds to SelectionRequest events by fetching image data on-demand from the cc-clip HTTP daemon via SSH tunnel. Supports TARGETS negotiation, direct transfer, and INCR protocol for images >256KB.

### Notification Bridge

11. **session** (`internal/session/`) — Ring-buffer session store tracking last 5 image transfers per session ID. `AnalyzeAndRecord()` atomically assigns sequence numbers and detects duplicates by fingerprint. TTL-based cleanup via `RunCleanup()`.
12. **classifier** (`internal/daemon/classifier.go`) — `ClassifyHookPayload()` translates Claude Code hook JSON (notification, stop, etc.) into a unified `NotifyEnvelope`. Maps hook types to urgency levels: `permission_prompt`=2, `idle_prompt`=1, `stop_at_end_of_turn`=0.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShunmeiCho/cc-clip](https://github.com/ShunmeiCho/cc-clip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
