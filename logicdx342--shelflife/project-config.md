---
trigger: always_on
description: - Treat the checked-in codebase as the source of truth for implementation work. `SPEC.md` is the product/technical target; when it disagrees with code, verify the code first and update docs or implementation deliberately.
---

## Workspace rules

- Treat the checked-in codebase as the source of truth for implementation work. `SPEC.md` is the product/technical target; when it disagrees with code, verify the code first and update docs or implementation deliberately.

## Project layout

**File creation policy:** Create new files in the module that owns the behavior. Do not append unrelated code to an existing file just because it already exists. If a feature crosses seams, keep Tauri/window code in command/runtime/dropzone modules, pure engine behavior in `engine/` or `rules/`, persistence in `storage/`, and shared structs in `models/`.

### Backend (Rust) — `src-tauri/src/`

```text
src-tauri/src/
├── main.rs                  # Windows entry point only — do not add logic here
├── lib.rs                   # Tauri builder setup, plugin init, command registration, mod declarations
├── dropzone.rs              # Tauri/window-specific desktop dropzone monitor and positioning
├── tray.rs                  # System tray menu setup and window lifecycle handling
├── commands/                # Tauri IPC command handlers (one file per domain)
│   ├── mod.rs               # Re-exports all command functions
│   ├── config.rs            # config, close behavior, watch pause/resume, reconciliation scan
│   ├── dropzone.rs          # dropzone preview/ingest/rule-group/hide commands
│   ├── external.rs          # external URL opening
│   ├── files.rs             # active files, rule explanations, previews, file location, directory picker
│   ├── rules.rs             # list/save/test/delete automation rules
│   ├── tray.rs              # tray label updates
│   ├── triage.rs            # single/bulk triage, undo, audit listing, notifications
│   └── updates.rs           # app update check/install
├── runtime/                 # Tauri lifecycle orchestration and background workers
│   ├── mod.rs               # AppRuntime, setup(), watcher/dropzone sync, pause/resume
│   ├── mock.rs              # debug mock data generation
│   ├── reconciliation.rs    # Async/manual/periodic reconciliation orchestration and events
│   ├── resource_limits.rs   # CPU usage limits for background tasks
│   └── rule_scheduler.rs    # Async/periodic automatic rule execution scheduling and events
├── engine/                  # File hygiene engine (no Tauri dependency)
│   ├── mod.rs               # Re-exports
│   ├── dropzone.rs          # Dropzone preview planning and shake detection logic
│   ├── executor.rs          # Safe action execution, dropzone ingest/action, undo, rename/collision handling
│   ├── freshness.rs         # freshness_at calculation, decay state transitions, origin evidence
│   ├── paths.rs             # PathScope, config path validation, watch/safe-folder scope checks
│   ├── quiescence.rs        # Transient/system/hidden path checks and file stability checks
│   ├── reconciliation.rs    # Full and incremental watched-path reconciliation
│   ├── rule_execution.rs    # Expired automatic rule execution and failure audit entries
│   ├── rule_projection.rs   # Tracked-file rule projection and automatic-rule candidate computation
│   ├── rule_refresh.rs      # Recompute tracked files after rule changes
│   └── watcher.rs           # notify watcher setup, debounced stable-path emission
├── rules/                   # Rule engine (no Tauri dependency)
│   ├── mod.rs               # Re-exports
│   ├── conditions.rs        # Extension, glob, regex, size, origin matching
│   ├── explanation.rs       # RuleMatchExplanation generation
│   ├── rule_set.rs          # CompiledRuleSet, decide_file, RuleDecision/RuleVerdict
│   └── validation.rs        # Rule validation, rename template validation
├── storage/                 # SQLite/Diesel persistence layer
│   ├── mod.rs               # Database init, DDL bootstrap, config persistence
│   ├── audit.rs             # AuditEntry CRUD and sequence management
│   ├── migrations.rs        # Schema migrations
│   ├── rules.rs             # AutomationRule CRUD
│   ├── schema.rs            # Diesel table! metadata mirroring SCHEMA_SQL
│   ├── test_util.rs         # Rust test fixtures
│   └── tracked.rs           # TrackedFile CRUD and tracked secondary indexes
└── models/                  # Shared data types (serde structs/enums; no business logic)
    ├── mod.rs               # Re-exports all model types
    ├── audit.rs             # AuditEntry, AuditActionKind, UndoStatus, bulk triage models
    ├── config.rs            # AppConfig, WatchTarget, CloseBehavior
    ├── dropzone.rs          # Dropzone preview/action result models
    ├── error.rs             # AppError and Diesel/io conversions
    ├── rule.rs              # AutomationRule, RuleMode, RuleAction, RuleConditions
    ├── runtime.rs           # ReconciliationReport
    └── tracked_file.rs      # TrackedFile, FileDecayState, Expiry
```

**Rules:**

- `main.rs` only calls `shelflife_lib::run()`. Never modify it unless the binary entry point itself changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LogicDX342/ShelfLife](https://github.com/LogicDX342/ShelfLife) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
