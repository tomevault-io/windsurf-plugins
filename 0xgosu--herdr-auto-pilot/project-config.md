---
trigger: always_on
description: Herd Auto Prompter (**hap**) — a Go plugin for the herdr terminal multiplexer that
---

# CLAUDE.md

Herd Auto Prompter (**hap**) — a Go plugin for the herdr terminal multiplexer that
watches every agent pane, auto-answers when a learned rule is confident, and escalates
to the operator (or a local LLM CLI) when not. `CONTRIBUTING.md` has the full ground
rules; this file is the day-to-day working reference.

**How to read the architecture rules.** Each names the identifier that implements it and
states which way it must fail. The full rationale — mechanism, measured numbers, the
incident that produced it — is the doc comment on that identifier, which is usually
richer than what is here; go read it before changing anything the rule covers. What this
file adds is reach: the hazard is normally in a file you had no reason to open, and the
**test traps** are the reason each of these regressions shipped green. Find the guarding
tests with `grep -rn "func Test<Topic>" --include=*_test.go`.

## Skills (`.claude/skills/`)

Prefer these for how-to detail.
- **`herdr`** — drive herdr from inside it (workspaces, tabs, panes, agents, waits).
- **`hap`** — operate the plugin via its CLI: status, tasks, escalations, config, safety
  rules, task sources.
- **`hap-development-local`** — the local dev loop: link the working tree, rebuild,
  hot-swap the daemon (`hap daemon --ensure`), live-test against a real agent.

The hap skill also ships in the binary: `hap --skill` prints it,
`hap skill install <claude|codex|agy|agents>...` (or the TUI Config tab) installs it.

## Build, test, lint

The semantic matcher links native code (llama.cpp via CGO, FAISS behind bleve's `vectors`
tag), so **the native deps are needed once** and the `vectors cpu` tags always — a build
without both fails to link.

```sh
bash scripts/check-submodule-gitlink.sh        # submodule must be a gitlink, not a symlink (#265)
bash scripts/setup-native.sh                   # one-time: submodules + llama-go libs + FAISS → /usr/local/lib
go build -tags "vectors cpu" ./...             # CGO; needs a C/C++ toolchain
go test -tags "vectors cpu" ./... -count=1     # what CI runs
gofmt -l . | grep -v submodule && go vet -tags "vectors cpu" ./...
golangci-lint run --build-tags "vectors,cpu"
```

- The real-model embedder test skips unless `models/all-minilm-l6-v2-q8_0.gguf` exists
  (download once from the HF repo in `release.yml`, or set `HAP_TEST_EMBED_MODEL`).
- Golden classifier fixtures: `internal/classify/testdata/`; regenerate with
  `UPDATE_GOLDEN=1 go test ./internal/classify/` and review the diff.
- Run the full suite before every commit that touches Go code.
- Profiling (opt-in, any verb): `HAP_PROFILE_DIR=<dir> [HAP_PROFILE_SECONDS=60] hap daemon --restart` writes
  ROLLING `<verb>-<pid>.cpu.pprof` / `.heap.pprof` windows (`internal/profiling`) — the files are always the latest
  complete window, so an idle daemon hours in can be read with `go tool pprof -top bin/hap <file>`. The detached
  daemon inherits the environment; nothing is written or listened on when the variable is unset. Each window also
  writes `<verb>-<pid>.mem.txt` (Go memory classes, plus `/proc/self/status` on Linux): the heap profile sees only LIVE Go
  objects, a few MB of a resident set mostly made of file-backed pages, freed-but-unreturned Go heap and native
  memory (Turso engine, FAISS, llama.cpp) — "go resident" is what to subtract from RssAnon to find the native part.
- Pipeline smoke test (fake herdr → real daemon → real LLM CLI):
  `go build -o /tmp/e2e ./e2e_harness && /tmp/e2e <short-dir> <hap-bin> <config-dir> <state-dir>`.

## Local integration suite (real herdr + claude)

`test/integration/` drives an **actual running herdr** (and, with `HAP_ITEST_CLAUDE=1` /
`HAP_ITEST_CODEX=1` / `HAP_ITEST_AGY=1`, a real agent CLI), gated by the `integration` build tag so
`go test ./...` and CI never run them. Each case **skips** (never fails) when its
dependency is absent.

```sh
go test -tags integration ./test/integration/ -v                    # from inside herdr, or set HERDR_BIN_PATH
HAP_ITEST_CLAUDE=1 go test -tags integration ./test/integration/ -v -timeout 20m  # spends tokens
HAP_ITEST_AGY=1 go test -tags integration ./test/integration/ -run TestRealAgy -v  # Gemini tokens
go test -tags "integration vectors cpu" ./test/integration/ -v      # + the real-model semantic case
```

**Recommended: run this once after finishing any feature**, before the PR — the unit suite
fakes herdr, so only this catches real CLI-shape drift.

Three traps, each of which already cost a shipped regression:

- **Anything asserting a CONFIRM must run its own daemon; an `App` with no `DaemonInfo` is
  the trap.** `AssessDaemonHealth` derives `Running` from `DaemonInfo`, so nil reads as "no
  daemon" whatever is actually up and `requireLiveDaemonFor` refuses while naming a daemon
  that IS running. `testDaemon.App` is the one correct wiring — and it deliberately holds NO
  herdr adapter, so keystrokes landing at all is the proof delivery went through the daemon.
  Never point such a test at the operator's live store: under turso that pushes scratch rows
  to their cloud database.
- **A scratch pane the DAEMON will classify must SCROLL.** `pane read --source recent`
  returns EMPTY for a pane whose output still fits on screen (verified live, herdr 0.8.2);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xGosu/herdr-auto-pilot](https://github.com/0xGosu/herdr-auto-pilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
