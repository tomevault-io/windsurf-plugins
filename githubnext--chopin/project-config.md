---
trigger: always_on
description: Chopin is an experimental collaborative authoring system: several people and a
---

# Working on Chopin

Chopin is an experimental collaborative authoring system: several people and a
hosted agent share one rich, repository-connected document. Plans are one
document workflow, not the product boundary. Read
[README.md](README.md) for the product framing and [Architecture](docs/architecture.md)
before changing cross-package behavior.

The most useful technical references are:

- [Authentication and authorization](docs/authentication.md)
- [Repository channels](docs/channels.md)
- [Storage and persistence](docs/storage.md)
- [Hosted agent (Planner)](docs/hosted-agent.md)
- [Background jobs and workers](docs/background-jobs.md)
- [Experimental implementation lifecycle](docs/implementation-lifecycle.md)
- [Self-hosting](docs/self-hosting.md)

## Commands

```bash
bun install             # install the workspace
bun run dev             # Vite and Bun development supervisor
bun run dev:exe         # exe.dev proxy and HMR mode
bun run db:up           # start the local PostgreSQL service
bun run db:down         # tear down the local Compose project
bun run migrate         # apply PostgreSQL migrations
bun test                # unit, domain, and memory-adapter tests
bun run test:postgres   # PostgreSQL contract and lifecycle tests
bun run e2e             # Chromium system integration suite
bun run e2e:ui          # Playwright UI mode
bun run e2e:browsers    # install Chromium once
bun run types           # TypeScript checks across packages and E2E
bun run ci              # dprint, oxlint, and token checks
bun run fix             # format and apply safe lint fixes
bun run build           # build the web client
bun run start           # start the server; build and migrate first
bun run docker:up       # build and start app plus PostgreSQL locally
bun run docker:down     # tear down the local Compose project
```

The repository pins Bun 1.3.2. Keep the package metadata, Docker image, CI, and
documentation synchronized when changing it.

`bun run e2e` starts two disposable PostgreSQL services, migrates them, builds
the client, starts applications on ports 8788 and 8789, and runs Chromium with
`AGENT=off`. Set `E2E_SKIP_BUILD=1` only when the existing client build is known
to match the checkout.

CI has three independent jobs: validation, browser integration, and a Docker
image build. A documentation-only change should still pass `bun run ci`.

## Repository map

| Area                | Responsibility                                         | Internal workspace dependencies               |
| ------------------- | ------------------------------------------------------ | --------------------------------------------- |
| `packages/dialect`  | Restricted MDX, MDAST, and Lexical schema              | none                                          |
| `packages/protocol` | WebSocket declarations and addressing helper           | none                                          |
| `packages/question` | Questionnaire definitions and shared drafts            | `protocol`                                    |
| `packages/viewport` | Browser geometry and subscriptions                     | none                                          |
| `packages/editor`   | Collaborative editor, decisions, comments, and widgets | `dialect`, `question`, `protocol`, `viewport` |
| `apps/server`       | Auth, channels, rooms, storage, Planner, MCP, tasks    | `dialect`, `question`, `protocol`             |
| `apps/web`          | Repository picker, navigation, conversation, workspace | `dialect`, `editor`, `protocol`, `viewport`   |
| `e2e`               | Browser and system integration harness                 | may import server internals as fixtures       |

Runtime workspace packages do not depend on an application. E2E and skill
contract tests may deliberately import server internals; do not treat those test
harnesses as runtime package boundaries.

## Runtime model

The browser edits Lexical bound to Yjs. One WebSocket multiplexes session,
document (`plan:*` on the wire), conversation, questions, comments, and
implementation lifecycle messages. The server keeps each open channel as an
authoritative Y.Doc with a headless Lexical mirror so it can validate and
serialize the document without trusting a browser.

Human updates are grouped for 5 ms, applied, projected to canonical MDX,
validated, and committed with sidecar state before acknowledgement or relay. An
invalid Yjs batch cannot be undone; the room rebuilds the latest known-good
state under a fresh epoch and clients reopen.

The Planner reads a plan revision and edits through structural block operations.
Operations stage against MDAST, pass dialect and Lexical round-trip validation,
reconcile into the live tree, and produce one Yjs delta. Unchanged or moved
MDAST object identity preserves existing Lexical nodes, selections, and undo
history.

PostgreSQL is the only runtime storage adapter. One renewable `chopin:writer`
lease permits one application process per database. Process-local browser
sessions and GitHub credentials are cleared on startup; collaborative state and
external implementation runs are durable.

## Authority and security

- **The dialect is an allowlist.** Document MDX is parsed and rendered, never

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [githubnext/chopin](https://github.com/githubnext/chopin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
