---
trigger: always_on
description: `scribe` is a single-binary Go CLI that runs an LLM-written knowledge-base pipeline. It extracts reusable knowledge from git repos, mines coding-agent sessions via ccrider's FTS5 index, absorbs captured URLs, and reindexes the KB with `qmd`. Designed to run on cron against a private KB repo. `scribe init` scaffolds a fresh KB anywhere; the user picks the KB's display name (defaulting to the directory basename).
---

# scribe — agent guide

`scribe` is a single-binary Go CLI that runs an LLM-written knowledge-base pipeline. It extracts reusable knowledge from git repos, mines coding-agent sessions via ccrider's FTS5 index, absorbs captured URLs, and reindexes the KB with `qmd`. Designed to run on cron against a private KB repo. `scribe init` scaffolds a fresh KB anywhere; the user picks the KB's display name (defaulting to the directory basename).

This file is the agent guide for hacking on scribe itself. End-user docs live in `README.md`.

---

## Repo layout

```
cmd/scribe/          Single Go main + every subcommand in one package
  main.go            Kong CLI root
  sync.go            `scribe sync` — discover → extract → absorb → reindex
  triage.go          FTS5 session scoring
  sessions.go        Session log debug/repair
  capture.go         iMessage chat.db reader + 3-tier URL fetcher
  dream.go           Weekly memory consolidation driver
  lint.go            Frontmatter + size + orphan checks
  doctor.go          Read-only health audit
  link.go            Orphan linker (See Also injection)
  cron.go            macOS LaunchAgent install/status/uninstall
  agent_refresh.go   upgrade self-heal: the first scheduled job of a new version refreshes stale LaunchAgents
  hook.go            SessionEnd hook: score + queue to pending-sessions.txt
  init.go            Bootstrap a new KB from embedded templates
  ingest.go          Drain inbox → raw/articles/
  fda.go             macOS Full Disk Access probe + interactive grant
  prompts/           Embedded LLM prompt templates
  templates/         Embedded KB scaffold (scribe.yaml, CLAUDE.md, dirs)

Formula/             Homebrew tap formula (brew install oliver-kriska/scribe/scribe)
scripts/             pre-commit hook shipped into user KBs
install.sh           curl-piped installer
Makefile             build/install/test with -tags sqlite_fts5
Justfile             dev shortcuts
.goreleaser.yml      release pipeline
site/                getscribe.dev marketing page (Cloudflare Workers, static assets only — no JS toolchain in repo)
```

One Go package under `cmd/scribe/`. No internal/ split yet — keep it that way until the package breaks 3000 LOC or a second binary is needed.

---

## Build

```sh
make build        # CGO_ENABLED=1, -tags sqlite_fts5 → ./bin/scribe (repo-local, gitignored)
make install      # build + deploy ./bin/scribe to $HOME/.local/bin — the binary cron runs
make test         # go test ./... -tags sqlite_fts5
make check        # test + vet
```

**Build never deploys.** `make build` writes only to `./bin/scribe`; the live binary at `~/.local/bin/scribe` (executed by cron) changes only on `make install`. On macOS, `make install` automatically Developer-ID-signs with the first available **Developer ID Application** identity; override `CODESIGN_IDENTITY` when more than one is installed. Signed rebuilds from the same team preserve the chat.db Full Disk Access grant. Without a Developer ID identity the install stays unsigned, and replacing it requires another `scribe fda`.

**FTS5 is mandatory.** ccrider's `messages_fts` virtual table uses it, and `scribe triage` runs weighted FTS5 `MATCH` queries against it. `go-sqlite3` ships without FTS5 — the `sqlite_fts5` build tag is what flips it on. Never drop that tag from the Makefile.

**CGO is required** for go-sqlite3. Cross-compilation across OS/arch needs a C toolchain; GoReleaser handles this in the release workflow.

---

## Key external surfaces

| Input                       | Path                                        | Notes                             |
| --------------------------- | ------------------------------------------- | --------------------------------- |
| ccrider sessions DB         | `~/.config/ccrider/sessions.db`             | FTS5, read-only access            |
| Claude Code session folders | `~/.claude/projects/*`                      | keyed by project cwd              |
| Codex CLI rollouts          | `~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl` | first line is `session_meta` with verbatim `cwd` — used by `sync --discover` |
| Claude handshake block      | `~/.claude/CLAUDE.md`                       | scribe-managed block between `<!-- scribe:begin -->`/`end` markers; written by `init` |
| Codex handshake block       | `~/.codex/AGENTS.md`                         | same markers/block as the Claude one; written by `init` so Codex CLI sessions query the KB + write drop files |
| Amp handshake block         | `~/.config/amp/AGENTS.md`                    | same markers/block; `$HOME`-relative on purpose (Amp documents that path, XDG would miss it). Amp *also* reads `~/.config/AGENTS.md` + project-root `AGENTS.md` — scribe manages neither |
| iMessage chat DB            | `~/Library/Messages/chat.db`                | needs Full Disk Access            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oliver-kriska/scribe](https://github.com/oliver-kriska/scribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
