---
trigger: always_on
description: BootUI is a **Spring Boot 4 starter** that adds a local-only developer console (Vue 3 SPA + REST API) to a host Spring Boot 4 app. The authoritative scope/behavior lives in `docs/SPECIFICATION.md`, `docs/PLAN.md`, and `docs/FEATURES.md` — read those before changing public behavior or visible panel behavior.
---

# BootUI — Copilot instructions

BootUI is a **Spring Boot 4 starter** that adds a local-only developer console (Vue 3 SPA + REST API) to a host Spring Boot 4 app. The authoritative scope/behavior lives in `docs/SPECIFICATION.md`, `docs/PLAN.md`, and `docs/FEATURES.md` — read those before changing public behavior or visible panel behavior.

## Toolchain

- Java 25, Spring Boot 4.0.x (`spring-boot.version` in root `pom.xml`; currently 4.0.6).
- Maven Wrapper (`./mvnw`) using Maven 3.9.16; do not require a system Maven install.
- Published Maven coordinates use `com.julien-dubois.bootui:*`; Java packages remain `io.github.jdubois.bootui.*`.
- Node.js / npm for the packaged Vue app are downloaded automatically by the `frontend-maven-plugin` (`node.version` / `npm.version` in root `pom.xml`); do not add a manual Node install step for the Maven build.

## Build, run, test

```bash
# CI-equivalent multi-module build (downloads Node, builds Vue UI, packages all JARs).
./mvnw -B -ntp clean install

# Backend-only iteration loop (skips the Vue build).
./mvnw -pl bootui-core,bootui-autoconfigure,bootui-spring-boot-starter,bootui-sample-app -am install

# Run the sample app (smoke-test path: http://localhost:8080/bootui).
./mvnw -pl bootui-sample-app spring-boot:run -Dspring-boot.run.profiles=dev

# Single test class / single test method.
./mvnw -pl bootui-core test -Dtest=SecretMaskerTests
./mvnw -pl bootui-core test -Dtest=SecretMaskerTests#detectsCommonSecretKeys

# Front-end inner loop (Vite dev server with HMR; proxies /bootui/api/* to a running sample app).
(cd bootui-ui/src/main/frontend && npm install && npm run dev)
# After changing UI code that needs to be re-bundled into the JAR:
./mvnw -pl bootui-ui install

# Browser end-to-end suite (required for UI, browser-facing API, or sample-app changes).
(cd bootui-sample-app/e2e && npm ci && npx playwright install --with-deps chromium && npm test)

# Maven Central release path for non-SNAPSHOT versions.
./mvnw -B -ntp -Prelease clean deploy
```

CI (`.github/workflows/build.yml`) runs `./mvnw -B -ntp clean install` on Java 25, installs Playwright Chromium, and runs `bootui-sample-app/e2e` with `npm test`. CodeQL covers Java/Kotlin and JavaScript/TypeScript when code scanning is enabled. The release workflow (`.github/workflows/release.yml`) publishes `v*` tags to Maven Central through the `release` Maven profile and the Sonatype Central Publishing plugin.

## Release plumbing (Maven Central)

A few subtle constraints that have already burned us in past releases — preserve them when touching `pom.xml` files or the release profile:

- **Source-less modules (`bootui-ui`, `bootui-spring-boot-starter`) must attach their empty `javadoc.jar` at phase `package`, not `verify`.** The parent's `release` profile binds `maven-source-plugin`, `maven-javadoc-plugin`, and `maven-gpg-plugin:sign` all to `verify`, and gpg runs before any child-pom executions in the same phase. If the empty javadoc is attached at `verify`, gpg signs everything *except* the javadoc.jar, and Sonatype Central rejects the deployment with `Missing signature for file: ...-javadoc.jar`. If you add another source-less module, copy the existing `attach-empty-javadocs` execution (phase `package`, classifier `javadoc`, `skipIfEmpty=false`, fed from `target/empty-javadocs`).
- **The signing GPG public key must be queryable by fingerprint on `keys.openpgp.org` and/or `keyserver.ubuntu.com`.** Sonatype Central validates signatures against those keyservers; if the public key isn't there, every signature comes back `Invalid signature ... Could not find a public key by the key fingerprint`. After rotating the `GPG_PRIVATE_KEY` secret, re-publish the matching public key. On macOS, `gpg --send-keys` often fails with `Invalid argument` from dirmngr; the reliable fallback is the HTTPS upload APIs (`POST https://keys.openpgp.org/vks/v1/upload` with a JSON `{"keytext": ...}` body, and `POST https://keyserver.ubuntu.com/pks/add` with form field `keytext`).
- **A failed deployment still consumes the version coordinate in Central.** Once the publishing plugin uploads `com.julien-dubois.bootui:<artifact>:<version>` — even if validation rejects it — you cannot re-upload that exact GAV without dropping the failed deployment from the Sonatype Central Portal first. The default path is to bump the version (`./mvnw -B versions:set -DnewVersion=… -DgenerateBackupPoms=false`), commit, tag `v<version>`, and let the tag push trigger `release.yml` again.
- **Use the `Prepare Release` workflow (`.github/workflows/prepare-release.yml`) to bump versions** rather than running `versions:set` by hand. It runs `versions:set` *and* `perl`-substitutes the old version into `README.md` (the install snippet there is the only documentation that references a specific version), then verify-builds, commits, tags, and pushes. Manual `versions:set` skips the README rewrite, leaving the install snippet pointing at a stale (and possibly never-published) version.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jdubois/boot-ui](https://github.com/jdubois/boot-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
