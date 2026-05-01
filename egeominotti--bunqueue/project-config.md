---
trigger: always_on
description: "When I report a bug, don't start by trying to fix it. Instead, start by writing a test that reproduces the bug. Then, have subagents try to fix the bug and prove it with a passing test."
---

# bunqueue

"When I report a bug, don't start by trying to fix it. Instead, start by writing a test that reproduces the bug. Then, have subagents try to fix the bug and prove it with a passing test."

**MANDATORY: After ANY code modification, ALWAYS run ALL THREE test suites before committing:**

```bash
bun test                                # Unit tests (~5000 tests)
bun scripts/tcp/run-all-tests.ts        # TCP integration tests (~50 suites)
bun scripts/embedded/run-all-tests.ts   # Embedded integration tests (~35 suites)
```

Never commit without all three passing. No exceptions.

**MANDATORY: After every commit, ALWAYS:**

1. Bump version in `package.json`
2. Update changelog in `docs/src/content/docs/changelog.md`
3. `git push origin main`
4. `bun publish` to publish new version to npm

No exceptions. Every commit = new version + changelog + npm publish.

High-performance job queue server for Bun. SQLite persistence, cron jobs, priorities, DLQ, S3 backups.

## Architecture Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT                                          │
│  Queue.add() ─────┐                              ┌───── Worker.process()    │
│  Queue.addBulk() ─┤                              │                          │
│                   ▼                              ▼                          │
│            ┌──────────┐                   ┌──────────┐                      │
│            │ TcpPool  │◄─── msgpack ────► │ TcpPool  │                      │
│            └────┬─────┘                   └────┬─────┘                      │
└─────────────────┼──────────────────────────────┼────────────────────────────┘
                  │ TCP :6789                    │
┌─────────────────┼──────────────────────────────┼────────────────────────────┐
│                 ▼           SERVER             ▼                            │
│          ┌───────────┐                  ┌───────────┐                       │
│          │ TcpServer │                  │ TcpServer │                       │
│          └─────┬─────┘                  └─────┬─────┘                       │
│                │                              │                             │
│                ▼                              ▼                             │
│   ┌────────────────────────────────────────────────────────────┐           │
│   │                    QueueManager                             │           │
│   │  ┌─────────────────────────────────────────────────────┐   │           │
│   │  │              N Shards (auto-detected)                │   │           │
│   │  │  ┌─────────┬─────────┬─────────┬─────────┐          │   │           │
│   │  │  │ Shard 0 │ Shard 1 │   ...   │ Shard N │          │   │           │
│   │  │  │┌───────┐│┌───────┐│         │┌───────┐│          │   │           │
│   │  │  ││PQueue ││PQueue ││         ││PQueue ││          │   │           │
│   │  │  │└───────┘│└───────┘│         │└───────┘│          │   │           │
│   │  │  └─────────┴─────────┴─────────┴─────────┘          │   │           │
│   │  └─────────────────────────────────────────────────────┘   │           │
│   │                           │                                 │           │
│   │  ┌────────────────────────┼────────────────────────────┐   │           │
│   │  │  jobIndex (Map)        │   completedJobs (Set)      │   │           │
│   │  │  customIdMap (LRU)     │   jobResults (LRU)         │   │           │
│   │  └────────────────────────┼────────────────────────────┘   │           │
│   └───────────────────────────┼─────────────────────────────────┘           │
│                               │                                             │
│   ┌───────────────────────────┼─────────────────────────────────┐           │
│   │                           ▼                                 │           │
│   │  ┌─────────────┐    ┌──────────┐    ┌─────────────┐        │           │
│   │  │ WriteBuffer │───►│ SQLite   │◄───│ ReadThrough │        │           │
│   │  │ (10ms batch)│    │ WAL Mode │    │   Cache     │        │           │
│   │  └─────────────┘    └──────────┘    └─────────────┘        │           │
│   │                      Persistence                            │           │
│   └─────────────────────────────────────────────────────────────┘           │
│                                                                             │
│   ┌───────────────────────────────────────────────────────────┐             │
│   │  Background Tasks                                          │             │
│   │  • Scheduler (cron, delayed jobs)    • Stall detector     │             │
│   │  • DLQ maintenance (retry, expire)   • Lock expiration    │             │
│   │  • Cleanup (memory bounds)           • S3 backup          │             │
│   └───────────────────────────────────────────────────────────┘             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Request Flow:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [egeominotti/bunqueue](https://github.com/egeominotti/bunqueue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
