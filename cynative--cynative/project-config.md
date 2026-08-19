---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) and other coding agents when working
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) and other coding agents when working
with code in this repository. `CLAUDE.md` is a symlink to this file.

## Commands

All targets are wired through `make`. Lint and test rely on `go tool golangci-lint` (pinned via
`go.mod`), so no separate install is needed. `moq` is pinned the same way and `make generate`
writes the gitignored `*_mock_test.go` mocks. **Run `make generate` before
`go test ./internal/<pkg>` on a fresh checkout**, or the tests won't compile.

- `make check`: the full gate CI runs, `check-go` + `check-scripts`.
- `make check-go`: `mod-tidy-check` (a non-mutating `go mod tidy -diff`, so the gate fails on an
  untidy `go.mod`/`go.sum` instead of rewriting them) + generate + lint + shell-complexity +
  format-diff + test + `windows-build` (GOOS=windows amd64/arm64 cross-compile). 100%
  `go.mod`-pinned; the `mod-tidy-check` step is non-mutating but may consult the module cache,
  so the gate is not fully network-free; **the pre-commit hook runs this**.
- `make check-scripts`: `shellcheck` (all tracked `*.sh`) + PSScriptAnalyzer (every tracked
  `*.ps1`) + Pester unit tests (every tracked `test/*.Tests.ps1`) +
  `sh-test` (the POSIX `install.sh` unit tests, a `python3`-backed loopback smoke
  test of the `CYNATIVE_BASE_URL` download-base seam, its non-loopback-HTTP reject, and the
  fail-closed checksum-mismatch abort, the
  live-e2e guardrails unit tests, the connector-e2e orchestration unit tests, the shared
  release-gate invocation-contract and gate-assert unit tests, the llm-smoke workflow golden,
  the gate trusted-caller pin check, the release publish-gate pin check, the llm-smoke
  secret-reference pin (the sorted-unique `secrets.<NAME>` set in `llm-smoke.yaml` must be
  exactly the two api keys; the `release.yaml` job that calls the gate must forward exactly
  those two names and each as an **identity forward**, `NAME: ${{ secrets.NAME }}`; in
  **both** workflows, bracket-form `secrets[...]` and
  whole-object uses like `toJSON(secrets)` are rejected, as is any `secrets:` key whose
  value is the scalar `inherit`). The forwarding arm is the load-bearing one and the one no
  earlier version had: the exact-set arm counts references *inside* `llm-smoke.yaml`, so by
  itself it is satisfied by a caller that forwards `OPENAI_API_KEY: ${{ secrets.APP_PRIVATE_KEY }}`
  - the name the gate sees never changes, only the value does. It is scoped to the job whose
  `uses:` names the gate (matched on basename, `@ref` tolerated), so the other reusable calls
  in `release.yaml` keep their own grants; a `release.yaml` with no such job fails closed
  rather than passing vacuously. The matched target must also be **this repo's own** workflow
  (`./` or `cynative/cynative/`), since a basename says nothing about the owner: a call
  retargeted at `attacker/collector/.github/workflows/llm-smoke.yaml@main` would otherwise
  satisfy every arm while forwarding both api keys out of the repository. The checker (`scripts/ci/check-llm-smoke-secrets.py`,
  unit-tested by `test/llm-smoke-secrets.unit.test.sh`) **parses both workflows with
  PyYAML** (a `SafeLoader` subclass with the YAML 1.1 implicit scalar resolvers cleared, so
  `on:` and `yes:` stay distinct string keys instead of colliding on `True` and dropping
  whichever expression the first one held; `!!python/*` tags are still refused)
  rather than grepping the text, which is what makes the pin hold: comments carry
  no meaning, so prose can neither hide a reference nor invent one; `secrets: inherit` is
  matched structurally, so the next-line, folded, quoted, `!!str` and anchor/alias
  spellings all fail alike; and expression scanning is confined to `${{ }}` spans with
  Actions string literals blanked, so a `}` inside `format('{0}', ...)` cannot end a span
  early, `inputs.secrets` is not mistaken for the secrets context, and the word "secrets"
  in a step name or a shell line is not a match. Anything unresolved fails closed: a missing,
  unreadable, non-UTF-8 or unparseable file, and an unterminated `${{` span (Actions would
  reject that workflow, and guessing where it ended used to mint a phantom secret name out of
  a shell comment). Both tree walks carry a visited set, so a self-referencing anchor cannot
  recurse forever and an alias-amplification document cannot burn the CI job's timeout.
  It is still a tripwire, **not** an Actions expression
  evaluator: it reads the workflow as written, so it cannot follow a secret name assembled
  at runtime. The suite asserts each fixture is parseable YAML before using it, since the
  case that motivated this (#216) was a fixture no parser accepts, which pinned nothing.
  Then the Scoop-manifest and Homebrew-Formula
  renderers and both strict asset-digest lookups (`sha_for` over the manifest TSV,
  `sha_for_checksums` over `checksums.txt`; each must fail on a duplicate row rather than return
  the first match) unit tests, the release asset-set assertion's unit tests (the
  fail-closed-on-missing-digest branches plus the generate-mode artifact-type allowlist:
  Archive, Checksum and Signature, never Binary or Certificate), the release signing contract

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cynative/cynative](https://github.com/cynative/cynative) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
