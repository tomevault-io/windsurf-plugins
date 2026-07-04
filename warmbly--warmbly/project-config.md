---
trigger: always_on
description: Warmbly is an email warmup and cold outreach platform.
---

# Warmbly Agent Notes

## Purpose

Warmbly is an email warmup and cold outreach platform.

At a product level, the app does four main things:

- manages sender accounts and their assignment to workers
- sends campaign and warmup mail through distributed workers
- syncs mailbox state back into the platform
- tracks opens, clicks, replies, suppression, and deliverability signals

The backend API is the control plane. Workers are the execution plane.

## Working In This Repo

CI is strict. `go build ./...` succeeding is not enough — `golangci-lint` runs `gofmt` as part of its checks, and a single unformatted import block or mis-indented doc comment will fail the PR even when the code compiles cleanly. Before declaring any Go change done:

- run `gofmt -w` on every Go file you touched (or `gofmt -w internal/ cmd/` to be safe)
- run `make lint` locally when the toolchain is installed, or at minimum `gofmt -l ./...` should print nothing
- do not rely on `go build` as the "ship signal" — it ignores formatting and stylistic lint rules that CI enforces

Other CI-touching rules:

- the frontend trees (`admin/`, `web/`, `site/`) each have their own CI jobs; run `pnpm typecheck` in any tree you touched and `pnpm lint` when the rules are non-trivial
- never push without first re-running the relevant `*build*` / `*typecheck*` / `*lint*` step on the affected tree
- a `make lint` (or `gofmt -l`) failure is always a real CI failure; do not push hoping it will pass

Docs stay in sync:

- the customer docs site lives in `docs/` (Fumadocs, served at docs.warmbly.com); content is MDX under `docs/content/docs/` in three sections: `guides/` (product behavior), `learn/` (fundamentals), `api/` (API reference)
- any change that alters user-visible behavior must update the matching docs page in the same change: a new or changed endpoint updates `api/endpoints.mdx` (scope map) and, where relevant, `api/authentication.mdx`; a new or changed API permission updates `api/permissions.mdx` including the permission table, presets, and all three language tabs in the constants section; a new or changed error code updates `api/error-codes.mdx`; a new or changed product feature, default, limit, or setting updates the relevant `guides/` page (or adds one, registered in `guides/meta.json` with an `icon`)
- removing or renaming a feature, endpoint, or permission means removing or updating its docs too; do not leave stale docs behind
- follow the docs conventions: frontmatter `title` is the H1 (no `#` heading in the body), every page has a lucide `icon`, sentence-case headings, no em dashes in prose, internal links use trailing slashes (`/guides/mailboxes/`)
- verify with `pnpm types:check` and `pnpm lint` in `docs/` (the site is a fully static export; `pnpm build` writes `out/`)

Commit hygiene:

- when instructed to make a commit, use the subject format `feat: <explanation>`
- one line, no body. Make the line long and specific (what changed and where), not a stub like `feat: fix docs`
- no `Co-Authored-By:` or other AI/agent attribution footers; rewrite any commit that has one before opening or updating a PR

Copy / writing style:

- do not lean on em dashes (`—`). Use them sparingly, only when one is genuinely the clearest option; prefer a period, comma, colon, or parentheses instead. This applies to user-facing copy and microcopy in `site/` and `web/`, and to docs. Overusing em dashes reads as machine-written.

Code comments:

- keep them short: one line stating the non-obvious constraint or intent. No multi-line essays; if a comment needs a paragraph, the explanation belongs in docs or the PR description

Data modeling / representation:

- we are happiest with the most **type-safe** option, but the rule is: pick the **most effective option for the actual use case**, not type-safety for its own sake.
- prefer real typed columns / enums when the data is fixed-shape, queried or filtered in SQL, or benefits from FK integrity.
- a `jsonb` column is the right call when the data is a free-form, evolving, read-then-execute blob that isn't filtered in SQL (e.g. the `sequences.conditions` branching tree and `sequences.action` node config) — keep it type-safe at the app boundary with a Go struct + validation on write, and a DB `CHECK` on any discriminator column.

### Verification: what to run, what to skip

Keep the loop fast. The signals that matter are formatting, lint, and typecheck — not local builds or browser automation.

Always, before calling a Go change done:

- run `make fmt` (or `gofmt -w cmd internal`); `gofmt -l ./...` must print nothing
- run `make lint` (golangci-lint)

For frontend changes, run `pnpm typecheck` and `pnpm lint` in any tree you touched.

Do not:

- do not run `go build ./...`, `pnpm build`, or docker image builds as a "did it work" check. They are slow and are not what CI gates on. `go run` (via the make dev targets) already compiles; `make fmt` + `make lint` + `pnpm typecheck` are the real signals.
- do not write or run Python/Playwright (or any browser-automation) scripts to test the app. Manual, in-browser verification is the user's job against the native dev stack (`make infra` + `make backend` + `make web`). Do not add screenshot/e2e test harnesses to this repo.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [warmbly/warmbly](https://github.com/warmbly/warmbly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
