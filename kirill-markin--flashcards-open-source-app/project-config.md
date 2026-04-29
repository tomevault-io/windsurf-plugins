---
trigger: always_on
description: Open-source, offline-first flashcards platform.
---

# flashcards-open-source-app

Open-source, offline-first flashcards platform.

- Domain: `flashcards-open-source-app.com`
- Repository: `kirill-markin/flashcards-open-source-app`
- Product stage: early development

## Product Direction

Build a practical Anki-like alternative focused on fast mobile UX, offline-first behavior, and transparent open-source architecture.

## Platform Plan

- Backend: AWS
- AWS infra deploys via CI/CD only. Do not run AWS deploys locally; push to `main` and watch CI/CD.
- Do not build AWS SDK bundles or other AWS deployment artifacts locally. Push code to `main`, and let CI/CD build and deploy everything.
- Web app: supported. Before making any web change, read [apps/web/README.md](apps/web/README.md).
- iOS app: Swift (priority). Before making any iOS change, read [apps/ios/README.md](apps/ios/README.md).
- Android app: Kotlin + Jetpack Compose. Before making any Android change, read [apps/android/README.md](apps/android/README.md).
- Terminal / AI-agent API client: supported via the canonical machine API entrypoint `GET https://api.flashcards-open-source-app.com/v1/` (the same discovery payload is also available at `GET https://api.flashcards-open-source-app.com/v1/agent`)

For iOS App Store screenshots and derived marketing materials, read [apps/ios/docs/marketing-screenshots.md](apps/ios/docs/marketing-screenshots.md).

We support the web app, the iOS app, the Android app, and the terminal-first AI-agent API flow. When making changes, we try to keep all supported clients aligned where relevant.
The platform READMEs are part of the working agreement for client work: [apps/web/README.md](apps/web/README.md) for web changes, [apps/ios/README.md](apps/ios/README.md) for iOS changes, and [apps/android/README.md](apps/android/README.md) for Android changes.
Before bumping release versions anywhere in the repo, read [docs/version-bump.md](docs/version-bump.md) and follow that flow so backend, web, Android, iOS, runtime-reported versions, and release metadata stay aligned.
The Web, iOS, and Android clients are multilingual, so always account for localization and follow the existing translation patterns in the surrounding code and dedicated client documentation.

## Testing Philosophy

We do not try to cover the whole product with tests, and we do not optimize for blanket test coverage metrics.
Do not add unit tests by default, and do not guard every helper or pure function with a dedicated test.
Do not add new unit tests for this repository as part of implementation or as part of a written plan unless the user explicitly asks for unit tests.
Targeted integration or module-boundary tests are allowed when they validate a real interaction between modules, storage layers, or contracts.
The most trusted tests are the native/browser smoke flows that run the real app on a simulator, emulator, managed device, or deployed environment because they are the closest checks to production behavior.
Keep tests minimal and focused on primary user flows, critical contracts, or important cross-module behavior.
When proposing a test plan, treat "real testing" as one of these two options only: either implement a real integration/end-to-end/smoke flow that exercises the app or API for real, or provide explicit manual test instructions for the user to run. Do not treat unit tests as the default validation plan.

For iOS local testing details, see [docs/ios-local-setup.md](docs/ios-local-setup.md).
For Android local testing details, see [docs/android-ci-cd.md](docs/android-ci-cd.md).
For the optional private analytical DB access path, granted reporting permissions, and operator setup flow, see [docs/analytics-db-access.md](docs/analytics-db-access.md).

## Release Gates and Monitoring

Pushes to `main` use three independent release streams: AWS/web, Android, and iOS.
Details, rollback rules, and live smoke references: [docs/release-gates.md](docs/release-gates.md).

## Repository Strategy

Use a single monorepo for now because it keeps shared API contracts and the data model together, makes coordinated backend/mobile changes easier, and lowers operational overhead at this stage.

## Architecture

- Source of truth: Postgres in AWS
- Mobile local database: SQLite on device
- Sync model: offline-first synchronization via API

## Database Decision

Use Postgres as the primary backend database because flashcards data is relational (workspaces, cards, review events, sync metadata), transactions/consistency matter, and schema/query evolution for scheduling, stats, and filtering is simpler than with DynamoDB as primary.

## Offline-First Sync Rules

- Read and write locally to SQLite first.
- Queue local writes in an `outbox`.
- When connectivity returns:
  1. push pending operations to the backend (idempotent by operation id)
  2. pull remote changes since the last sync cursor
  3. apply updates locally
  4. advance the sync cursor and clear acknowledged outbox rows

The sync approach and the data model changed multiple times during development. To understand how the system works now, read `docs/architecture.md` together with the full relevant migration chain in `db/migrations`, and do not infer current behavior from only one or two isolated migrations.

## Data Modeling Notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kirill-markin/flashcards-open-source-app](https://github.com/kirill-markin/flashcards-open-source-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
