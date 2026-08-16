---
trigger: always_on
description: Instructions for a coding agent asked to verify a Beacon change end to end.
---

# beacon-sandbox: agent guide

Instructions for a coding agent asked to verify a Beacon change end to end.

This tool rents a disposable Linux sandbox, installs the Beacon under test, runs a **real Claude
Code session** inside it, and checks whether Beacon recorded what the agent actually did. Human
documentation is in `README.md` and on the docs site; this file is the operating manual.

## Always start here

```bash
cd beacon-sandbox
go run ./cmd/beacon-sandbox doctor                      # for a Linux run (the default)
go run ./cmd/beacon-sandbox doctor --provider github    # for a Windows run
```

`--provider` decides which prerequisites are checked, because they differ: a Linux run needs a
Modal account and a local Beacon build, a Windows run needs `gh` and a dispatchable workflow.
Running the wrong set reports failures for things the chosen provider does not use.

`doctor` is free, needs no sandbox, and catches the failure modes that otherwise appear
mid-run as something else entirely. Add `--fix` to let it download the collector binary.

If `doctor` reports FAIL, apply the printed `fix` and rerun it. Do not proceed to `run` with a
failing check — the run will fail later and more confusingly.

`doctor --json` gives machine-readable output with a top-level `ready` boolean.

### What you can fix yourself, and what you must not

Resolve these two directly:

```bash
cd cli/beacon && make build-linux-amd64                          # beacon_binary
cd beacon-sandbox && go run ./cmd/beacon-sandbox doctor --fix    # collector_binary (downloads it)
```

**Do not attempt the other two.** They need the user, and trying them yourself wastes a turn or
hangs outright:

- **`modal_auth`** — `modal token new` completes through an authenticated *web session*, so it
  opens a browser and blocks. Running it yourself hangs until timeout. Ask the user to run it,
  suggesting `! pip install modal && modal token new` so the output lands in the conversation. In
  CI, `MODAL_TOKEN_ID` and `MODAL_TOKEN_SECRET` work non-interactively instead.
- **`anthropic_credential`** — never invent, echo, or write a key. Ask which path the user wants:
  `ANTHROPIC_API_KEY`, `--api-key-command CMD`, or `--modal-secret NAME` (most secure; the value
  never enters this process, at the cost of the artifact leak check reporting *unverified*).

## Before spending money

A run costs roughly **$0.06 of sandbox time plus a few cents of Anthropic API usage**. That is
real money on the user's account, so:

- Tell the user what you are about to run and roughly what it costs.
- Use `--scenario <id>` while iterating. Only run the full suite when asked or before a PR.
- Never loop the suite unattended, and never use `--repeat` without being asked.
- If you only changed *what counts as correct*, use `verify` instead — it is free.

## Choosing what to run

| You changed | Run |
|---|---|
| Command capture / exporter tool handling | `--scenario s02-bash-command` |
| File read or write signals | `--scenario s03-file-write` or `s04-file-read` |
| Prompt, session, token, or cost capture | `--scenario s01-hello` |
| Approval or permission handling | `--scenario s07-denied-tool` |
| `endpoint install`, config paths, service startup | `--scenario i01-install-supervised` |
| The systemd backend, unit files, Linux system mode | `--scenario i02-install-systemd` |
| Anything Windows | `--provider github --scenario w00-probe` |
| Something broad, or preparing a PR | the whole suite (no `--scenario`) |

```bash
go run ./cmd/beacon-sandbox run --scenario s02-bash-command   # one scenario
go run ./cmd/beacon-sandbox run                               # the whole suite, ~30 min, ~$0.70
```

`run` with no `--scenario` is the suite command. Other flags: `--repeat N` to tell flaky from
broken, `--keep-sandbox` to leave the instance up for debugging.

## The trap that matters most

**If the change touches `collector-builder/`, the collector must be rebuilt.** Telemetry
normalization compiles into `beacon-otelcol`, not the `beacon` CLI. Verifying an exporter change
against a downloaded or previously built collector produces a passing run that proves nothing
about the change.

`doctor` warns about this (`collector_freshness`). When it does, rebuild:

```bash
go install go.opentelemetry.io/collector/cmd/builder@v0.121.0
cd collector-builder && mkdir -p dist
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 "$(go env GOPATH)/bin/builder" --config builder.yaml
cp dist/beacon-otelcol/beacon-otelcol dist/beacon-otelcol/linux_amd64/beacon-otelcol
```

This has already caused one wasted investigation: a stale collector made an
already-fixed bug still look broken.

`collector_freshness` covers three ways the binary can be wrong: uncommitted changes under
`collector-builder/`, a downloaded release binary with exporter changes committed since that
release, and a locally built binary older than its sources. It also reports a release tag the clone
cannot resolve as *unknowable* rather than fresh — if you see that, run `git fetch --tags` or
rebuild. Rebuilding locally clears the warning; provenance is tied to the binary on disk, so the
warning can always be satisfied by doing what it asks.

## Reading the result

Three outcomes, and the third is what makes the tool trustworthy:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Asymptote-Labs/agent-beacon](https://github.com/Asymptote-Labs/agent-beacon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
