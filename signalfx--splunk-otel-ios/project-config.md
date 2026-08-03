---
trigger: always_on
description: This repo is a modular Swift Package for the Splunk RUM iOS agent. It instruments iOS, iPadOS, tvOS, visionOS, and macCatalyst apps and sends telemetry to Splunk Observability Cloud.
---

# Splunk RUM Agent for iOS - AI Assistant Context

This repo is a modular Swift Package for the Splunk RUM iOS agent. It instruments iOS, iPadOS, tvOS, visionOS, and macCatalyst apps and sends telemetry to Splunk Observability Cloud.

## How to Use This Guide

- Treat this file as the primary agent-facing guidance for this repository. If it conflicts with `CODESTYLE.md`, `Development.md`, `CONTRIBUTING.md`, or a module-local pattern, follow the more specific guidance for the touched code and call out the conflict.
- Keep changes scoped to the user request.
- Get explicit confirmation before changing public API, dependencies, distribution metadata, privacy-sensitive telemetry, or CI workflows.
- Update this file when build, test, packaging, dependency, or review conventions change.

## Project Facts

- Swift 5.9+, SPM, minimum iOS 13.0.
- Public products: `SplunkAgent`, `SplunkAgentObjC`.
- Core dependency: `opentelemetry-swift-core` API/SDK only. Do not add upstream protocol exporters; this repo uses a custom OTLP/JSON exporter to control binary size.
- Main modules: `SplunkAgent`, `SplunkAgentObjC`, `SplunkCommon`, `SplunkOpenTelemetry`, `SplunkOpenTelemetryBackgroundExporter`, and instrumentation modules named `Splunk<Feature>`.
- Instrumentation modules conform to `Module` in `SplunkCommon/Sources/SplunkCommon/Modules/Module.swift`; infrastructure targets such as `SplunkOpenTelemetry` and `SplunkOpenTelemetryBackgroundExporter` do not.
- Public API lives in `SplunkAgent/Sources/SplunkAgent/Public API/`; stable public files use `API-1.0-*.swift`.
- Deprecated public API stays in `Public API/.../Deprecated/` with `@available(*, deprecated, message: "Use <replacement>")`.
- Public module APIs have both real proxies (`Proxies/Module/`) and no-op `*NonOperational` proxies (`Proxies/Non-Operational/` or `Proxies/NonOperational/`) for pre-install, disabled, or sampled-out states.
- The binary distribution uses `tools/xcframework/Project.swift` with library evolution enabled. Keep it in sync with `Package.swift`.

## Build, Test, and Validation Commands

- Do not use `swift test` as the default validation command for this repository. The test suite must run through the Xcode scheme because Apple-platform destinations, resources, and package wiring matter.
- Build validation:
  `xcodebuild -scheme SplunkAgent -destination "generic/platform=iOS Simulator" build`
- Test validation:
  `xcodebuild -scheme SplunkAgent -destination "OS=<installed OS>,name=<installed iPhone simulator>" test`
- The simulator OS and device name change over time. Before running tests, discover currently available scheme destinations with `xcodebuild -scheme SplunkAgent -showdestinations`, then substitute an installed iPhone simulator. Use `xcrun simctl list devices available` only as a fallback when the Xcode destination output is not enough. Example:
  `xcodebuild -scheme SplunkAgent -destination "OS=26.5,name=iPhone 17" test`
- For targeted test runs, keep the same scheme and destination and add `-only-testing:<TestBundle>/<TestClass>` or `-only-testing:<TestBundle>/<TestClass>/<testMethod>`.
- Report the exact command, destination, and result for any validation performed. If validation cannot run because no compatible simulator is installed, report the discovered destinations and the blocker.

## Implementation Defaults

- Inspect nearby code before editing and match the established module, naming, `// MARK: -`, and test-support patterns.
- For new modules, follow the existing target layout in `Package.swift` (for example `Splunk<Module>/Sources` and `Splunk<Module>/Tests`) with `Testing Support/Builders` and `Testing Support/Mocks` when that support structure is needed.
- Public API and protocols need DocC; new source files need the license header from `CODESTYLE.md`.
- SDK instrumentation must be defensive: never let telemetry collection crash the host app. Prefer no-op, drop, or internal logging over `fatalError`, `try!`, forced unwraps, or uncaught errors in production paths.
- For style-heavy, build/distribution, or contribution-process changes, read `CODESTYLE.md`, `Development.md`, or `CONTRIBUTING.md` before editing.

## Security, Privacy, and Dependency Guardrails

- Do not add new third-party runtime dependencies, new upstream OpenTelemetry exporters, or dependency-version overrides unless the user explicitly requests a dependency change and the binary-size, license, and xcframework impacts are reviewed.
- Do not hard-code or commit Splunk realms, access tokens, credentials, customer data, private repository paths, or local machine paths in source, tests, fixtures, docs, manifests, or generated files.
- Do not emit raw request or response bodies, cookies, authorization headers, credentials, or obvious PII through spans, logs, crash payloads, Session Replay metadata, or internal agent events. Reuse existing masking/redaction utilities where available.
- Treat privacy-impacting resource changes, Session Replay capture changes, crash-report changes, and telemetry attribute changes as customer-visible behavior changes.

## Telemetry Model


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [signalfx/splunk-otel-ios](https://github.com/signalfx/splunk-otel-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
