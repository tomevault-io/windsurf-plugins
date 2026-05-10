---
trigger: always_on
description: > The contextual privacy gate for OpenClaw — your agent decides what to share, like a Wallfacer.
---

# Clawfacer

> The contextual privacy gate for OpenClaw — your agent decides what to share, like a Wallfacer.

This document is the persistent context for Claude Code sessions on this project. Read it at the start of every session before doing any work. If you're tempted to make architectural changes that contradict anything here, stop and surface the conflict to the human first.

## What this is

Clawfacer is an OpenClaw plugin that gates outbound messages from an OpenClaw agent based on contextual integrity. Before the agent sends a message to any counterparty (in WhatsApp, Slack, Telegram, ACP, etc.), Clawfacer extracts the information flows in the message, classifies each flow against the user's policy, and either rewrites the message to remove disallowed flows or surfaces an approval request.

The mechanism is Microsoft Research's PrivacyChecker (arXiv 2509.17488, EMNLP 2025), adapted as an OpenClaw-native plugin.

## Why this exists

OpenClaw's published security model explicitly states it is not a multi-tenant trust boundary. Cross-user agent-to-agent communication has no contextual integrity layer. Existing OpenClaw security plugins (SecureClaw, knostic/openclaw-shield) are regex-based DLP — they catch credit card numbers, not "is sharing my home address with this counterparty contextually appropriate." That's the gap Clawfacer fills.

The RFC #49971 discussion on the OpenClaw repo closed with the maintainer pointing at the existing plugin hook surfaces and saying "trust/privacy plugins go here." Clawfacer is one such plugin, focused specifically on disclosure policy (separate from identity verification, which is a different layer).

## Scope of v0.1, and what v0.1 deliberately keeps open

v0.1 ships as an OpenClaw plugin. That is the entire deliverable. We are not building a multi-platform tool, a starter policy library, or a deterministic fast-path tier in v0.1.

However, v0.1 should not foreclose v0.2 options. Specifically:

- The core (`src/core/`) must be designed as if other people will import it directly into non-OpenClaw projects. Public API surface, exports, and types should be sensible to consume from a generic TypeScript codebase. v0.2 may publish the core as a standalone library (`clawfacer/core`) without requiring a refactor.
- A `policies/` directory exists at the repo root with a placeholder README explaining that starter policies will live there. v0.1 does not include any starter policies — but the directory's existence signals that policies are first-class to the project, not an afterthought.
- The architecture leaves room for a deterministic fast-path tier (regex DLP runs before the LLM judge, blocking obvious cases without an LLM call) without committing to building it. v0.2 can layer this on if v0.1 validation reveals the LLM-judge cost or latency is a real problem.

The discipline: keep doors open at zero or near-zero cost, validate the primitive in production for Amiko, decide later whether v0.2 broadens (protocol contribution path) or deepens (Amiko-internal path). The session-1 spec is correct either way; the choice doesn't need to be made now.

Build, in order:

1. Core checker library: takes (message, context) → returns (revised_message, audit_log). Contains the PrivacyChecker prompt template, an LLM client abstraction, and a flow-judgment pipeline. Importable as a library independent of OpenClaw.
2. OpenClaw plugin shell that registers the right hooks and calls the core.
3. Policy file format and loader (markdown-based, lives in OpenClaw workspace).
4. Audit logging and the "your twin almost said X to Y, was that okay?" feedback loop in chat.
5. Adversarial hardening: regex DLP layer beneath the LLM judge, prompt-injection isolation, fail-closed defaults.
6. Eval harness against PrivacyLens scenarios.
7. Docs, install path, ClawHub-ready packaging.

Not in v0.1, deferred to v0.2 or later:

- Hermes plugin support.
- Starter policy library (`policies/` directory exists but is empty in v0.1).
- Deterministic fast-path tier for compliance-style use cases.
- TrustChain / MolTrust DID integration (recipient identity is platform-handle-only for now).
- Cross-platform policy portability spec.
- Multi-user / team policies.
- Cloud-hosted version.
- Any UI beyond chat-based interactions.

## Validation discipline

Before integrating Clawfacer into Amiko's runtime, write `VALIDATION.md` at the repo root specifying what "the primitive works for Amiko" looks like in concrete observable terms. The v0.2 decision (broaden vs deepen vs pivot) gets made against that document, not against general impressions after the fact.

This is critical because the alternative — running v0.1 in production for three months and then making the v0.2 call on vibes — is the failure mode that defeats the whole "validate before deciding" sequence. Write down what you're trying to learn before you start learning, or you'll convince yourself you learned it regardless of what actually happened.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mattyryze/clawfacer](https://github.com/mattyryze/clawfacer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
