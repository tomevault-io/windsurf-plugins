---
trigger: always_on
description: > Persistent memory and architectural spec for JarvisOS.
---

# AGENTS.md — JarvisOS Project Spec
> Persistent memory and architectural spec for JarvisOS.
> Read at the start of every session. Update after every sprint.
> Last updated: April 2026

---

## Project Vision

JarvisOS is a privacy-first Android OS built on LineageOS that runs LLM inference,
RAG, and tool-calling as privileged Android system services — baked into the OS,
not running as apps. No cloud dependencies. All inference on-device via Cactus.

---

## Team

- **Kevin** — Lead, Android system layer, RAG architecture, project vision
- **Sam** — NLP engineer, Cactus integration
- **Desmond** — In-house app development (integrates with Tool Registry)
- **Chindu** — Contributor, PR workflow
- **Gerald** — Security and configuration

---

## Repository Structure

| Repo | GitHub | Path (local) | Branch |
|------|--------|--------------|--------|
| Android framework fork | ocansey11/android_frameworks_base | `frameworks/base/` | `lineage-21.0` |
| JarvisOS vendor overlay | ocansey11/vendor_jarvisos | `vendor/jarvisos/` | `main` |
| Cactus inference engine | ocansey11/cactus | `vendor/cactus/` | `main` |

**Local manifest:** `.repo/local_manifests/jarvos.xml`
- Removes LineageOS default `frameworks/base`, replaces with `ocansey11/android_frameworks_base`
- Adds `ocansey11/vendor_jarvisos` → `vendor/jarvisos`
- Adds `ocansey11/cactus` → `vendor/cactus` (`sync-s=false`)

**Build environment:**
- WSL2 Ubuntu 24.04 on Windows
- lineage-22.2 re-sync pending — must do at uni on fast connection
- Current base: lineage-21.0. All JarvisOS code is pure Java — no device-specific deps

---

## Architecture Overview

```
User / App
    |
    | AIDL (Binder IPC)
    v
JarvisService.java  (com.android.server.jarvis)
    |
    +-- ToolDispatcher          tool path (runs first on every query)
    |       |
    |       +-- ToolScannerService   discovers tools from installed APKs
    |       +-- AppRecord / ToolRecord   ObjectBox entities
    |       +-- CactusWrapper.embed + indexQuery (tools index)
    |       +-- sendBroadcast → app BroadcastReceiver → ResultReceiver
    |
    +-- JarvisFileObserver      watches Documents/ Downloads/ Pictures/
    |       |
    |       v
    |   IndexQueue              BlockingQueue (cap 500)
    |       |
    |       v
    |   JarvisIndexWorker          WorkManager (15min, charging only)
    |       |
    |       +-- TextExtractor        file → raw text
    |       +-- ChunkingStrategy     text → chunks
    |       +-- CactusWrapper.embed  chunk → float[]
    |       +-- ObjectBox            persist SourceFile, DocumentChunk
    |
    +-- processQuery()
             |
             v
        MetadataSearch (Stage 1 — ObjectBox keyword/score)
             |
             v
        CactusWrapper.embed + indexQuery (Stage 2 — semantic)
             |
             v
        CactusWrapper.complete → response string
```

---

## Tech Stack

- **Base OS:** LineageOS 21.0 (Android 14) — pending upgrade to 22.2
- **Inference engine:** Cactus (forked) — `vendor/cactus/`
- **Vector DB:** ObjectBox 4.0.3 with HNSW
- **Embedding / chat model:** Qwen / nomic-embed-text
- **Tool selection model:** FunctionGemma (270M, zero-shot only)
- **Build system:** Soong (Android.bp)
- **Languages:** Java (services), C++ (JNI/Cactus)

---

## File Map

Server service code in `frameworks/base/services/core/java/com/android/server/jarvis/`:
Public API layer in `frameworks/base/core/java/android/jarvis/`:

```
jarvis/                              ← renamed from rag/ this session
├── JarvisService.java              ✅ System service entry point
├── IJarvisService.aidl             ✅ Binder interface (server-side copy)
├── Android.bp                   ✅ Build config — library: services.jarvis
│
├── core/
│   ├── JarvisStore.java         ✅ ObjectBox singleton
│   ├── ModelRegistry.java       ✅ Named (modelHandle, indexHandle) pairs
│   ├── IndexQueue.java          ✅ Singleton BlockingQueue, cap 500
│   ├── JarvisManager.java          ✅ Public API manager
│   └── JarvisException.java        ✅
│
├── inference/
│   └── CactusWrapper.java       ✅ JNI bridge — only entry point to Cactus
│
├── indexing/
│   ├── JarvisIndexWorker.java      ✅ WorkManager background indexer
│   ├── JarvisFileObserver.java  ✅ Watches Documents/Downloads/Pictures
│   ├── TextExtractor.java       ✅ File → raw text (.txt .md .csv .pdf .docx)
│   └── ChunkingStrategy.java    ✅ Sentence-boundary splitting + overlap
│
├── search/
│   └── MetadataSearch.java      ✅ Stage 1 — ObjectBox keyword scoring (6 passes)
│
├── model/                       ✅ ObjectBox entities
│   ├── SourceFile.java
│   ├── DocumentChunk.java
│   ├── Chunk.java
│   ├── Conversation.java
│   ├── Message.java
│   ├── Folder.java
│   ├── UserContext.java
│   ├── AccessLog.java
│   └── TaskMemory.java
│
└── tools/                       ✅ Phase 4 — Tool Registry
    ├── AppRecord.java           ✅ ObjectBox: one per installed app
    ├── ToolRecord.java          ✅ ObjectBox: one per tool, ToOne<AppRecord>
    ├── ToolScannerService.java  ✅ Scans APKs on install, embeds tools
    └── ToolDispatcher.java      ✅ Resolves + fires tools via broadcast

android/jarvis/                      ← public API layer ✅
├── IJarvisService.aidl             ✅ package android.jarvis

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ocansey11) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
