---
trigger: always_on
description: Luke is a macOS-first Electron sidecar that observes coding-agent sessions while
---

# Agent guide

Luke is a macOS-first Electron sidecar that observes coding-agent sessions while
preserving existing provider workflows. Product naming belongs at the app and
packaging boundary; keep reusable implementation types brand-neutral.
Deployable products belong in `apps/`, and reusable packages belong in
`packages/`. Keep Electron main/preload code in `apps/desktop/` thin, keep the
renderer sandboxed, and put platform-independent behavior in
`packages/sidecar-core/`.

Canonical commands:

- `./scripts/bootstrap.sh` — install pinned workspace dependencies
- `./scripts/check.sh` — run portable repository, type, test, and build checks
- `./scripts/test-macos.sh` — package and validate the macOS app
- `./scripts/verify.sh` — complete macOS validation plus visual evidence
- `pnpm release:macos` — create a local signed, notarized, and verified DMG
- `./scripts/run.sh` — launch the app against live sessions, replacing any
  running instance (`--fixture smoke` for fixture data, `--keep-running` to keep
  the running instance)
- `./scripts/evidence.sh` — write the fixture PNG under `artifacts/`
- `pnpm evidence:record` — record the fixture transition on a physical Mac
- `pnpm lint:fix` — apply repository formatting and safe lint fixes

Trust constraints:

- Never write provider transcripts or session-state files. Reading them is what
  Luke is for; writing to them is never.
- Never inject terminal input, simulate keystrokes, or request Accessibility.
- Product behavior must not require provider MCP, plugins, hooks, wrappers,
  credentials, or live sessions. A provider whose sessions exist only in a cloud
  service may read a user-supplied API key, but it must observe nothing until
  the user supplies one and must leave every other provider working without it.
- One registration is the exception the previous rule's word "require" leaves
  room for, and it is bounded on every side: Luke may merge an observation
  hook into a provider's own user-level hook configuration — today Claude
  Code's `settings.json` and Codex's `hooks.json`, and nothing else of either
  provider's — so local rows can tell a turn that just ended from a session
  walked away from, and can see a tool call holding for permission at all. The
  hook itself writes one fixed status token into a spool under Luke's own
  application data, named by the session's id; the envelope the provider hands
  it — piped in, or passed as an argument — is read only for that id and never
  reaches disk. The merge preserves the user's own entries and settings as
  parsed, recognizes its own entries by the script's name, refuses to rewrite
  a file it cannot parse, converges at launch rather than accumulating, and
  skips a machine with no provider home to join. The registration is part of
  observing at all, like reading the transcripts, so it converges at every
  launch rather than answering to a preference; an entry outliving Luke is a
  guarded no-op, and everything the hook sharpens still observes from the
  transcripts alone wherever the hook is absent — including behind Codex's own
  review gate, which shows a new entry to the user and runs nothing until they
  trust it. Widening it to another provider or another lifecycle event is a
  product decision, not an implementation detail.
- The one thing Luke may change about a session is what the user just asked to
  send it: a message typed on its row, a control its provider advertised for
  it, or the same two acts asked of Luke — out loud, or typed into his own
  composer — in a conversation the user is holding — each through the
  provider's own documented endpoint under the same user-supplied credential,
  and each validated against the observed roster before an adapter sees it.
  Observation passes stay read-only by construction; where a provider's
  documented read answers only a POSTed query — Conductor's transcripts view,
  like Linear's GraphQL — observation sends a read document fixed by the
  build, and nothing enters that document's text but identifiers the same
  pass reported, each validated against the shape its provider documents.
  Nothing that decides on the user's behalf may reach a write path: the
  attention evaluator above all, and every turn Luke opens himself — a
  proactive readout, the reply that voices a tool's outcome — which carries no
  tools, at the API and again at a runtime gate, so a session summary or a tool
  output that reads like an instruction can never become an act. A tool call
  in that conversation runs only in a turn the developer opened themselves, by
  speaking or by typing; a write is the direct product of a turn the developer
  opened, never of anything Luke read or was told. The one act not aimed at an
  existing session keeps the same shape: a new workspace, asked of Luke in
  conversation, lands only in a project its provider reported on the latest
  observation pass and documents a creation endpoint for — the ask names a
  reported project, never a repository URL or path of its own, and a provider
  that documents no such endpoint offers nowhere to create. The ask may carry
  the new agent's opening task — the developer's own words, bounded and
  delivered like a message to an existing session, through the provider's

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ReviewStage/luke](https://github.com/ReviewStage/luke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-18 -->
