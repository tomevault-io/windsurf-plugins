---
trigger: always_on
description: handles all Obsidian API interactions and UI rendering. A dedicated Web Worker
---

# ZotFlow — Agent Guide

> This document is the single source of truth for any AI agent working on this
> codebase. Read it fully before making changes.

---

## 1. Project Identity

| Field                 | Value                                             |
| --------------------- | ------------------------------------------------- |
| **Name**              | ZotFlow (`obsidian-zotflow`)                      |
| **Type**              | Obsidian Community Plugin                         |
| **Language**          | TypeScript (strict mode)                          |
| **Bundler**           | esbuild (custom config in `esbuild.config.mjs`)   |
| **Package manager**   | npm                                               |
| **Entry point**       | `src/main.ts` → bundled to `main.js`              |
| **Release artifacts** | `main.js`, `manifest.json`, `styles.css`          |
| **License**           | AGPL-3.0-only                                     |
| **Mobile**            | `isDesktopOnly: false` — code must be mobile-safe |

---

## 2. Architecture Overview

ZotFlow uses a **Main Thread + Web Worker** split architecture. The main thread
handles all Obsidian API interactions and UI rendering. A dedicated Web Worker
handles Zotero API communication, sync logic, database access, and PDF
processing.

```
┌─────────────── Main Thread (Obsidian) ──────────────────────┐
│                                                              │
│  main.ts (Plugin lifecycle, commands, view registration)     │
│       │                                                      │
│       ├── services/  (ServiceLocator singleton)              │
│       │     ├── IndexService   (vault file → zotero-key)     │
│       │     ├── LogService     (in-memory log buffer)        │
│       │     ├── NotificationService  (styled Notice)         │
│       │     └── TaskMonitor    (pub/sub task updates)        │
│       │                                                      │
│       ├── ui/                                                │
│       │     ├── reader/   (ZoteroReaderView, IframeReaderBridge, │
│       │     │              LocalReaderView, LocalDataManager)    │
│       │     ├── tree-view/ (React: ZotFlowTree, Node)        │
│       │     ├── activity-center/ (React: ActivityCenterModal) │
│       │     ├── modals/suggest.ts (BaseItemSearchModal + ZoteroSearchModal) │
│       │     ├── zotflow-lock-extension.ts (CM6 readonly)      │
│       │     └── zotflow-comment-extension.ts (CM6 deco)       │
│       │                                                      │
│       └── settings/ (tab-based settings UI)                  │
│                                                              │
│  bridge/index.ts  ←─ WorkerBridge singleton (Comlink)        │
│  bridge/parent-host.ts ←─ ParentHost (exposed to Worker)     │
│                                                              │
└──────────────── Comlink (postMessage) ───────────────────────┘
                          │
┌─────────────── Web Worker ──────────────────────────────────┐
│                                                              │
│  worker/worker.ts  (exposes WorkerAPI via Comlink)           │
│       │                                                      │
│       ├── services/                                          │
│       │     ├── zotero.ts       (Zotero Web API wrapper)     │
│       │     ├── sync.ts         (bidirectional sync engine)  │
│       │     ├── attachment.ts   (download + LRU cache)       │
│       │     ├── webdav.ts       (WebDAV file download)       │
│       │     ├── library-note.ts (library source note CRUD)   │
│       │     ├── local-note.ts   (local source note CRUD)     │
│       │     ├── other-template.ts (LiquidJS path + citation templates) │
│       │     ├── library-template.ts (LiquidJS library templates) │
│       │     ├── local-template.ts (LiquidJS local templates) │
│       │     ├── tree-view.ts    (tree topology builder)      │
│       │     ├── pdf-processor.ts (nested PDF.js Worker)      │
│       │     ├── annotation.ts   (reader annotation CRUD)     │
│       │     ├── key.ts          (API key/library metadata)   │
│       │     └── db-helper.ts        (general-purpose DB queries) │
│       │                                                      │
│       └── tasks/                                             │
│             ├── base.ts         (BaseTask abstract class)    │
│             ├── manager.ts      (TaskManager + AbortController) │
│             └── impl/           (SyncTask, BatchNoteTask, …) │
│                                                              │
│  db/  (Dexie.js — IndexedDB, WORKER-ONLY)                    │
│       ├── db.ts          (schema: keys, groups, items,       │
│       │                   collections, libraries, files)     │
│       ├── normalize.ts   (API response → IDB shape)          │
│       └── annotation.ts  (IDB ↔ AnnotationJSON conversion)   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 2.1 Communication patterns

| Path                 | Mechanism                             | Module                                  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duanxianpi/obsidian-zotflow](https://github.com/duanxianpi/obsidian-zotflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
