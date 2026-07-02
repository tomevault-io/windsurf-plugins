---
trigger: always_on
description: A banking platform reference implementation: a Quarkus/Kotlin monorepo of ~34 `openbank-*`
---

# OpenBank — agent & contributor guide

A banking platform reference implementation: a Quarkus/Kotlin monorepo of ~34 `openbank-*`
microservices, a Next.js `openbank-admin-ui`, and a shared `openbank-libs`. Hexagonal
architecture per service (ADR-0002).

**Authoritative rules live in [`openbank-libs/governance/rules.yaml`](openbank-libs/governance/rules.yaml).**
This file is the human-readable summary; `rules.yaml` is what CI enforces — when they disagree,
`rules.yaml` wins. [`CONTRIBUTING.md`](CONTRIBUTING.md) is the long-form narrative.

## The non-negotiables

Before a change is "done", run the ship-checklist (`/ship-check` runs the same checks CI gates on,
ADR-0029):

1. **Open a PR.** No direct commits to `main`. Branch `<type>/<scope>-<summary>`; squash-merge via PR.
2. **Versioning is automatic.** A change under `<service>/src/main/**` is released by **release-please**
   from your Conventional Commit — so the commit message *is* the changelog. Do **not** hand-edit
   `version.txt`, `CHANGELOG.md`, or `quarkus.application.version` (it derives from `version.txt`).
   A module is a released component **iff** it has a `version.txt` (registered in
   [`release-please-config.json`](release-please-config.json) + [`.release-please-manifest.json`](.release-please-manifest.json)).
3. **API change ⇒ `openapi.yaml` updated + contract test.** Two independent version axes (ADR-0048):
   the **release** version (`version.txt`) and the **API-contract** version
   (`openapi.yaml:info.version`, whose major == URL `/api/v{N}`). An API change classifies its own bump
   from the OpenAPI diff (`oasdiff`), never forced equal to the release version.
4. **DB change ⇒ Flyway migration + rollback note. Event change ⇒ schema versioned backward-compatibly.**
   **Config change ⇒ no duplicate YAML keys** in `application.yaml` — SmallRye/SnakeYAML keep only the
   *last* of a repeated mapping key and silently drop the rest (CI enforces this).
5. **Test the new behavior.** Coverage is ratchet-only (never lower); money-path services aim higher.
6. **Derived data is never hand-edited.** Catalog, coverage, and the governance manifest are
   CI-generated — edit the source, not the artifact.
7. **Money-path services** (`rules.yaml: money_path_services`) need 2 approvals + a threat model
   (`docs/threat-models/<service>.md`, ADR-0030).

## Commit format

```
<type>(<scope>): <imperative summary>
```
`type` ∈ feat|fix|perf|refactor|docs|test|chore|build|ci|security. `scope` = service without the
`openbank-` prefix (e.g. `ledger`, `sepa-payment`) — **the scope selects the released component**.
Sign every commit: `git commit -s -S`. Full vocabulary in `rules.yaml: commits`.

## Issues — the actionable backlog (ADR-0052)

Issues track *what needs doing*; they don't duplicate ADRs (decisions) or release-please (changelog).
Open one for a **fleet sweep**, a **governance follow-up** (the actionable tail of an ADR), a **bug**,
or an **enhancement** — not for architectural decisions (→ `docs/adr`), questions (→ Discussions), or
security holes (→ private Security Advisories). Every PR links its issue (`Closes #<n>` / `Refs #<n>`).
Labels are code (`.github/labels.yml`, applied by the Label-sync workflow) — don't create them by hand.

## Build

```
./gradlew :<module>:build                          # one service
./gradlew detekt ktlintCheck koverVerify build     # the local gate before a PR
```
CI is path-scoped (only changed services build). The domain layer has **zero** framework imports.

## Skills

- `/ship-check` — authoritative pre-merge preflight; mirrors the CI gates.
- `/bump <service>` — raise `version.txt` + `openapi.yaml` info.version per change type.
- `/open-pr` — branch, PR from template, verify bump + changelog present.
- `/release <service>` — assemble release notes (release-please).

## Engineering notes (common pitfalls)

These are real, repeatable gotchas — worth knowing before they cost you a debugging session.

### Kotlin / Quarkus
- **Always fast-jar, never uber-jar.** Service Dockerfiles use `-Dquarkus.package.jar.type=fast-jar`
  and COPY `quarkus-app/`; an uber-jar leaves `quarkus-app/` empty → crashlooping pod.
- **`@ConfigProperty` optional fields must be `Optional<String>`,** not plain `String`, or a missing
  value throws `SRCFG00040` at boot. Use `Optional<String>` + `defaultValue`.
- **Kotlin JUnit5 + `runBlocking` silent drop.** `fun foo() = runBlocking { }` infers a non-`Unit`
  return type and JUnit5 ignores the method. Write `fun foo(): Unit = runBlocking { }` or use a
  coroutine test runner.
- **`openbank.outbox.dispatch-enabled` defaults to `false`.** Any service with an outbox entity must
  set it `true` in `application.yaml`, or events never dispatch (no error, `attempt_count` stays 0).
- **CDI wiring isn't validated by `ktlintCheck` + unit tests.** Add `:svc:quarkusBuild` to your
  pre-push gate; ArC/CDI failures only surface there.

### ktlint
- Path-scoped CI only lints changed files, so a pre-existing wildcard import or a latent
  `function-signature` violation surfaces the first time you touch an older file. Let `ktlintFormat`
  collapse multi-line signatures; expand wildcard imports rather than hand-wrapping.

### Flyway

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JiRaska/open-bank-oss](https://github.com/JiRaska/open-bank-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
