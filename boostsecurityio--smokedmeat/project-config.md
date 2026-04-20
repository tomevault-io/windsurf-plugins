---
trigger: always_on
description: **ALL new source files MUST include the AGPLv3 header at the top:**
---

# AGENTS.md

## CRITICAL: AGPLv3 License Headers Required

**ALL new source files MUST include the AGPLv3 header at the top:**

```go
// Copyright (C) 2026 boostsecurity.io
// SPDX-License-Identifier: AGPL-3.0-or-later
```

This header goes before any package documentation or package declaration. No exceptions.

## CRITICAL: No Em Dashes

**Never use em dashes (U+2014).** Use a regular hyphen-minus with spaces (` - `) instead.

Good: `Kitchen - C2 teamserver with WebSocket API`
Bad: `Kitchen — C2 teamserver with WebSocket API`

## CRITICAL: No Unnecessary Comments

**DO NOT add explanatory comments to code.** This codebase prefers self-documenting code:

- **NO inline comments** explaining what code does
- **NO commented-out code** - delete it, git has history
- **NO TODO comments** unless explicitly requested
- **NO redundant godoc** like `// New creates a new X`

Good: `// Timeout prevents infinite hangs from malicious payloads`
Bad: `// Create a new context with timeout`

## Quick Start (Local Dev)

```bash
# One command  - starts Docker (tunnel + NATS + Kitchen), prewarms cloud shell image, and launches Counter via go run
make dev-quickstart

# Release-backed quickstart uses the pinned version in configs/quickstart-release.mk
make quickstart

# If state gets corrupted:
make dev-quickstart-purge && make dev-quickstart
```

## Make Commands

```bash
# Dev Quick Start (Docker infra + local Counter)
make dev-quickstart            # Start or reuse everything + launch Counter TUI
make dev-quickstart-up         # Start tunnel + NATS + Kitchen (no Counter)
make dev-quickstart-counter    # Launch Counter TUI (after dev-quickstart-up)
make dev-quickstart-down       # Stop containers
make dev-quickstart-purge      # Stop and delete all data

# Release Quick Start (pinned published artifacts)
make quickstart                # Start the pinned release quickstart + Counter TUI
make quickstart-up             # Start pinned release infrastructure only
make quickstart-counter        # Launch pinned release Counter TUI
make quickstart-down           # Stop pinned release containers
make quickstart-purge          # Stop pinned release containers and delete data
make quickstart-version        # Show pinned quickstart release
make quickstart-pin VERSION=v... # Maintainer only - verify and update quickstart pin

# Counter (Remote Kitchen)
make counter               # Run Counter TUI (uses ~/.smokedmeat config)

# Testing
make test                  # Run tests
make lint                  # Run linter

# E2E Testing (Docker + tmux, used by coding agents)
make e2e-exploit           # Full exploit flow test (automated)
make e2e-up                # Start infrastructure
make e2e-down              # Stop containers
make e2e-purge             # Stop and delete all data
make e2e-counter           # Launch Counter in tmux (127x40)
make e2e-capture           # Capture tmux pane output
make e2e-keys KEYS='...'   # Send keystrokes to tmux
make e2e-kitchen-rebuild   # Rebuild Kitchen only (tunnel stays)

# Build
make build-brisket         # Build implant (Linux amd64)
make build-brisket-all     # Build for all platforms
make tag VERSION=v...      # Create, sign, and push a release tag
make pinact                # Pin GitHub Actions (run after workflow changes)
```

## When to Rebuild

| Working on... | Command needed |
|---------------|----------------|
| Counter TUI only | Just restart Counter (no rebuild needed) |
| Kitchen code | `make e2e-kitchen-rebuild` or `make dev-quickstart` |
| Corrupted state | `make dev-quickstart-purge && make dev-quickstart` |

## CRITICAL: Kitchen DB Schema Versioning

Kitchen persistence in `internal/kitchen/db/` must use an explicit schema version stored in DB metadata. Treat the DB schema version as separate from the app release version.

- Start the public `v0.1.x` line at schema `1.0`.
- Patch and minor app releases within the same schema major must be able to open existing quickstart and dev-quickstart Kitchen volumes.
- If a DB file predates schema metadata but already has the known buckets for the current line, treat it as legacy-current and backfill the schema metadata on open.
- On schema major mismatch, fail fast during Kitchen startup with a clear error telling the operator to purge the Kitchen volume with `make quickstart-purge` or `make dev-quickstart-purge`.

### When To Bump Schema Minor

Bump schema minor for additive, backward-compatible on-disk changes that old code can safely ignore or new code can default:

- adding a new bucket that does not change meaning of existing buckets
- adding optional JSON fields to persisted rows or Pantry assets
- adding new Pantry asset properties or relationships that older code can ignore
- adding metadata keys, timestamps, or derived caches that can be rebuilt

### When Not To Bump Schema Version

Do not bump schema version for changes that do not alter the persisted on-disk contract:

- Counter-only rendering, layout, or tree placement changes
- in-memory Pantry logic changes when persisted JSON shape and meaning stay the same
- test-only, docs-only, or logging-only changes
- internal refactors that keep the same buckets, keys, JSON fields, and semantics

### When To Bump Schema Major


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boostsecurityio/smokedmeat](https://github.com/boostsecurityio/smokedmeat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
