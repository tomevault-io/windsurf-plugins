---
trigger: always_on
description: These guidelines apply to the entire repository. Paths below are relative to the
---

# Agent guidelines

These guidelines apply to the entire repository. Paths below are relative to the
repository root.

## Task completion checks

Before finishing each task, format and fix lint issues in its changed files. Run
from the repository root; replace `<files>` with explicit quoted paths of that
row's type. Skip untouched types and deleted files; preserve unrelated files and
user changes, applying fixes manually if crate-wide tools would change them.

<!-- markdownlint-configure-file { "MD013": { "tables": false } } -->

| Files                                 | Format / lint fix commands                                                                                                     |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Go                                    | `gofmt -w <files>`; `(cd <module> && go vet ./...)` (fix diagnostics manually)                                                 |
| Rust (run inside each affected crate) | `cargo fmt --all`; `cargo clippy --locked --all-targets --fix --allow-dirty --allow-staged -- -D warnings`; `cargo fmt --all`  |
| JS / TS / JSX / TSX / MJS / CJS       | `bunx oxlint@1.19.0 --fix --deny-warnings <files>`; `bunx prettier@3.6.2 --write <files>`                                      |
| JSON / JSONC / CSS / HTML / YAML      | `bunx prettier@3.6.2 --write <files>`                                                                                          |
| Markdown                              | `bunx prettier@3.6.2 --write --prose-wrap always <files>`; `bunx --package markdownlint-cli@0.45.0 markdownlint --fix <files>` |

Go modules: `core`, `convo`, `contracts`. Rust crates: `apps/desktop/src-tauri`,
`apps/privacy-worker`, `apps/classifier-worker`. If Tauri needs staged sidecars,
run `make desktop-sidecar`. For desktop TS, also run
`(cd apps/desktop && bun run typecheck)`; it does not replace linting.

Recheck after fixing: `gofmt -l <files>` must print nothing; rerun `go vet`; use
`cargo fmt --all -- --check`; rerun Clippy without
`--fix --allow-dirty --allow-staged`, Oxlint/markdownlint without `--fix`, and
Prettier with `--check` instead of `--write`. Finish with `git diff --check` and
the task's required tests. Report unavailable tools or remaining failures.

## Upstream forwarding identity

AstrLink is an API gateway. Requests sent to upstream providers must not
identify AstrLink as the forwarding client.

- Do not inject AstrLink branding into upstream headers (including `originator`,
  `User-Agent`, `Via`, `X-Powered-By`, and `X-AstrLink-*`), URL parameters,
  generated request IDs, metadata, system prompts, or request bodies.
- Keep gateway-owned headers local; strip the reserved `X-AstrLink-*` namespace
  before HTTP forwarding and WebSocket handshakes, including target overlays.
- When a provider requires a client identity, reuse its shared identity policy
  and keep related fields consistent. Codex, Claude, and Grok subscription
  identity enforcement defaults to on, with independent persisted controls in
  Routing. Neither mode may introduce an AstrLink originator or User-Agent.
- Apply this rule to inference, retries, protocol conversion, model discovery,
  connection tests, OAuth/device authorization, token refresh, and quota/profile
  requests. Verify the final outgoing request, not only intermediate headers.
- Preserve caller-authored prompts, files, and tool schemas. Do not remove or
  rewrite user content merely because it mentions AstrLink. Local UI, logs,
  storage, control APIs, and explicitly installed debug tools may retain their
  product names; they are not gateway-injected upstream identity.

## Documentation changes

- Do not modify README files, including those in subdirectories, unless the user
  explicitly requests README changes.
- Do not add documentation files unless the user explicitly requests them or
  they are necessary to complete the requested task. Avoid unsolicited notes,
  summaries, reports, and implementation plans in the repository.

## GitHub issues and pull requests

Apply these rules when preparing or submitting an issue or PR, including when
using `gh`. Drafting a body does not authorize publishing it: create or edit
GitHub issues, PRs, or comments only when the user explicitly requests that
action.

**Issues:** Read `.agents/github/ISSUE.md` before drafting an issue. Check its
scope rules, then search `docs/guides/`, `CONTRIBUTING.md`, the README, relevant
code, and existing issues. Answer usage, configuration, or integration questions
from those sources instead of filing them. For an in-scope bug or feature, fill
the agent template as the entire body; do not use the human GitHub issue forms.
Quote the user's request faithfully, preserving its language and line breaks.
Keep answers short and factual. Record actual behavior, impact, frequency, and
applicable type-specific details. For features, describe the current limitation
and use case. Ask only for required facts that cannot be established from
available evidence, and wait before filing; do not invent answers or ask the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Calcium-Ion/AstrLink](https://github.com/Calcium-Ion/AstrLink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
