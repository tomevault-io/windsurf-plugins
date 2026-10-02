---
trigger: always_on
description: An **accelerator for WSO2 Identity Server 7.3.0** — not a standalone application. The build
---

# Working in this repo

## What this is

An **accelerator for WSO2 Identity Server 7.3.0** — not a standalone application. The build
produces a zip that is unpacked over an existing IS distribution: OSGi bundles into
`repository/components/dropins`, a WAR into `repository/deployment/server/webapps`, and a
complete `deployment.toml` that replaces the product's own. Nothing here runs on its own; almost
everything needs a live IS to exercise.

The rest of this file is conventions that have emerged across the codebase — follow them when
adding a feature, a module, or a config option, since they're not written down anywhere else yet.

## Build

Requires JDK 11+ (JDK 21+ to run the server), Maven 3.6.3+, Node.js 20.19+/22.12+ with npm.

```sh
mvn clean install                    # from the REPOSITORY ROOT
```

Run from the repository root, **not** from `dpdp-accelerator/`. The root pom aggregates
`dpdp-accelerator` *and* `dpdp-accelerator/accelerators` separately (the accelerators subtree
parents to the root pom, matching the Financial Services accelerator layout), so building from
`dpdp-accelerator/` silently skips the accelerator zip and only builds the portal. See
[`README.md`](README.md) for more background on prerequisites.

Output: `dpdp-accelerator/accelerators/dpdp-is/target/wso2-dpdpiam-accelerator-<version>.zip`

### Tests

| Scope | Command |
| --- | --- |
| Java (TestNG via surefire, suite defined in `src/test/resources/testng.xml`) | `mvn test` |
| Single Java test class | `mvn test -pl dpdp-accelerator/components/org.wso2.dpdp.accelerator.identity.extensions -Dtest=DPDPConsentPortalAppProvisioningUtilTest` |
| Frontend (Vitest) | `cd dpdp-accelerator/react-apps/consent-portal/frontend && npm test` |
| Single frontend test | `npm test -- src/__tests__/SomeThing.test.tsx` |
| Frontend lint / format | `npm run lint` / `npm run format:check` |
| E2E (Playwright, needs a deployed IS) | `cd dpdp-integration-test-suite && ./run-e2e.sh [tests/03-consents]` |

CI splits into four workflows. `.github/workflows/pr-build.yml` builds the Java and frontend
modules on every PR to `main` and `dev`. The E2E suite is not automatic: `pr-e2e.yml` deploys an
Identity Server from scratch and runs Playwright against it only once a maintainer applies the
`Action/trigger-e2e` label, because the job runs PR code with write permissions and repository
secrets in scope. `pr-e2e-gate.yml` strips that label on every new push and publishes the
`E2E (label-gated)` commit status, so the label can never carry over to unreviewed code.

`pr-e2e.yml` runs only the `multi-tenant` Playwright project - `super-tenant`'s own coverage
(unqualified-root routing, the tenant-provisioning-skip path) isn't exercised on every PR. Instead
`nightly-e2e.yml` runs every project (`e2e.yml`'s own default when its `projects` input is empty)
against the default branch nightly, so a super-tenant-only regression surfaces within a day rather
than going unnoticed until the Saturday weekly jobs or a release gate. `release-builder.yml`
likewise passes no `projects` override, so a release is still gated on every project.

Every caller of `e2e.yml` takes its `db_type` default, `mysql`, except `nightly-e2e.yml` and
`release-builder.yml`, which run it as a matrix over every database type (`h2`, `mysql`,
`postgresql`), in parallel and with `fail-fast: false`; a manual nightly dispatch can narrow it to
one database and/or one project. A PR is gated on MySQL alone.

The Identity Server under test comes from the `updates2.0` S3 bucket (`IS_PACK_S3_URI`) with U2
updates applied. The published GitHub release zip is *not* U2-updatable — don't reintroduce that
path. `e2e.yml` still accepts `is_source: master`, which `weekly-e2e-is-master.yml` runs on a
schedule so upstream breakage surfaces before the next IS upgrade rather than during it. The
updated pack is cached, and **only `workflow_dispatch` / `schedule` / `push` runs may write that
cache** — never the labelled-PR path, or a PR could poison the pack for every later run,
including the release gate. Keep the restore read-only. Cache entries are immutable, so
`nightly-e2e.yml` refreshes it: unless a manual run ticks `use_cached_is_pack`, it deletes the
default branch's `is-pack-*` entry, rebuilds the pack at the latest U2 level once
(`e2e.yml` with `pack_only`), saves it, and runs every database leg on that same pack.
`release-builder.yml` does the same whenever its E2E gate runs from the default branch; from any
other branch each database leg applies the latest U2 itself. Either way a release is gated on the
latest U2 level, never on whatever the cache holds.

Role *membership* is the one thing the accelerator never provisions, so both CI and a fresh local
install get their accounts from `dpdp-integration-test-suite/scripts/provision-test-users.sh`
(idempotent).

**Use npm, not pnpm.** `package-lock.json` is the committed lockfile and the Maven build invokes
`npm install` / `npm run build`. The frontend `README.md` and `AGENTS.md` both say pnpm — they are
stale on this point; don't follow them for package management even though they're otherwise the
canonical frontend policy (see [Frontend conventions](#frontend-conventions) below).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wso2/dpdp-accelerator](https://github.com/wso2/dpdp-accelerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
