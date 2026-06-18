---
trigger: always_on
description: **Obsidian S3 Sync + Backup** — An Obsidian community plugin that provides bi-directional vault synchronization and scheduled backups for S3-compatible storage (AWS S3, Cloudflare R2, RustFS) with optional end-to-end encryption.
---

# AGENTS.md

## Project Overview

**Obsidian S3 Sync + Backup** — An Obsidian community plugin that provides bi-directional vault synchronization and scheduled backups for S3-compatible storage (AWS S3, Cloudflare R2, RustFS) with optional end-to-end encryption.

**Plugin ID:** `simple-storage-sync-and-backup`

| Attribute | Value |
|-----------|-------|
| Type | Obsidian Community Plugin |
| Language | TypeScript (strict mode) |
| Runtime | Browser (NO Node.js APIs) |
| Package Manager | npm |
| Bundler | esbuild |
| Output | `main.js`, `manifest.json`, `styles.css` |

## Agent Directives

1. **Strict Linting:** After EVERY code modification, run `npm run lint`. Fix errors immediately before proceeding.
2. **Up-to-Date Knowledge:** If you lack current information on any dependency (AWS SDK, Obsidian API, etc.), use the `context7` tool when it's available. Do not guess.
3. **Document Everything:** Over-documenting is preferred. See the [Documentation](#documentation) section.
4. **Maintain Consistency:** When you change logic or code, update all related mentions (docs, comments, tests, README, CONTRIBUTING, AGENTS).
5. **Scrutinize Existing Code:** Don't assume it's perfect. If you see a bad pattern, fix it.
6. **Fix the Code, Not the Test:** If a valid test fails, the code is broken. Never weaken a test to pass buggy code.
7. **Root Cause Analysis:** Fix the root cause of bugs, don't patch symptoms.
8. **Human contributor guide:** See [CONTRIBUTING.md](CONTRIBUTING.md) for the full human-facing development guide.
9. Refer `OBSIDIAN-PLUGIN-GUIDE.md` for the official Obsidian plugin development guidelines and best practices. You should adhere to these strictly, as they are the basis for plugin approval and user trust.

## Architecture

### Project Structure

```
obsidian-s3-sync-and-backup/
├── src/
│   ├── main.ts                      # Plugin entry point (keep minimal)
│   ├── settings.ts                  # Settings tab UI
│   ├── statusbar.ts                 # Status bar component
│   ├── commands.ts                  # Command palette registration
│   ├── types.ts                     # TypeScript interfaces & constants
│   │
│   ├── sync/                        # Sync engine (v2 — three-way reconciliation)
│   │   ├── SyncEngine.ts            # Thin orchestrator (~150 lines)
│   │   ├── SyncPlanner.ts           # Discovers state, classifies, builds plan
│   │   ├── SyncDecisionTable.ts     # Pure-function L/R decision matrix
│   │   ├── SyncExecutor.ts          # Bounded-concurrency plan executor
│   │   ├── SyncJournal.ts           # IndexedDB per-file baseline persistence
│   │   ├── ChangeTracker.ts         # Local dirty-paths tracker
│   │   ├── SyncScheduler.ts         # Periodic sync scheduling
│   │   ├── SyncPathCodec.ts         # Local ↔ S3 key conversion
│   │   ├── SyncObjectMetadata.ts    # S3 custom metadata encoding
│   │   └── SyncPayloadCodec.ts      # Encryption-aware content encoding
│   │
│   ├── backup/                      # Backup engine
│   │   ├── BackupScheduler.ts       # Backup scheduling with catch-up logic
│   │   ├── SnapshotCreator.ts       # Full vault snapshot creation
│   │   ├── RetentionManager.ts      # Old backup cleanup (by days/copies)
│   │   ├── BackupDownloader.ts      # Backup download as zip
│   │   └── BackupListModal.ts       # Modal UI listing recent backups with download
│   │
│   ├── storage/                     # S3 abstraction layer
│   │   ├── S3Provider.ts            # S3 operations wrapper
│   │   ├── S3Config.ts              # S3 client configuration
│   │   ├── ObsidianHttpHandler.ts   # Custom HTTP handler for Obsidian
│   │   └── ObsidianRequestHandler.ts
│   │
│   ├── crypto/                      # Encryption modules
│   │   ├── KeyDerivation.ts         # Argon2id key derivation (hash-wasm)
│   │   ├── FileEncryptor.ts         # XSalsa20-Poly1305 (tweetnacl)
│   │   ├── VaultMarker.ts           # Encryption marker file (vault.enc)
│   │   └── Hasher.ts                # SHA-256 file hashing (hash-wasm)
│   │
│   └── utils/                       # Shared utilities
│       ├── retry.ts                 # Retry with exponential backoff
│       ├── time.ts                  # Time formatting (relative times)
│       ├── paths.ts                 # Path normalization & glob matching
│       └── vaultFiles.ts            # Vault file read helpers (text/binary)
│
├── tests/                           # Unit & integration tests (Jest)
│   ├── __mocks__/                   # Obsidian API mocks
│   ├── helpers/                     # S3 test utilities
│   ├── backup/                      # Backup module tests
│   ├── crypto/                      # Crypto module tests
│   ├── storage/                     # Storage module tests
│   ├── sync/                        # Sync module tests
│   └── utils/                       # Utility tests
│
├── manifest.json
├── package.json
├── esbuild.config.mjs
└── tsconfig.json
```

### Sync Engine (v2) Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ceilaolabs/obsidian-s3-sync-and-backup](https://github.com/ceilaolabs/obsidian-s3-sync-and-backup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
