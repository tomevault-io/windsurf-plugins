---
trigger: always_on
description: Guide for AI agents working on this repository.
---

# AGENTS.md

Guide for AI agents working on this repository.

## 1. Core repository philosophy

The repository is not fundamentally:

> "an Alight Motion patch project that happens to allow other apps."

It is:

> **A collection of Morphe patches for Android applications, currently containing Alight Motion work and intended to grow to other applications.**

Existing Alight Motion patches are simply the first/older set of patches in the collection.

Future patches may target:

* Alight Motion
* entirely different Android applications
* multiple versions of the same application
* multiple independent behaviors within the same application
* bytecode, resources, raw APK files, or native binaries where appropriate

The repository should be able to grow naturally into a multi-app patch collection without requiring a separate miniature project for every application.

## 2. Do not over-isolate patches

The previous guidance placed too much emphasis on:

> "never modify existing patches"

and:

> "existing app-specific code is not shared infrastructure."

Those rules are too absolute.

Replace them with a more nuanced principle:

> **Do not modify unrelated existing code merely to make a new patch work. Reuse existing infrastructure or utilities when they are genuinely applicable, and modify shared infrastructure when the new functionality legitimately requires it.**

An existing patch being related to an application does not automatically mean it should be changed.

However, existing code is also not sacred.

A new patch may legitimately:

* reuse a helper
* reuse a fingerprinting technique
* use an existing extension mechanism
* share constants
* add a dependency
* extend shared build infrastructure
* introduce a new generic utility
* improve common infrastructure when the change benefits the collection as a whole

The key question is:

> **Is this change actually part of the new patch's implementation or shared infrastructure, or is the agent modifying unrelated code simply because it is nearby?**

Avoid the latter.

## 3. Use hoo-dles/morphe-patches as the architectural reference

Study `hoo-dles/morphe-patches` when determining how a mature Morphe patch collection organizes multiple applications and patches.

Use it to understand:

* how patches for different applications coexist
* how multiple patches for the same application coexist
* how shared infrastructure is separated from app-specific logic
* how extensions are handled
* how common utilities are reused
* how the repository grows without becoming a collection of completely independent projects
* how naming, packages, dependencies, and compatibility are handled in a real-world Morphe repository

Do **not** assume their exact directory layout, Gradle configuration, package names, or build tooling should be copied here.

The existing `anxy-patches` build system is authoritative.

The reference repository is for **architecture and organization philosophy**, not for blindly copying files.

## 4. Three categories of code

Agents should distinguish between three broad categories.

### A. Morphe framework code

This is provided by Morphe dependencies/APIs.

Examples include the patch APIs, fingerprinting mechanisms, patch annotations/builders, compatibility declarations, and related framework functionality.

Do not modify the external framework inside this repository.

Use it correctly.

### B. Repository infrastructure

This is code belonging to `anxy-patches` itself rather than to a specific Android application.

Examples include:

* build logic
* patch discovery
* packaging
* D8 compilation
* `.mpp` generation
* patch list generation
* genuinely generic helpers/utilities
* common extension/build mechanisms

This code may be modified when necessary.

Do not treat repository infrastructure as immutable.

At the same time, do not alter it casually. A change should have a concrete reason and should be validated against existing patches.

### C. App-specific implementation

This is code whose purpose is tied to a particular Android application.

Examples include:

* fingerprints targeting classes/methods from an app
* patches modifying that app's behavior
* extensions containing classes expected by that app
* app-specific constants
* app/version-specific compatibility declarations
* native patches targeting that app's `.so` files

App-specific code should be organized clearly enough that another agent can tell which application it belongs to.

Do not accidentally treat app-specific implementation as generic infrastructure.

## 5. The desired mental model for adding a patch

When asked to add a new Morphe patch, do **not** begin by asking:

> "Which existing patch can I modify?"

Instead think:

```text
New requested behavior
        ↓
Which Android app is being targeted?
        ↓
What does the target APK actually contain?
        ↓
Which part of the APK needs modification?
        ↓
Which Morphe patch mechanism is appropriate?
        ↓
Can an existing generic repository utility be reused?
        ↓
Can an existing app-specific implementation actually be reused?
        ↓
If yes, reuse it carefully.
If no, create the new implementation.
        ↓
Integrate it into the existing patch collection.
        ↓
Build and validate.
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anxyis/anxy-patches](https://github.com/anxyis/anxy-patches) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
