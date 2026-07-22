---
trigger: always_on
description: Dakota is a [BuildStream 2](https://buildstream.build/) project producing **Dakota** — Project Bluefin's bootc OCI desktop image built from source. No RPMs. No dnf. No Containerfile package overlays. BST elements only. Historical `bluefin/` paths in this repo are Dakota build paths, not permission to use bluefin's dnf/RPM workflow. Load [`docs/skills/not-bluefin.md`](docs/skills/not-bluefin.md) FIRST if you have any bluefin context.
---

# AGENTS.md

Dakota is a [BuildStream 2](https://buildstream.build/) project producing **Dakota** — Project Bluefin's bootc OCI desktop image built from source. No RPMs. No dnf. No Containerfile package overlays. BST elements only. Historical `bluefin/` paths in this repo are Dakota build paths, not permission to use bluefin's dnf/RPM workflow. Load [`docs/skills/not-bluefin.md`](docs/skills/not-bluefin.md) FIRST if you have any bluefin context.

Load **[docs/SKILL.md](docs/SKILL.md)** for the full reference skill tree. Only load docs relevant to your task.

> **Before using any tool or library: look up its docs via Context7 first. Always.**
> BuildStream, bootc, cosign, skopeo, GitHub Actions — every tool has live, authoritative docs.
> Pattern: `resolve-library-id` → `get-library-docs` → implement → cite the section.
> Guessing, flag-hunting, and trial-and-error are banned. The docs exist. Read them.

## Org pipeline — projectbluefin

### Repo map

```
common ──────────────────────────┐
(shared OCI layer)               │
                                 ▼
bluefin  (testing→main→:stable)       ←── images ──→ testsuite (e2e gate)
bluefin-lts (main→:lts, migrating to testing-first)  ←── images ──→ testsuite (e2e gate)
dakota  (testing→main→:stable)        ←── images ──→ testsuite (e2e gate)
dakota  (next→:next/:btw, rolling nightly, no stable promotion)
                                 │
                                 ▼
                                iso (installation media)
```

Each image repo pulls `ghcr.io/projectbluefin/common:latest` as a base layer.
testsuite gates PR changes; stable promotion for Dakota is daily and automated (SHA freshness + cosign verify + boot-check).

**Dakota image streams:**
- `:testing` — `testing` branch, publishes on every BST-changing push (GHA-only changes filtered)
- `:stable` — `main` (bookmark), promoted daily from `:testing` via `execute-release.yml` — no PR, no human approval
- `:next` / `:btw` — `next` branch, GNOME 51 master, fully automated rolling nightly, **no promotion to stable ever**

**`elements/bluefin/common.bst` strips bluefin-only content from common.** Any file added to `common/system_files/shared/` that does not apply to a fresh dakota install must be explicitly `rm -f`'d in the `install-commands` block of that element. Current stripped files: `rechunker-group-fix` script, service, and preset (chunka migration aid — not needed on fresh dakota).

### 🚫 Absolute prohibition — ublue-os org

**NEVER create issues, pull requests, comments, forks, webhook calls, API writes, automated reports, or any other programmatic action targeting any `ublue-os/*` repository.**

This applies in every situation, without exception:
- Issues, comments, PRs, forks → **BANNED**
- Automated reports (bonedigger output, CI notifications, diagnostic uploads) → **BANNED**
- `workflow_dispatch` or `repository_dispatch` calls to `ublue-os/*` → **BANNED**
- Any `gh` CLI command that writes to `ublue-os/*` → **BANNED**

If a task seems to require touching an upstream `ublue-os` repo → **stop and tell the human to report it manually.** Violating this risks getting the projectbluefin organization banned from GitHub.

---

## The Self-Improvement Loop

> **This is the core operating model. Read it.**

Every agent session produces two outputs:
1. **The work** — the PR, fix, or improvement.
2. **The learning** — what you discovered that a future agent should know.

Output 1 without Output 2 leaves the system no smarter. **The loop only compounds if agents write back.**

```
Agent works on task
  └─ discovers pattern / workaround / convention
       └─ writes it to the relevant skill file in docs/skills/
            └─ commits in the same PR
                 └─ next agent starts smarter
                      └─ loop
```

### Skill-improvement mandate

**Before marking your work complete / before requesting final review:**

- [ ] Did I discover any workaround, non-obvious pattern, or convention?
- [ ] Is there a skill file for the area I worked in?
- [ ] If yes — did I update it?
- [ ] If no — did I create one?
- [ ] Is the skill file committed in this same PR?

### What counts as a learning worth writing back

**Write it:**
- A workaround for an upstream bug (include component + issue link)
- A non-obvious pattern required for correctness
- A convention that isn't obvious from the code
- Something you had to discover by trial and error

**Don't write it:**
- One-off task notes ("use commit message X for this PR")
- Obvious things any developer would know
- Ephemeral state ("currently broken, fix pending")

### What is banned

These patterns actively harm the factory. **Delete them on sight.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [projectbluefin/dakota](https://github.com/projectbluefin/dakota) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
