---
trigger: always_on
description: Guidance for coding agents (and humans) contributing to this repo. Every command and
---

# AGENTS.md — working on rindexer

Guidance for coding agents (and humans) contributing to this repo. Every command and
path in this file is taken from the repo's actual CI workflows, Makefiles, and source —
follow it exactly rather than guessing.

rindexer is an open-source, high-speed EVM indexing toolset written in Rust, compatible
with any EVM chain. Users index chain events either with a single `rindexer.yaml` file
("no-code" mode) or by generating a typed Rust project ("rust" mode), and get storage
(PostgreSQL / ClickHouse / CSV), streams (Kafka, webhooks, SNS, Redis, RabbitMQ,
Cloudflare Queues), chat alerts, and a GraphQL API out of the box. Docs: https://rindexer.xyz

## Repo map

| Path | What it is |
|---|---|
| `core/` | The `rindexer` crate — all indexing logic, manifest parsing, databases, streams, reorg handling, codegen templates |
| `cli/` | The `rindexer_cli` crate — the `rindexer` binary (clap CLI wrapping core) |
| `graphql/` | Node.js PostGraphile server, compiled to a binary by `core/build.rs` and embedded into the Rust binary |
| `documentation/` | Docs site (vocs). **Contains the changelog** — see below |
| `e2e-tests/` | Standalone E2E harness (workspace member, custom runner binary, not `#[test]`-based) |
| `examples/` | 22 example projects: no-code YAML, Rust projects (4 are workspace members), and `tables_*` examples |
| `xtask/` | Dev tooling (`cargo xtask …`) for maintaining `.blockclock` timestamp files — not part of releases |
| `helm/` | Kubernetes chart (`helm/rindexer/`). Not versioned/published by CI |
| `providers/` | Deployment templates (Railway; `providers/aws/` is an empty placeholder) |

There is no root Makefile. Makefiles exist only at `cli/Makefile` and `e2e-tests/Makefile`.

## ⚠️ The changelog rule — always update it

**Every change you make must be reflected in the changelog, in the same PR/commit as the
code.** The changelog is NOT a root `CHANGELOG.md` — it lives at:

```
documentation/docs/pages/docs/changelog.mdx
```

### How to add an entry

Add a bullet to the **unreleased section at the very top of the file** (above the
`## Releases` heading), under the heading matching your change type. The top of the file
looks like this — add bullets directly below the divider of the right section:

```markdown
# Changelog

### Breaking changes
-------------------------------------------------
- breaking: <entry>

### Bug fixes
-------------------------------------------------
- fix: <entry>

### Features
-------------------------------------------------
- feat: <entry>

## Releases
-------------------------------------------------
```

Entry format rules (copied from real entries):

- Prefix with `- fix:`, `- feat:`, or `- breaking:` to match the section.
- Small change → one lowercase sentence: `- fix: disable reth on windows`.
- Significant change → bold title, em-dash, then user-facing detail in the same bullet:
  `- feat: **Parallel historical backfill** — new `fetch_concurrency` network config splits historic block ranges across N concurrent workers…`
- Write for **users of rindexer**, not for reviewers of your diff: name the YAML
  field / env var / metric / CLI flag in backticks, state defaults, and call out
  migration impact (e.g. "run `rindexer codegen typings` to regenerate",
  "downstream consumers need to allow the new field").
- Link issues inline where relevant: `([#383](https://github.com/joshstevens19/rindexer/issues/383))`.

### What NOT to do in that file

- **Never edit anything below the `## Releases` heading** and never restructure the
  headings. `.github/workflows/edit-releases.yml` and `migrate-releases.yml`
  machine-parse this file (they split on `## Releases` and match `X.Y.Z-beta` version
  headings) to generate GitHub release notes — format deviations break them.
- Don't add new dashed dividers; the `### <Section>` + 49-dash divider lines already
  exist at the top. Just add bullets beneath them.
- Don't create a dated `# X.Y.Z-beta - <date>` heading — that happens at release time,
  not in feature/fix PRs.

The only changes exempt from a changelog entry are ones with zero user-visible effect
(CI config, comment/typo fixes, internal docs like this file). When in doubt, add an
entry — features are expected to ship with one 100% of the time.

## Breaking changes — what counts and how to handle them

A change is breaking if an existing user's project stops working **or silently changes
behavior** after upgrading — not just Rust API breakage. Every breaking change needs a
`- breaking:` changelog entry with explicit migration instructions (see the 0.41.0
`Arc<dyn ChainProvider>` entry for the expected shape: what breaks, what to change,
what command to re-run).

### rindexer.yaml is the most sensitive surface

Users have long-lived `rindexer.yaml` files that must keep working across upgrades.
Rules, grounded in how the manifest actually behaves:

- **New fields must be optional AND round-trip-safe.** The established pattern is
  `#[serde(default, skip_serializing_if = "Option::is_none")]`. The `skip_serializing_if`
  half is not cosmetic: `rindexer add contract`, `rindexer new`, and `rindexer phantom`
  deserialize the user's whole manifest and write it back (`write_manifest` in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joshstevens19/rindexer](https://github.com/joshstevens19/rindexer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
