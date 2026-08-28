---
trigger: always_on
description: This file applies to the entire repository. A nested `AGENTS.md` may add or override rules only for its own directory. The user's current request controls the task; repository instructions define how to perform it safely and how to prove completion.
---

# DeliveryGuard repository instructions

## Scope and precedence

This file applies to the entire repository. A nested `AGENTS.md` may add or override rules only for its own directory. The user's current request controls the task; repository instructions define how to perform it safely and how to prove completion.

DeliveryGuard is a public, clean-room delivery validation toolkit. Keep code, documentation, examples, tests, and history suitable for public release.

## Responsibility and success

Complete requested code, schemas, documentation, tests, and local validation within the authorized scope. Work is complete only when the requested outcome exists, relevant checks pass or are accurately reported, unrelated work is preserved, and remaining risks or external blockers are explicit.

Do not confuse a plan, an edited file, a submitted source revision, successful tests, acceptance, deployment, or release. Each claim needs its own evidence.

## Authority by request type

- For answers, reviews, explanations, diagnoses, and status reports, perform relevant read-only inspection; do not implement a fix unless requested.
- For planning and design, inspect the repository and create requested local artifacts; do not silently begin implementation.
- For build, change, implement, or fix requests, make scoped, reversible local edits and run proportionate verification.
- Publishing, pushing, deploying, messaging, changing remote configuration, mutating external data, or acting in production requires explicit authorization for the exact target.
- Destructive cleanup, history rewriting, force pushes, broad deletion, and irreversible migrations require explicit approval.

Infer low-impact details from repository facts. Ask only when missing information changes behavior, target, authorization, security, or an external result.

## Repository map and facts

- `deliveryguard.config.json` defines repositories, environments, paths, document types, and policies.
- `.deliveryguard/versions/` contains version facts; `.deliveryguard/acceptance/` contains Evidence Manifests; `.deliveryguard/repairs/` contains Repair Cases.
- `openspec/changes/` contains active specification changes.
- `schemas/` is the public JSON Schema contract. Keep it aligned with `src/types.ts`, semantic validation, examples, and documentation.
- `src/` contains the TypeScript API and CLI. `dist/` is generated output.
- `examples/synthetic-shop/` is the end-to-end fixture and must remain wholly synthetic.
- `.agents/skills/` is the repository-discoverable skill suite. `templates/codex/.agents/skills/` is the npm initializer mirror; tests require byte-for-byte equality.
- `templates/default/` contains files installed by `deliveryguard init`. Never make initialization overwrite existing user files.

Use pnpm only. Support the Node range declared in `package.json`.

## Delivery fact contract

Lifecycle stages are derived from facts and are never manually overridden:

```text
planned -> specified -> implemented -> verified -> released
```

- A version is `specified` only when its registered documents and required OpenSpec artifacts are valid.
- OpenSpec records intent and task completion; it is not source, acceptance, or release evidence.
- A version is `implemented` only when every required repository has qualifying source facts.
- Acceptance requires complete document-to-requirement-to-case coverage and repository-relative evidence.
- Acceptance and production release are independent facts. Tests or acceptance must never imply release.
- A release requires successful production facts and concrete anchors for every required repository, plus acceptance when policy requires it.

Record only facts that exist. Missing evidence stays missing or `pending`; do not invent branches, commits, URLs, timestamps, reports, or deployment anchors.

## Repair Cases

Use a Repair Case for a reproducible defect when red-green-regression evidence is meaningful. A verified repair requires a failing baseline revision, a different passing candidate revision, and passing regression checks. Keep configuration, data, dependency, environment, and test-infrastructure failures distinct from code defects.

Repair commands are untrusted argv arrays. Execute them only through an explicit `deliveryguard repair run`, with repository-relative working directories, bounded timeouts, and `shell: false`. `deliveryguard check` must never execute repair commands.

## Public and security boundaries

- Keep examples, screenshots, evidence, names, URLs, identifiers, and commits fictional or explicitly public. Use `.invalid` domains for synthetic hosts.
- Never copy proprietary source, private documents, messages, screenshots, production data, credentials, internal repository history, or absolute user paths.
- Do not add deployment, messaging, business-data, credential, or proprietary platform integrations to the core.
- Bundled skills may describe provider-neutral plans and evidence contracts, but must not embed private adapters, endpoints, environment coordinates, business schemas, or automatic external writes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wzf1997/delivery-harness](https://github.com/wzf1997/delivery-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
