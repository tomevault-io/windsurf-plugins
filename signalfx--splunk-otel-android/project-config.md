---
trigger: always_on
description: This repo is the Splunk Android RUM SDK: a lightweight, multi-module Gradle/Kotlin library built on OpenTelemetry and embedded in customer Android apps. Keep this file concise; move long module notes to focused docs only when this guide can no longer stay scannable.
---

# AI Agent Guidelines for Splunk Android RUM SDK

This repo is the Splunk Android RUM SDK: a lightweight, multi-module Gradle/Kotlin library built on OpenTelemetry and embedded in customer Android apps. Keep this file concise; move long module notes to focused docs only when this guide can no longer stay scannable.

## Non-Negotiables

- **No new dependencies** unless the user explicitly asks. Even dependencies listed in `buildSrc/src/main/kotlin/Dependencies.kt` need explicit approval because SDK size and consumer conflicts matter.
- **Public API is stable.** Treat `agent/`, any module exposed through `api(...)` dependencies, and customer-facing Gradle plugin surfaces under `instrumentation/buildtime/` as compatibility-sensitive.
- **Do not change public API or default behavior without confirmation.** This includes signatures, defaults, visibility, constants, data classes, errors, side effects, deprecations, removals, and telemetry semantics.
- **Maintain backward compatibility.** SDK consumers should be able to upgrade without source changes unless a breaking change is intentional, documented, and approved.
- **SDK runtime paths must not crash host apps for predictable conditions.** Prefer graceful fallback plus internal diagnostics over `throw`, `error`, or `require` in production runtime paths.
- **Production mobile performance is a primary correctness concern.** Optimize for low startup, main-thread, memory, CPU, network, storage, and battery impact.

## Repo Map

| Path | Purpose |
| --- | --- |
| `agent/` | Main SDK entry point and stable public API |
| `integration/` | Feature instrumentation; public when exposed through `:agent` |
| `integration/{anr,applicationlifecycle,crash,customtracking,httpurlconnection-auto,interactions,lifecycle,navigation,networkmonitor,okhttp3-*,sessionreplay,slowrendering,startup,webview}/` | Instrumentation modules |
| `integration/agent/api/` | Public API interfaces |
| `integration/agent/common/` | Transitive public API via `integration/agent/api`; compatibility-sensitive shared agent types |
| `integration/agent/internal/` | Internal agent implementation |
| `common/otel/` | Transitive public API via `integration/agent/api`; compatibility-sensitive OpenTelemetry helpers |
| `common/storage/`, `common/utils/` | Internal shared helpers unless exposed through an `api(...)` dependency |
| `instrumentation/buildtime/` | Customer-facing Gradle plugins and build-time instrumentation; plugin IDs, extensions, tasks, and defaults are public API |
| `instrumentation/runtime/` | Runtime hooks used by instrumentation |
| `buildSrc/src/main/kotlin/Dependencies.kt` | Allowed dependency catalog, not approval to use dependencies |
| `app/` | Sample application |

## Common Commands

Use the narrowest module command that validates the change.

| Task | Command |
| --- | --- |
| Format module | `./gradlew :<module>:ktlintFormat` |
| Check module | `./gradlew :<module>:check` |
| Build module | `./gradlew :<module>:build` |
| Device/emulator tests | `./gradlew :<module>:connectedCheck` |
| Whole repo build | `./gradlew build` |
| Whole repo format | `./gradlew ktlintFormat` |

## Change Rules

- Follow existing Kotlin, Gradle, threading, storage, and module patterns before adding abstractions.
- Keep changes inside the smallest reasonable module boundary.
- Prefer `internal` for SDK implementation details.
- Document public APIs with KDoc.
- Deprecated APIs must continue to work and include migration guidance.
- New features should be opt-in unless the user explicitly approves a default behavior change.
- Do not add coroutine, threading, reactive, serialization, or utility dependencies to solve local implementation problems.
- GitHub Actions must be pinned to commit SHAs, not mutable version tags.
- When runtime state is invalid or optional components are absent, no-op or partially degrade when safe.
- Exceptions are acceptable only for invalid developer usage at API boundaries with no safe fallback, tests/test utilities, or build-time tooling.

## PR Review Priorities

Review findings in this order. Lead with customer impact, not style.

### P1 - Host App Safety and Production Performance

- Block or request measurement for avoidable overhead in customer apps at scale.
- Watch hot paths: SDK init, lifecycle callbacks, UI instrumentation, ANR/crash handling, network interceptors, span/log/event creation, processors, exporters, and background work.
- Flag any `throw`, `error`, `require`, forced unwrap, unchecked cast, or non-null assertion in SDK runtime paths when the state can predictably happen in production.
- Flag repeated allocations, reflection, regex parsing, per-signal encoder/formatter creation, disk/network I/O, locks, O(n) scans over growing data, retained `Context`/`Activity`, and large retained objects.
- Flag unbounded queues, retries, timers, listeners, observers, or background work that grows with sessions, spans, events, logs, screens, requests, or lifecycle churn.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [signalfx/splunk-otel-android](https://github.com/signalfx/splunk-otel-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
