---
trigger: always_on
description: Handles built from Laser preserve content-addressed ids and run log
---

# LaserData - Laser SDK Agent Guidelines

This repo is one workspace holding two published crates. **laser-wire** is the LaserData wire contract (types, codes, envelopes, dictionaries, caps, the golden fixture corpus, and the Agent Data Exchange Protocol envelope), runtime-free and wasm-portable, consumed by LaserData Cloud, Iggy server, and the SDK alike. **laser-sdk** is the open, customer-facing **data-platform SDK** over [Apache Iggy](https://iggy.apache.org) built on top of it: typed publish/consume, declared projections with a query DSL, a key-value store, and copy-on-write forks over one connection, plus an optional agent layer (provenance-tagged messages, a conversation/causality spine, agent topic routing, a reliable consumer, context assembly, a memory seam, and an `Agent` builder). It carries `gen_ai.*` provenance describing LLM calls but never makes them: it only moves and coordinates messages.

> Skills under `.claude/skills/` cover the SDK by area. Load [laser-sdk-overview](.claude/skills/laser-sdk-overview/SKILL.md) first, then the focused skill for the area you are touching.
>
> The [AGDX spec](docs/agdx.md) is the authoritative wire/convention reference (streams, topics, headers, envelopes, query DSL, caps), kept byte-identical to the LaserData Cloud's wire types. Update it whenever the wire contract changes.

## Contents

- [STOP and ask the user before](#stop-and-ask-the-user-before)
- [Verification order](#verification-order)
- [Structure](#structure)
- [Repo-wide principles](#repo-wide-principles)
- [Conventions](#conventions)
- [Testing](#testing)
- [What is shipped vs planned](#what-is-shipped-vs-planned)

## STOP and ask the user before

These change the on-the-wire or public contract and break data or downstreams:

- Editing ANYTHING in `wire/src/` that alters encoded payload: command codes, op versions, header keys (`wire/src/headers.rs`), topic names, envelope field names or serde attributes, the content-type / task-state / error-code u8 dictionaries, or the caps. Every message already on the log and every consumer (LaserData Cloud, Iggy server) is pinned to those bytes. The golden corpus under `wire/fixtures/` will fail on drift. Before the first published contract, intentional breaking changes keep operation versions at 1 and must update fixtures plus every consumer and implementation as one release unit. Once a contract version ships, a breaking change increments the affected operation version.
- Changing `ConversationId::derive` (the FNV-1a algorithm in `sdk/src/types/ids.rs`) without bumping `DERIVE_VERSION` - silently remaps every `SessionPolicy::PerUser` conversation. (`AgentId::wire_id` is no longer a hash: the wire agent id is the name string verbatim.)
- Renaming `AgentTopic` names (`sdk/src/provenance/topic.rs`) - repoints live topics.
- Changing `Provenance::partition_key` (currently `conversation_id`) - breaks the per-conversation ordering guarantee.
- Changing the public signatures of `Laser`, `AgentHandler`, `Agent`, `AgentConsumer::run`, or `AgentHandle` - these are the customer-facing API.
- Dropping an attribution field (`agent`, `root_conversation_id`, ...) when rebuilding `Provenance` in `spawn_subconversation` / `AgentCtx::reply_provenance` silently breaks causality and cost rollup across composed flows.
- Query requires a managed deployment, either LaserData Cloud or Laser Stack: it rides the `AGDX_QUERY` managed command off the log (`send_raw_with_response`, no reply topic, no correlation poll), and against Apache Iggy without a managed backend returns `LaserError::Unsupported`. There is no topic request/reply query path. The connect-time `AGDX_HELLO` probe sets the connect-local `managed_host` **and** lights up `query.available` (with the other managed surfaces), so a plain `Laser::connect(..)` against a managed deployment queries with no manual caps.

## Verification order

Enforced by CI (`.github/workflows/ci-rust.yml`: jobs `lint`, `lint-detached`, `build`, `wire-feature-matrix`, `feature-matrix`, `wasm`, `deny`, `test`, `fuzz`, `bdd`). `ci-python.yml` additionally gates publication on artifact-only jobs (`wheel-install` on every platform at Python 3.10 and 3.13, `sdist-install` from a checkout-free directory). `ci-typescript.yml` builds the npm tarball and installs it into a clean consumer before publication. Managed end-to-end validation belongs to Laser Stack's `scripts/smoke`, which accepts a local SDK tarball. The `bdd/rust` and `fuzz` crates sit OUTSIDE the workspace, so `--workspace` does not reach them. `lint-detached` (and `just lint`) run fmt/sort/machete/clippy inside each.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laserdata/laser-sdk](https://github.com/laserdata/laser-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
