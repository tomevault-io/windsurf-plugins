---
trigger: always_on
description: This is the canonical repository harness. Read [CONTRIBUTING.md](CONTRIBUTING.md) before
---

# Agent instructions

This is the canonical repository harness. Read [CONTRIBUTING.md](CONTRIBUTING.md) before
planning, editing, testing or reviewing. Its PR checklist is mandatory. The user removed
the alternate agent instruction files; do not recreate copies or links.

## Workflow

- Use descriptive `feat/` or `fix/` branches, conventional commits and PRs against
  `ohne-b/twitch-drops-miner:main`. Keep branches, commits and documentation free of assistant branding.
- Preserve existing user changes, data, credentials, logs and backups. Ask before significant
  refactoring unless the current task already authorizes it. A full rewrite authorization
  covers its necessary cleanup. Never change a running deployment without authorization.
- Use one implementation agent when requested. Required independent adversarial review uses
  a separate read-only reviewer; the author cannot approve their own work.
- Integrate current main before final validation/review and again before merge if it advances.
  Keep incomplete PRs in draft. Record tested/reviewed revisions and actual results; never
  describe unrun checks, live progress or review as successful.
- Add backend unit/regression tests; cover frontend changes where practical. Always update
  README and this harness when behavior/architecture changes. Update English messages for UI
  or console changes. There are no other locales or language settings.
- Keep user guidance in README and contributor policy in CONTRIBUTING. Do not create
  `docs/`, `plans/` or repository planning documents.
- GitHub release publication uses the repository owner's scoped `prod` `RELEASE_TOKEN`;
  verify its identity before publishing images. Registry publication retains the workflow's
  built-in token. Keep tokens out of source, logs and chat; never reuse a local CLI credential
  as an Actions secret.

## Architecture

The product is Twitch Drops Miner (dashboard: Drops Miner), repository and
Cargo package/binary are twitch-drops-miner. Preserve the existing Compose service/container name,
data/log directories, TDM log prefix, auth cookie and CSRF header for upgrade compatibility.

One Rust Cargo package owns the backend. Use concrete structs with methods and composition
for domain/services (the repository's OOP requirement), typed enums and DRY shared policies.
Do not add forwarding hierarchies or speculative traits with one implementation.

| Module | Responsibility |
| --- | --- |
| `main.rs` | CLI/env, credential-safe logging, owned shutdown, healthcheck |
| `config.rs`, `dto.rs` | Typed settings, compatible migration, API snapshots |
| `domain.rs`, `policy.rs` | Campaign/drop/channel eligibility and dependency-aware ignores |
| `store.rs` | Exclusive data directory, atomic settings/history/archive/claim journal |
| `auth.rs`, `origin.rs` | Dashboard passwords/sessions, origin and cookie policy |
| `twitch/` | OAuth, bounded HTTP/GQL, inventory/catalog, beacon watch, PubSub shards |
| `app/` | Application commands/settings, typed Activity, revisioned snapshot publication and patches |
| `miner/` | One session supervisor and owned jobs; session, watch, inventory and claim responsibilities |
| `store/records.rs` | Frozen version-one archive/journal records, independent of runtime presentation fields |
| `web/` | Axum HTTP, Socketioxide, protected snapshots and embedded frontend |
| `fixture.rs`, `bin/dashboard-fixture.rs` | Feature-gated offline browser fixture |
| `frontend/`, `lang/English.json` | React/TypeScript/Tailwind and one message catalog |

Build frontend assets before backend/static tests. `web/` is ignored output. Release binaries
embed it and run without a build tool/runtime companion. Production builds never enable
`dashboard-fixture`; fixture routes must return 404 in production.

## Mining contracts

- Discovery does not select games. `games_to_watch` is the ordered Unicode-casefolded
  automatic allowlist. Opt-in auto_mine_badges/auto_mine_emotes default false and add matching
  watch rewards from other games, with dependency closure and existing benefit/ignore rules.
  Selected games rank first; other campaigns rank by soonest end. Use the shared mining
  policy for discovery, selection, estimates, progress eligibility and Up next. Display
  filters remain separate. Empty with both options off means no automatic watch events. Mine selects a game across
  eligible campaigns. An explicitly chosen manual channel overrides automatic selection
  without changing saved games, filters or priorities.
- `mining_priority_mode` defaults to `manual` and preserves the saved game order.
  `short_events` promotes active selected-game reward windows of at most 24 hours total;
  `ending_soonest` ranks active selected-game reward deadlines regardless of duration.
  Use campaign/drop date intersections, earliest deadline then saved game order, and
  preserve equal-priority watching. Required earnable prerequisites inherit target deadlines
  (bounded by their own expiry); impossible, ignored or filtered target branches cannot
  boost unrelated rewards. Confirmed completion ends a boost without inventing a claim;
  estimates do not. These modes do not predict completion feasibility, reorder saved games,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ohne-b/twitch-drops-miner](https://github.com/ohne-b/twitch-drops-miner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
