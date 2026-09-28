---
trigger: always_on
description: Exercise independent judgment, remain evidence-driven, and avoid rigid or dogmatic application of this document. These guidelines are intended to improve engineering quality, not to replace technical reasoning. If an agent determines that a rule, workflow, reference requirement, or architectural recommendation in this document is inappropriate, excessive, outdated, or counterproductive for the specific development task, it may adapt, reinterpret, or revise it as necessary. Such adjustments shoul
---

# AGENTS.md

## Pragmatic Interpretation

Exercise independent judgment, remain evidence-driven, and avoid rigid or dogmatic application of this document. These guidelines are intended to improve engineering quality, not to replace technical reasoning. If an agent determines that a rule, workflow, reference requirement, or architectural recommendation in this document is inappropriate, excessive, outdated, or counterproductive for the specific development task, it may adapt, reinterpret, or revise it as necessary. Such adjustments should preserve the document's underlying positive intent: correctness, technical rigor, maintainability, modularity, security, and sound engineering judgment. Prefer practical truth over formal compliance, and do not follow a rule mechanically when doing so would clearly produce a worse technical result.


## Purpose

DuckDetector is an Android environment integrity diagnostics project.

All implementation work must prioritize:

* correctness,
* modularity,
* low coupling,
* traceable technical reasoning,
* version-aware behavior,
* reproducibility,
* and conclusions proportional to the evidence collected.

Security-sensitive behavior must not be implemented from assumptions, memory, copied snippets, community folklore, or undocumented expectations.

Research must precede design, and design must precede implementation.

---

# 1. Architecture Principles

DuckDetector follows strict principles of:

* modularity,
* separation of concerns,
* explicit ownership,
* low coupling,
* narrow interfaces,
* replaceable implementations,
* isolated platform-specific code,
* and minimal shared mutable state.

Each feature should own its own implementation wherever practical.

A feature should not depend on internal implementation details of another feature unless that dependency is explicitly part of the architecture.

Prefer:

* feature-oriented modules,
* small and explicit interfaces,
* stable domain models,
* dependency inversion where appropriate,
* dedicated platform adapters,
* dedicated JNI bridges,
* isolated native probes,
* explicit unsupported states,
* testable components.

Avoid:

* global state,
* hidden coupling,
* cross-feature implementation leakage,
* duplicated platform logic,
* unrelated shared utility classes,
* broad abstractions with unclear ownership,
* implicit dependencies,
* monolithic detector implementations.

Shared infrastructure must exist because multiple features genuinely share the same abstraction, not merely because moving code into a common package appears convenient.

---

# 2. Research Before Design

Before implementing, modifying, or drafting a non-trivial detector or platform-dependent feature, identify the subsystem responsible for the observed behavior.

Do not begin from:

> "How can DuckDetector detect this?"

Begin from:

> "What subsystem produces this observable behavior, and what authoritative source establishes its semantics?"

The expected workflow is:

```text
Observed behavior
    ↓
Responsible subsystem
    ↓
Authoritative documentation
    ↓
Actual implementation
    ↓
Version and device applicability
    ↓
Detection model
    ↓
Architecture
    ↓
Implementation
```

A detector must not be designed around an observation whose origin is not sufficiently understood.

---

# 3. Mandatory Technical References

The exact references depend on the subsystem being modified.

There is no single reference hierarchy that applies equally to every detector.

The following sources are mandatory when applicable.

## 3.1 Android Platform Documentation

Consult official Android documentation first for public platform behavior.

Primary sources include:

* Android Developers documentation
* Android platform documentation on `source.android.com`
* Android security documentation
* Android compatibility documentation where relevant
* Android API references
* official Android architecture documentation

Documentation is not always sufficient by itself.

When implementation details affect detector semantics, verify them against the corresponding source code.

---

# 4. AOSP Source Code

For Android userspace and framework behavior, inspect the actual AOSP subsystem responsible for the behavior.

Relevant source trees may include:

```text
frameworks/base
system/core
system/security
system/sepolicy
bionic
art
packages/modules/*
hardware/interfaces
system/libhidl
external/*
```

This list is illustrative, not exhaustive.

Do not cite or reason from "AOSP" generically.

Locate the exact subsystem, class, native component, service, HAL interface, daemon, policy, or runtime code that defines, exposes, or enforces the behavior.

For example:

* process behavior may require inspecting Zygote and ActivityManager implementation,
* property behavior may require Bionic and init/property service code,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eltavine/Duck-Detector-Refactoring](https://github.com/eltavine/Duck-Detector-Refactoring) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
