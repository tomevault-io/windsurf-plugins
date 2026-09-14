---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm test                              # node:test over test/**/*.test.ts via tsx
node --import tsx --test test/race-engine.test.ts   # one test file
node --import tsx --test --test-name-pattern "idempotent" test/*.test.ts
npm run build                         # tsc -p tsconfig.json (type-check + emit to dist/)
npm run check                         # test + build; run this before committing
npm run dev                           # arena API only, 127.0.0.1:3001
npm run dev:all                       # arena API (3001) + deterministic course (4000)
npm run smoke:steel                   # live Steel check: 4 real sessions; needs real keys
```

`npm test` and `npm run build` use fakes and never touch Steel or OpenRouter. Only
`smoke:steel` and a live `POST /races` require credentials. Copy `.env.example` to `.env`.

## Architecture

A backend for a four-racer browser-agent race. Four LLM agents drive four Steel cloud
browser sessions through the same course while a master LLM sabotages them and spectators
trade a virtual prediction market on the winner. The spectator web app is in `web/`
(Vite + React + TypeScript; see `web/README.md`).

Layering is strict and dependency-inverted: `domain` knows nothing about I/O, `application`
depends only on interfaces declared in `src/application/contracts.ts`, and `agents`,
`infra`, `course`, and `persistence` supply the implementations. Tests exercise the
application layer with hand-written fakes rather than mocks; keep new I/O behind a contract
interface so this stays possible.

```
api/server.ts (Fastify)     → RaceRegistry → RaceCoordinator (one per race)
                                                 ├─ RaceEngine        pure state machine, emits RaceEvent[]
                                                 ├─ VirtualPredictionMarket
                                                 ├─ RacerSessionManager   → SteelSessionManager (+ SteelKeyPool)
                                                 ├─ CompetitorAgentRunner → PlaywrightCompetitorRunner (+ OpenRouter model per racer)
                                                 ├─ CourseVerifier        → DeterministicCourseVerifier → HttpCourseStateGateway
                                                 ├─ ObstacleProvider      → MasterObstacleProvider → CdpObstacleProvider
                                                 └─ RaceEventStore        → JsonlRaceEventStore
```

`createProductionRaceCoordinator` (`src/application/production-race-factory.ts`) is the only
place that reads env vars and wires the real implementations; `buildApi` takes the factory
as a parameter so tests inject their own.

### Race lifecycle

`prepareAndStart` creates all four Steel sessions, prepares all four agents, arms the
sabotage plan, passes the readiness barrier, then calls `RaceEngine.start` so all racers
begin at the same timestamp. `prepare` runs everything before the start on its own (once):
with `startHoldMs` (`FIGHT_INTRO_HOLD_MS`, 10 s in live mode, 0 simulated) the registry
prepares a fight created to start now, publishes `startsAt` as ready + hold, and starts it
then on a timer (`startTimer`; `tickAll` is the fallback). Its market stays `pending` (no
trading) until that start, so the intro video (played over the lobby's featured card) ends
exactly as the agents and the market start; scheduled fights keep pre-fight trading. Agent loops then
run detached; a rejected loop marks that racer `failed` rather than failing the race. An
API-level ticker calls `tick` every second.

The ticker calls `RaceEngine.tick`, but elapsed time no longer freezes hazards or ends
a race. The first racer whose finish passes verification wins; the coordinator then
freezes and resolves the market, stops the other runners, and releases every Steel
session. A race is explicitly aborted if startup fails or all runners fail.

Every method takes an explicit `now` parameter defaulting to `Date.now()`. Tests pass fixed
timestamps; preserve this when adding time-dependent logic.

### Verification and idempotency

Racer self-reports are never trusted. `recordCheckpoint`/`recordFinish` call the
`CourseVerifier` first and throw if the course's own state disagrees, and the verifier also
checks that raceId, racerId, courseId, seed, and Steel session id all match the run. The
engine separately enforces idempotency: a repeated `racerId:checkpoint` claim is a no-op,
checkpoints must advance by exactly one, duplicate observations are idempotent, and sabotage
fires at most once per racer. Transient course-state transport failures are retried by the
deterministic verifier; hard authorization or run-proof failures are not hidden.

The master completion judge (`completionJudge`, the master model) is consulted only for a
run the verifier says it does not cover (`CourseVerifier.coversRun` returns false). A
verifier without that method covers every run, and `DeterministicCourseVerifier` covers every
run, so no course-server race (arena-shop or the test course) ever reaches the judge: its
workers get no judge and no page review, and a checkpoint or finish reported with source

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tkachyn/polybot](https://github.com/tkachyn/polybot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
