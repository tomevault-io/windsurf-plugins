---
trigger: always_on
description: Expojet is an Expo-first project generator. It turns one validated configuration into either an Expo SDK 57 app or a full-stack TypeScript workspace. The generator combines a checksum-verified base pack with declarative adapters, checks the rendered tree, then moves it into place atomically.
---

# Expojet

Expojet is an Expo-first project generator. It turns one validated configuration into either an Expo SDK 57 app or a full-stack TypeScript workspace. The generator combines a checksum-verified base pack with declarative adapters, checks the rendered tree, then moves it into place atomically.

## What must stay true

These constraints matter more than local convenience.

### Generation is deterministic

The same input and SDK pack must produce the same files. Do not fetch templates or dependency versions while building a generation plan. Do not make generated output depend on machine-specific state, timestamps, or network responses.

The SDK 57 template in `packages/sdk-packs/sdk-57/template` is the canonical base. Changes to it must go through `pnpm sdk:sync`, which updates the generated TypeScript module and SHA-256 checksum. Never edit `template.generated.ts` by hand.

### Adapters describe changes

Adapters return typed `Operation[]` values. They do not write files, run package managers, prompt users, or mutate shared state. `packages/core/src/executor.ts` owns filesystem writes and applies the completed plan.

If an integration cannot fit the current operation model, improve the operation model first. Do not hide imperative work inside an adapter.

### Generation is atomic

Render into the unique sibling staging directory created by the executor. Validate there, then commit with a rename. On failure or dry run, remove only that staging directory. Never leave a target half-generated and never clean an existing target to make generation succeed.

All plan paths must pass through the core path guards. Generated trees cannot contain symlinks or unresolved Expojet template tokens.

### Mobile code never receives server secrets

Code under a generated mobile workspace cannot contain database URLs, service-role keys, auth secrets, or backend-only environment modules. Variables intentionally exposed to Expo code must start with `EXPO_PUBLIC_`.

Keep database drivers, migrations, and server credentials inside `apps/api` for generated monorepos. Shared packages may contain types, not backend runtime imports.

### Branding has one owner

Product names, commands, manifest filenames, package names, and taglines come from `@expojet/brand`. Do not hardcode Expojet identity in generator logic or adapter output when the brand package already owns the value.

## A note on taste

Prefer the smallest model that makes correct output unsurprising. Expojet already has many combinations. A new abstraction has to remove more branching than it adds.

Generated code is the product. A tidy adapter that emits awkward app code is still a bad change. Read the generated files as if you had just started a real project with them. Dependencies should make sense, setup should be visible, and unsupported combinations should fail before any files are written.

Do not preserve complexity because it exists. Do not add a framework-shaped layer for one integration. Make the constraint explicit, put it in the narrowest package that can own it, and keep the rest of the pipeline boring.

## Shared language

Use these terms consistently in code, docs, and reviews:

- **input** is the validated `CreateInput` that describes one requested project.
- **SDK pack** is a versioned, checksum-verified Expo base template.
- **adapter** is a declarative integration for auth, styling, navigation, backend, database, or another project choice.
- **operation** is one typed file, JSON, dependency, environment, Metro, or app-config change.
- **plan** is the destination plus the complete ordered list of operations.
- **executor** detects conflicts, renders a plan, validates the result, and commits it.
- **staging directory** is the temporary sibling directory used before the atomic rename.
- **manifest** is the generated `expojet.jsonc` record of SDK, structure, adapters, and feature choices.
- **fixture** is a checked-in reference project for a supported configuration.
- **stable adapter** is available through the normal CLI flow.
- **experimental adapter** requires explicit opt-in and has an incomplete promotion gate.

## The easiest ways to break Expojet

1. **Write from an adapter.** Direct filesystem access bypasses conflict detection, dry-run behavior, path guards, and atomic cleanup.
2. **Change one layer of a choice.** Adding a schema enum without CLI flags, prompts, adapter dispatch, doctor checks, fixtures, and docs produces a choice that only partly exists.
3. **Patch generated files from several owners.** Overlapping writes become order-dependent. Use the JSON, dependency, Metro, and app-config composition operations. Let conflict detection reject ambiguous ownership.
4. **Leak a backend value into mobile code.** An environment variable name alone can become public bundle content. Keep the boundary explicit and run the doctor checks against generated output.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Owusu1946/stackjet](https://github.com/Owusu1946/stackjet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
