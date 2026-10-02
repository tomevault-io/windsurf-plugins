---
trigger: always_on
description: This is a **Tauri desktop application** for managing CPU affinity on Windows.
---

# Repository Guidelines

## Project Structure & Module Organization

This is a **Tauri desktop application** for managing CPU affinity on Windows.

```
cpum/
├── src/                                # Vue 3 + TypeScript + Vuetify frontend
│   ├── api.ts                          # Tauri IPC command wrappers
│   ├── App.vue                         # Main application shell (orchestrator)
│   ├── i18n.ts                         # Bilingual (zh-CN / en-US) message dictionary & t() helper
│   ├── types.ts                        # Shared TypeScript types (mirrors Rust models)
│   ├── constants.ts                    # Shared constants and type aliases
│   ├── vuetify.ts                      # Vuetify plugin setup (theme + icons)
│   ├── components/
│   │   ├── AffinityEditor.vue          # Per-process affinity editor dialog
│   │   ├── AffinityRuleManager.vue     # Persistent rule manager dialog
│   │   └── ProBalancePanel.vue         # ProBalance dynamic-optimization settings panel
│   └── composables/
│       ├── useTopology.ts              # CPU topology cache + refresh lifecycle
│       ├── useProcessManager.ts        # Process list bootstrap, refresh, search
│       ├── useMetricsStream.ts         # Real-time metrics event handling
│       ├── useProcessTree.ts           # Process tree/flattening logic
│       └── useTheme.ts                 # Dark/light theme switcher (localStorage: cpum-theme)
├── src-tauri/                          # Rust backend (cargo workspace root)
│   ├── src/
│   │   ├── lib.rs                      # Tauri command registrations & entry point
│   │   ├── models.rs                   # Shared data models (serde)
│   │   ├── topology.rs                 # CPU topology detection (CCD, cores, SMT)
│   │   └── process/                    # Process subsystem (GUI-side)
│   │       ├── sampling.rs             # Sampling primitives + differential cache
│   │       ├── enumerate.rs            # ToolHelp fast scan + parallel handle walk
│   │       ├── metrics.rs              # Per-second metrics stream (4-wave events)
│   │       ├── net_probe.rs            # Network IO diagnostic probe
│   │       └── system_metrics.rs       # Logical-processor usage sampling
│   └── crates/
│       ├── cpum-core/                  # Shared core library (no tauri dependency)
│       │   └── src/
│       │       ├── rule.rs             # Rule data model (schema v2) & validation
│       │       ├── matcher.rs           # Name/path matching (exact / wildcard / path)
│       │       ├── store.rs             # Rule file persistence (v1→v2 auto-migration)
│       │       ├── procwin.rs           # Win32 writes: affinity / CPU Sets / 3 priority classes
│       │       ├── engine.rs            # Rule application engine (enumerate→match→apply)
│       │       ├── monitor.rs           # CPU differential sampling + foreground detection
│       │       ├── ipc.rs               # Named-pipe privileged bridge (GUI↔service)
│       │       └── probalance/          # Dynamic optimization engine (config, engine,
│       │                                #   journal, runtime, tests — split by responsibility)
│       └── cpum-service/                # Windows service binary (LocalSystem daemon)
│           └── src/main.rs              # Rule daemon + ProBalance tick + one-shot helper
├── scripts/                            # Dependency-free Node scripts run by CI / by hand
│   ├── lint-i18n.mjs                   # i18n guard: locale parity, t() keys, stray CJK
│   ├── snapshot-releases.mjs           # Refreshes the download page's fallback data
│   └── lib/source.mjs                  # Comment stripper + tokenizer shared by the two
├── docs/
│   ├── ROADMAP.md                      # What is next, and the non-goals with reasons
│   ├── UPDATER.md                      # Updater keys, artifacts, verification
│   ├── releases/                       # The GitHub Pages site root (download page)
│   │   ├── index.html                  # Reads the releases API, never builds filenames
│   │   ├── assets/releases.js          # Fetch / cache / classify — the only copy
│   │   └── data/releases.json          # Committed snapshot for when the API is down
│   └── signpath-foundation-application.md
├── screenshots/                        # README images; the spec lives in that directory
├── .github/                            # CI / CodeQL / Pages workflows, issue + PR templates, dependabot
├── .gitleaks.toml                      # Secret-scan allowlist (public keys that look like keys)
├── CHANGELOG.md                        # Keep a Changelog; add to [Unreleased] with every PR
├── CONTRIBUTING.md                     # Build setup, house rules, good first issues
├── SECURITY.md                         # Privilege boundary and vulnerability reporting
├── CODE_OF_CONDUCT.md
└── package.json
```

## Build, Test, and Development Commands

| Command | Description |
|---------|-------------|
| `npm ci` | Install frontend dependencies. **npm is the only supported package manager** — there is no `yarn.lock` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-nexa/cpum](https://github.com/open-nexa/cpum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
