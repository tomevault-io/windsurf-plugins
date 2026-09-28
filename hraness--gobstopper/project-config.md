---
trigger: always_on
description: - `crates/gobstopper-core/` holds the normalized transcript model, the `Edit` IR, the `Strategy` trait, all built-in strategies, and the `compaction-events-v1` telemetry schema. No I/O beyond event-log append.
---

# Contents

- `crates/gobstopper-core/` holds the normalized transcript model, the `Edit` IR, the `Strategy` trait, all built-in strategies, and the `compaction-events-v1` telemetry schema. No I/O beyond event-log append.
- `crates/gobstopper-adapters/` holds session discovery, Codex and Claude Code JSONL parsing and pure byte transforms, separate-copy publication, structural verification, and the `vault` content-addressed snapshot store. Direct provider-file replacement APIs refuse mutation.
- `crates/gobstopper-adapters/src/request/` holds the request-time compaction engine (a port of CliffCompaction's rule; MIT notice in `THIRD_PARTY_NOTICES.md`): Anthropic Messages, OpenAI Responses, and OpenAI Chat Completions dialects, the prefix store, and offline replay. It is pure JSON in, JSON out.
- `crates/gobstopper-cli/` holds the `gobstopper` binary, layered config/preset resolution, the read-only `mcp` stdio server (`mcp.rs`) that exposes vault/recall/plan/verify as agent tools (it must never surface a mutating operation), and the loopback `proxy` (`proxy.rs`).
- `docs/design.md` is the architecture and research record; `docs/roadmap.md` is the phased plan; `docs/integration-contract.md` is the historical integration contract for the runtime that preceded xcb (retired 2026-09-19).
- `site/content/blog/` holds the blog post bodies in Markdown; `site/app/blog/articles.ts` holds each post's title, sources, and review record, and `bun run sync:blog` renders the bodies into `site/app/blog/posts.generated.ts`.
- `STYLE.md` and `WRITING.md` are synced from hraness/.github. Their “Repository additions” list the Gobstopper facts that public copy most often gets wrong.

# Guidelines

- Strategies are pure: transcript in, plan out. Execution lives in adapters.
- Payload transforms preserve original record order and linkage (`parentUuid`, `ordinal`); publication creates a separate copy and preserves the source.
- Never emit transcript content (prompts, tool output, paths beyond what the provider record carries) into stdout, logs, or digests unless the user asked for that field.
- `detect`/`policy-check`/`plan --json`/`verify --json`/`vault --json` are the stable machine surfaces; keep their fields additive-only.
- Every mutating path (`apply`, `watch`) snapshots into the vault before writing and emits a `compaction-events-v1` record after; telemetry failures are non-fatal, snapshot failures abort the edit.
- The `auto` strategy prefers provider delegation for live sessions. Released native dispatch is disabled until provider ownership and correlation are verified. Separate copies require source binding and structural verification.
- Keep dependency count small; prefer `std` + `serde_json` over new crates.
- The proxy binds loopback only, passes request headers to `curl` through its environment (never argv), logs no request or response content, and forwards the client's original bytes on any parse, engine, or non-length provider failure.

# Local development and install

For fast iteration, build and install the release binary once instead of running `cargo run` each time:

```bash
cargo build --release
cargo install --path crates/gobstopper-cli --locked
```

`~/.cargo/bin/gobstopper` is then on `$PATH` after a shell restart and `gobstopper --version` reflects the current checkout.

<!-- hraness-public-copy:start -->
- Public copy (websites, READMEs, docs, package and GitHub descriptions, CLI help, `llms.txt`, generated pages) follows `STYLE.md`, synced from hraness/.github. Text a model writes for publication also follows `GENERATION_STYLE.md`.
- The delivery vocabulary in this file (admission, qualification, custody, receipt, bounded, lane, gate, surface, projection) is internal. Translate it into what the reader gets.
- Take one-line product and sibling descriptions from the portfolio registry and versions from the release record. Tests pin facts, not prose.
- Run `bun run check:copy` before handoff when the repository has it.
<!-- hraness-public-copy:end -->

<!-- hraness-releases:start -->
- GitHub Release pages follow `RELEASES.md` in hraness/.github: the title is the registry product name and the tag, and the body is a summary, `## Changes`, `## Install`, `## Verify`, then the repository's identity record as a trailing HTML comment.
- The summary and changes come from the version's section of `CHANGELOG.md` in the tagged commit. Write that section in the version bump pull request. The release workflow copies it, generates Install and Verify from the release record, fails when the section is missing or empty, and never uses GitHub's generated notes.
<!-- hraness-releases:end -->

<!-- hraness-articles:start -->
- Essays and blog posts follow the essay addendum in `GENERATION_STYLE.md` and `ARTICLE_COPY.md` in `@hraness/design-kit`. The byline is “Hraness”, every post shows the provenance note naming its recorded reviewer, and no AI-drafted post is credited to a person unless that person rewrites and adopts it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hraness/gobstopper](https://github.com/hraness/gobstopper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
