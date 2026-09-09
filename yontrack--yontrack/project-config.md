---
trigger: always_on
description: Yontrack (formerly Ontrack) is a **continuous delivery monitoring platform**. It tracks projects,
---

# Yontrack (Ontrack) Developer Guide for Claude Code

Yontrack (formerly Ontrack) is a **continuous delivery monitoring platform**. It tracks projects,
branches, builds, promotions, and validations across the software delivery lifecycle.

## Tech Stack

- **Backend:** Kotlin + Java, Spring Boot 3.5+, Spring GraphQL, Spring Security
- **Frontend:** React, Next.js 13, Ant Design v5, graphql-request
- **Database:** PostgreSQL with Flyway migrations
- **Search:** Elasticsearch
- **Messaging:** RabbitMQ
- **Auth:** OIDC/JWT
- **Build:** Gradle (Kotlin DSL), JDK 21 required

---

## Rules

These rules apply unconditionally. Follow them in every change, without exception.

### Database
- **Never** modify existing Flyway migration files — always add a new one
- **Never** put a Flyway migration in a patch release. A patch is cherry-picked onto a
  `release/X.Y` branch while `main` keeps moving, so a `V82` on the patch branch and an unrelated
  `V82` on `main` give any user upgrading from the patch to the next minor a checksum conflict. If
  a fix needs a schema change it is not a patch — see `doc/dev-guide/patch-release.md`.
- **Always** add `ON DELETE CASCADE` on FK references to entity tables
- Use `SERIAL PRIMARY KEY NOT NULL` for auto-increment primary keys

### Frontend
- **Never** access `localStorage` directly — always use the wrapper functions in `@components/storage/local`
- **Never** introduce a new usage of the deprecated `useGraphQLClient` hook — always use `useQuery`,
  `useMutation` or `callGraphQL` from `@components/services/GraphQL`. This holds even inside a file
  that still uses `useGraphQLClient` elsewhere.
- **Always** import `useQuery` from `@components/services/GraphQL` — the identically named hook in
  `@components/services/useQuery` is deprecated (it wraps `useGraphQLClient`)
- **Never** store a value in `useState` + `useEffect` when it's purely derived from props/state —
  compute it directly in the render body instead (e.g. `const items = changeLog ? [...] : []`, not
  `useState([])` filled by a `useEffect`). Beyond being an unnecessary extra render, a value that's
  briefly wrong/empty on first render before the effect fires can break children that make first-render
  assumptions — e.g. `GridTable`'s `items` starting empty while `layout` was already fully populated
  made `react-grid-layout` sync its internal layout against 0 children, permanently collapsing every
  widget to a default 1x1 slot once the items arrived a tick later (issue #1634). See `BuildContent`
  for the correct pattern: compute `items` as a plain `const` from already-available props.

### Property Types
- **Never** rename a `PropertyType` class after it is deployed — its fully qualified class name (FQCN) is its persistent storage ID

### Documentation
- **Never** edit `ontrack-docs/src/docs/asciidoc/` — that tree is dead. No asciidoc plugin remains in the
  build, so nothing there is ever published. User documentation lives in **mkdocs** under
  `ontrack-docs/docs/content/`, and a new page must be added to the `nav:` in `ontrack-docs/mkdocs.yml`
  or it will not be reachable.
- **Never** hand-edit `ontrack-docs/docs/content/generated/` — it is gitignored and rebuilt by the
  `ontrack-docs` integration tests from `@APIDescription` and the event/metric/property declarations.
  To change generated docs, change the annotations, then run `./gradlew :ontrack-docs:integrationTest`.
- Verify docs changes with `./gradlew :ontrack-docs:buildDocs`, which renders the site into
  `ontrack-docs/site/` and surfaces broken links and missing nav entries.

### Running Yontrack locally

- **Always** start Yontrack with `scripts/dev-stack.sh up` — never launch the middleware, the backend
  or the frontend by hand. One `up` starts all three, waits until each answers, installs the npm
  dependencies on a fresh checkout, and prints the URLs it allocated.
- **Never** assume `localhost:3000` or `localhost:8080`. Each checkout gets its own ports so that
  several worktrees can run at once: the main working copy keeps the historical ports, and a linked
  worktree offsets every port. Read the actual ones from `.yontrack-dev/instance.env` in the
  checkout, or from the output of `scripts/dev-stack.sh status`.
- When something fails to start, the stack is deliberately left running — read
  `scripts/dev-stack.sh logs backend` (or `frontend`, or `infra`) rather than restarting blindly.
- After a Kotlin change, `scripts/dev-stack.sh restart backend` — a full `down`/`up` needlessly pays
  for the containers again.
- `scripts/dev-stack.sh down` keeps the data; only `down --clean` drops the volumes. Never use
  `--clean` on a stack you did not create.
- Log in through Keycloak with `admin`/`admin`.
- The API itself is Bearer/OIDC only — `curl -u admin:admin` gets a 401. To call it, seed the
  demo, or run a Playwright spec against the stack, mint a token first:
  `docs/agents/local-api-access.md`.

### Workflow

Every change follows this lifecycle, end to end — don't stop after step 2:

1. **Branch locally** — before making any change, create a branch named `claude/<short-description>-pipeline`
   (use the `/fix-issue` skill when working from a GitHub issue)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yontrack/yontrack](https://github.com/yontrack/yontrack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
