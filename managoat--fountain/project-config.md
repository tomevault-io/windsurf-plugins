---
trigger: always_on
description: This file is read by Claude Code (and other AI coding tools) at session start. Keep it accurate — stale guidance misleads every downstream dispatch.
---

# CLAUDE.md — Contributor Guide

This file is read by Claude Code (and other AI coding tools) at session start. Keep it accurate — stale guidance misleads every downstream dispatch.

## Product and goal

What Fountain is, in one paragraph, is the top of [README.md](README.md).
The standing goal since the May-2026 launch is **100 weekly active users by
month 6** (November 2026), and the one gate that hangs on it: org/team
features stay out of scope until that traction shows. Onboarding is the open
product decision on the way there (#1039). These two lines used to live in
`OPERATING_MODEL.md` and `ROADMAP.md`, the bus files for the captain-picard
orchestrator that built the first cut; that fleet stopped running against this
repo after launch and both files were deleted, so this is now the only place
the goal is written down.

## Working with Review Loop

When working on a Review Loop PR, read the
[repository skill](.agents/skills/review-loop/SKILL.md) for bot commits, CI repairs,
human decisions and bounded retries. Report outcomes briefly. The approved-base
`.github/review-loop.yml` and its referenced files remain the policy authority.

The skill is copied unchanged from
[Review Loop Action `9278790`](https://github.com/managoat/review-loop-action/blob/92787907f1b236d471555e3d231443582e46ce12/skills/review-loop/SKILL.md).
Review upstream changes before replacing it; its MIT license is included.

## Quick start

```bash
mise install                        # pin Erlang/OTP 28 + Elixir 1.19.2
mix deps.get
mix setup                           # dev DB: create + migrate
MIX_ENV=test mix ecto.create && MIX_ENV=test mix ecto.migrate
mix test                            # full suite (~1,850 tests)
mix precommit                       # the core CI checks, locally
```

See [SETUP.md](SETUP.md) for full workstation bootstrap (Postgres, OAuth keys, etc.).

## Repo layout

```
fountain/                  umbrella root
  apps/
    fountain/              core business logic (Elixir OTP app)
      lib/fountain/        contexts: Accounts, Agents, Environments, Vaults,
      |                              Conversations, Crypto, Audit
      lib/fountain_web/    Phoenix: controllers, LiveView, plugs, router
      test/fountain/       context unit tests (async: true, DataCase)
      test/fountain_web/   controller/LiveView integration tests
      test/support/        DataCase, ConnCase, factory.ex
    fountain_buzz/         the two first-party extensions (ADR 0043, tracker
    fountain_support/      #1503 and #1528). Each is an AGPL OTP app that
                           depends on :fountain, is named in
                           `config :fountain, :extensions` and is reached only
                           through the `Fountain.Extension` callbacks. Buzz owns
                           FountainBuzz.*, buzz_identities and its migrations,
                           /api/buzz + /api/mcp/buzz, and the harness tree;
                           Support owns FountainSupport.*, support_reports,
                           /api/support and the report forwarder.
                           `apps/fountain` depends on neither in any direction
                           and names no module of either
                           (extension_guard_test.exs); the release's
                           `applications:` decides inclusion. Their suites run
                           from their own directories, so
                           `scripts/test-libraries.sh` runs them in CI. A new
                           extension needs a row in extension_guard_test.exs
                           and a boundary test of its own; adding a tenth
                           `Fountain.Extension` callback needs an ADR.
                           Buzz also owns its supply chain (#1509):
                           buzz-acp.version, buzz-acp.source and the
                           buzz-acp-publish workflow's inputs live in its app,
                           and FountainBuzz.Assets both finds the binaries and
                           refuses one whose version does not match the pin.
                           `BUNDLE_EXTENSIONS=false` builds the core distribution —
                           no Buzz application in the release (mix.exs) and no
                           binaries in the image (Dockerfile), one switch for
                           both.
                           Buzz also owns a Go module, apps/fountain_buzz/cli
                           (#1508): the buzz-backend-fountain remote-agents
                           provider Buzz Desktop discovers by name on PATH,
                           Apache-2.0 like cli/ and released for the same four
                           platforms as the fountain binary. A module of its
                           own so Go's internal/ rule stops it reaching
                           cli/internal/...; it uses the public cli/api +
                           cli/credentials instead. `fountain buzz agents ...`
                           deliberately STAYS in cli/ (ADR 0043 decision 6) —
                           removing a subcommand from a released binary is a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [managoat/fountain](https://github.com/managoat/fountain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
