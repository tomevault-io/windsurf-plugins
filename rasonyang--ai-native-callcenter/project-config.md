---
trigger: always_on
description: `CLAUDE.md` (Claude Code) and `AGENTS.md` (Codex) carry the same text; change them together.
---

# AGENTS.md

`CLAUDE.md` (Claude Code) and `AGENTS.md` (Codex) carry the same text; change them together.

## What this is

An open-source AI-native call center: one Go binary (chi/pgx/sqlc/slog/OTel) serving a REST API, an SSE stream and an embedded React SPA. It drives FreeSWITCH over ESL for human agents (WebRTC agents on the web-sip-phone Chrome extension, queues on mod_callcenter) and terminates its own SIP/RTP for AI calls. Single tenant: no `tenant_id` anywhere. Apache-2.0: new Go, SQL, Lua and script files start with an `SPDX-License-Identifier: Apache-2.0` line.

Requirements and owner decisions (A1, A6, A7, …): `docs/phase1-decisions.md`. Design: `docs/design/NN-*.md`. Live findings that amended the design: `docs/design/*-findings.md`; check them before trusting a design doc's original claim. `docs/design/07-naming.md` is the **mandatory naming spec**: Go `CallID` ↔ JSON `callId` ↔ TS `callId` ↔ DB `call_id`; SCREAMING_SNAKE enum values byte-identical across JSON/TS/DB; `xxxAt`/`xxxMs`/`xxxSec`; `is_`/`has_` booleans; no upstream FreeSWITCH/Genesys tokens outside boundary layers.

## The API is the product; the UI is optional (owner directive)

`docs/openapi.json` is the product surface. The embedded SPA is one consumer of it, with no more privilege than a customer's integration. When a screen and the contract disagree about what an operation means, the contract is right.

- No route exists that the contract does not declare, and every operation is routed (`TestEveryMountedRouteDeclaresItsAuthorization` in `internal/httpapi/contract_gate_test.go`, `TestEveryContractOperationIsRouted`).
- Anything a session cookie can reach, a properly scoped API key can reach (`TestASystemCanReachWhatAPersonCan`, two registered exceptions). A rule written because "the panel does not need it" is in the wrong place.
- Authorization is scopes. Each operation's `security` block is generated into `api.OperationSecurityByRoute` and enforced by one middleware inside the generated wrapper. There are no role guards on routes; a role only decides which scopes a login is granted (`grantedScopes`, derived by `docs/auth/scopemap.py`). A session cookie and `Authorization: Bearer <key>` are equal credentials. Design: `docs/design/04-api-sse.md` §2.

## Language

Commit messages, code comments and documentation are in English (owner directive; the repository is public). Other languages appear only in product content: the `zh` half of bilingual flows, prompts and UI labels, `README.zh-CN.md`, and the `zh` i18n resources.

## Commands

```sh
go build ./...
go test -race ./...                               # always -race (`make test` runs this, then the web tests)
go test -race -run TestName ./internal/voice/     # one test
go test -run XXX -bench . -benchmem ./internal/media/ ./internal/aicall/  # hot paths: 0 allocs/op, CI fails otherwise
make lint                                         # go vet + gofmt + oxlint
sqlc generate                                     # after editing internal/store/sql/*.sql
# migrations: add internal/store/migrations/NNNNN_name.sql (goose); they run at server startup

# dev server; prerequisites, order and ports: docs/dev-stack.md
make dev-up                                       # PostgreSQL 18 + SeaweedFS containers
deploy/dev/restart.sh                             # build web/dist and /tmp/aicc, restart, wait until it serves
/tmp/aicc useradd -username admin -password … -role ADMIN
/tmp/aicc flowadd -file internal/seed/flows/x.json -did 95001   # load + publish; same slug = update + republish
go run ./cmd/aicc-mockbackend                     # business APIs the reference flows call (127.0.0.1:8770); the app uses it when AICC_BOT_BACKEND_BASE points there
# logs: stderr and logs/aicc-<starttime>.log (read the file to analyse a run)

# live provider tests: real money (OPENAI_/ALIYUN_/DOUBAO_/GEMINI_API_KEY)
AICC_LIVE_PROVIDER_TEST=1 go test -count=1 -run Live -v ./internal/provider/...

cd web && npm run dev                             # Vite on 5173
cd web && npm run build                           # web/dist, embedded via go:embed
cd web && npm run test                            # vitest

make stack-up                                     # the whole product in containers, seeded (deploy/README.md)
```

Load harness: `docs/load-tests.md`.

## Database and migrations

PostgreSQL runs in a container (`make dev-up`), never as a host install. Database tests (store, seed, httpapi) skip unless `AICC_TEST_DATABASE_URL` is set; CI sets it.

```sh
AICC_TEST_DATABASE_URL='postgres://aicc:aicc@127.0.0.1:5432/aicc?sslmode=disable' go test -race ./internal/store/
```

**A migration is not reviewed until it has run.** The store tests apply every migration from zero, roll them all back, re-apply them, and migrate databases that already hold rows. A migration that narrows a CHECK must rewrite existing rows before installing the constraint, or PostgreSQL rejects it ("is violated by some row") on every deployment with history. A migration that changes an enum's allowed values gets a fixture test in `migrate_test.go` or a sibling `migrate_*_test.go`: migrate to the previous version, insert old-shape rows, migrate up, assert.

## Config


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rasonyang/ai-native-callcenter](https://github.com/rasonyang/ai-native-callcenter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
