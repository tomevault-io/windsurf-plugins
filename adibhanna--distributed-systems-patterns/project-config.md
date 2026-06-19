---
trigger: always_on
description: Apply distributed-systems, messaging, and integration patterns to architectural decisions, message contracts, runbooks, and launch decisions for event-driven, microservice, queue, broker, saga, outbox, CDC, workflow, scaling, resilience, and multi-region work. Triggers include Kafka, RabbitMQ, SQS/SNS/EventBridge, Pub/Sub, NATS, Pulsar, Temporal, Step Functions, Debezium, CloudEvents, AsyncAPI, OpenTelemetry, plus idempotency, DLQs, retries, ordering, schema evolution, replay, sharding, backpres
---


# Distributed Systems Patterns

## Purpose

This skill produces durable architectural artifacts for distributed systems work: design docs, message contracts, ADRs, runbooks, and launch decisions. Six slash commands write these artifacts; one (`/review`) reads them plus the implementation diff and produces architectural findings.

The artifacts the skill produces (decisions, contracts, runbooks, launch decisions) are valuable when multiple teams or services must coordinate. The skill does NOT generate implementation code or tests - that's your team's job in their normal dev environment. The skill's value is in the decision and review layer, where teams typically under-invest.

The skill is technology-neutral. Specific package picks (which Kafka client, which ORM) are team decisions; the skill recommends categories. Default outputs are architectural artifacts at canonical paths under `docs/features/<slug>/` and `docs/system/`.

## Who this skill is for

Distributed-systems engineers, tech leads, staff/principal engineers, platform teams, and architects making cross-service decisions at scale. Six commands produce durable artifacts (design docs, ADRs, contracts, runbooks, launch decisions) that are valuable when multiple teams or services must coordinate; they are overhead when one engineer can hold the whole system in their head.

**Not for**: single-process apps, single-function utilities, frontend-only work, quick local refactors, ETL jobs without service coordination, beginner pattern questions ("what is a queue?"), or pre-MVP prototypes that don't yet have users or operational costs.

**Threshold for invoking the skill**: at least two services or two teams must coordinate; or the work introduces durable infrastructure (broker, workflow engine, schema registry, mesh, cache fleet, shard, new consistency model); or the request explicitly asks for an ADR / RFC / runbook / launch decision.

**Decline behavior when below the threshold**: if the user invokes the skill (via vocabulary trigger or slash command) on a problem that does not meet the threshold, the skill must explicitly decline. Tell the user one sentence about why ("This is a single-process refactor; a regular prompt is faster") and answer the question simply, without producing design docs / ADRs / contracts. Examples that should trigger decline:

- A single-team prototype, hackathon, or side project.
- A single-process app with one database and no async work.
- A frontend-only change.
- A "what is X?" beginner pattern question (answer the question; don't run the pipeline).
- An ETL job that doesn't cross service or team boundaries.

When in doubt about whether the threshold is met, ask the user one question: "How many services/teams will this touch, and is this for production with real users?" before deciding whether to run the pipeline.

## Shared knowledge across features

Some knowledge applies to every feature, not just one. The skill organizes this under `docs/system/`:

- `docs/system/catalog.md` - the service/feature registry
- `docs/system/adrs/` - **platform-wide** decisions (broker choice, mesh policy, schema-registry vendor, multi-region strategy)
- `docs/system/runbooks/` - **platform-wide** runbooks (broker outage, schema-registry rollback, region-wide failover)
- `docs/system/standards/` - conventions every feature must follow (channel naming, observability, security baseline, deployment, on-call expectations)
- `docs/system/glossary.md` - shared vocabulary (domain terms used by multiple features)
- `docs/system/topology.md` - team ownership map and Conway-Law boundaries
- `docs/system/capacity.md` - platform capacity envelope (broker throughput, total cost budget, regional limits)
- `docs/system/compliance.md` - PII / GDPR / SOC2 / data-residency baseline that all features inherit
- `docs/system/dr.md` - DR strategy and region-failover plan that applies across features

**The principle: reference, don't restate.** When a feature design touches a shared concern (e.g. "follows the platform observability standard"), link to the shared doc rather than copy-pasting the rule into every feature. If the same fact appears in three feature docs, it belongs in `docs/system/standards/` instead.

Feature artifacts cross-link into `docs/system/` using `../../system/<path>` (one `..` for the feature subdir, one for `features/`). The per-feature README's `## Shared references` section names which platform docs apply to this feature.

This skill is an operating procedure. Load only the reference file needed:

**Operating context loaded automatically when relevant** (these live in the user's repo, not the skill):
- `docs/system/standards/*.md` - platform conventions (when a feature artifact would touch a convention)
- `docs/system/adrs/*.md` - platform-wide decisions (when a feature artifact must comply)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adibhanna/distributed-systems-patterns](https://github.com/adibhanna/distributed-systems-patterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
