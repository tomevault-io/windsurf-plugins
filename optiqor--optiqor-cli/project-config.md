---
trigger: always_on
description: The open-source CLI: `@optiqor/cli`. Apache-2.0. Standalone repo so it stays independently auditable, which is the entire reason it doesn't live in the proprietary monorepo. Strategy reference: `docs/open_source_cli_playbook.md` in the Optiqor org docs (not public).
---

# optiqor-cli — Claude Conventions

The open-source CLI: `@optiqor/cli`. Apache-2.0. Standalone repo so it stays independently auditable, which is the entire reason it doesn't live in the proprietary monorepo. Strategy reference: `docs/open_source_cli_playbook.md` in the Optiqor org docs (not public).

This file is the operating manual. Read it before writing or reviewing code.

---

## Stack

- Go 1.24, single module: `github.com/optiqor/optiqor-cli`
- Cobra for command parsing, charmbracelet/lipgloss for terminal rendering
- Pure stdlib for the rest. No HTTP framework, no logger library, no DI framework.
- npm wrapper (`@optiqor/cli`) downloads the platform-specific Go binary at install time
- GoReleaser for cross-platform builds (linux + darwin, amd64 + arm64)
- Cosign for release artifact signing
- SHA-256 verified on download (in progress, see open issues)

If you reach for a new dependency, justify it in the PR. Adding a transitive dep widens the audit surface of an Apache-2.0 binary that runs on customer laptops.

---

## Hard rules (non-negotiable)

These exist because the OSS funnel breaks if any of them slip. They are not preferences.

1. **No LLM calls.** The CLI is a deterministic rule engine. The Sonnet/Opus/Haiku-driven Apply Fix flow lives in the proprietary backend. If you find yourself wanting to call an LLM from here, the answer is "POST to the SaaS sandbox endpoint instead and let it return the LLM-enriched response."

2. **No telemetry by default.** Zero-config install must not phone home. The only opt-in network egress is `--share`, which POSTs a sanitised analysis to `optiqor.dev/r/<hash>`. Every new network call needs an explicit user opt-in and must consult `offlineMode()` in `cmd/optiqor/main.go`.

3. **Accuracy disclosure in every output.** Every renderer (text, JSON, HTML, roast) emits the mandatory disclosure: `Sandbox accuracy: ±40%. Install the Optiqor agent for exact numbers (optiqor.dev/get).` The string is canonical. The honesty is the whole pitch; don't water it down.

4. **No proprietary backend code may be imported.** `go.mod` must never reference `github.com/optiqor/optiqor`. The CLI is independently buildable, auditable, licensable. CI greps imports on every PR.

5. **`pkg/` is the stable public API.** External programs may import it, including the proprietary backend (which depends on `pkg/rules` and `pkg/parser`). Breaking changes go through semver and a deprecation notice. New detectors land in `pkg/rules` first; the backend follows automatically via `go get -u`.

6. **`internal/` is private.** Refactor freely. `internal/{analyze,render,share,config,roast}` is CLI-side composition that stays out of the public API.

---

## Distribution

- Primary: `npm install -g @optiqor/cli` or `npx @optiqor/cli analyze ./chart`. The npm `postinstall` script downloads the right binary from GitHub Releases.
- Secondary: `go install github.com/optiqor/optiqor-cli/cmd/optiqor@latest`.
- Releases are GoReleaser-built (`-trimpath`, version ldflag from `git describe`) and Cosign-signed.
- We do **not** publish to Homebrew or Cargo in Year 1. npm is where the platform engineers are.
- Windows is out of scope. `.goreleaser.yaml` and `package.json` both reflect this. Adding Windows handling needs the OSS-scope exclusion in the playbook removed first.

---

## Commands

12 commands phased over 12 months. Year 1 ships the deterministic core.

| Command | Status | Purpose |
| --- | --- | --- |
| `analyze <chart>` | shipped | Primary entry point. Reads a Helm chart dir or `values.yaml`, prints findings. |
| `demo` | shipped | Runs analysis on a bundled chart so `npx @optiqor/cli demo` shows output without input. |
| `diff <a> <b>` | shipped | Cost delta between two values files. |
| `score <chart>` | shipped | 0-100 efficiency score (qualitative band Year 1, numeric Year 2). |
| `audit <chart>` | shipped | Security-focused subset of `analyze`. Stricter `--fail-on` defaults. |
| `watch <chart>` | stub | Re-runs `analyze` on file change. Phase 2+. |
| `compare <a> <b>` | stub | Side-by-side renderer. Phase 2+. |
| `--roast` (flag) | shipped | Humorous tone on `analyze`. Findings stay accurate. |

Stubs return "not yet implemented" and exit 1. The surface is registered so the command name is locked in.

---

## Output

The CLI's output is the product. Treat it with the same care as code.

### Stream discipline

- `stdout` is data: the report, JSON, or HTML when piped.
- `stderr` is everything else: status, warnings, errors, share URLs, the accuracy disclosure when output is redirected.
- Mixing the two breaks pipelines. `optiqor analyze ./chart | jq` must always work without `2>/dev/null` tricks.

### Color

- Respect `NO_COLOR` (de-facto standard, https://no-color.org).
- Respect `--no-color`.
- Auto-detect via `isatty(stderr)`. Don't auto-color when piped.

### Exit codes

- `0` success, no findings at or above `--fail-on`
- `1` runtime error (parse failure, file not found, invalid flag)
- `2` findings at or above `--fail-on` severity


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [optiqor/optiqor-cli](https://github.com/optiqor/optiqor-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
