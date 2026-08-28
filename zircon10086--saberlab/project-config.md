---
trigger: always_on
description: You are working on **SaberLab**, a local-first Beat Saber training laboratory.
---

# AGENTS.md



## Purpose



You are working on **SaberLab**, a local-first Beat Saber training laboratory.



SaberLab is not merely a score viewer or replay player. Its long-term purpose is to provide a reliable, deterministic, and extensible environment for:



* Beat Saber replay analysis
* Motion telemetry
* Player performance analysis
* Training experiments
* 3D replay visualization
* Score and difficulty analysis
* AI-assisted coaching



The project may use AI/Vibe Coding extensively, but **AI-generated code must preserve the existing architecture and data integrity**.



Your primary objective is:



> \\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\*Extend SaberLab without breaking SaberLab.\\\\\\\\\\\\\\\*\\\\\\\\\\\\\\\*



When implementing a feature, aim for a precise, elegant solution that fulfills the functionality first, while keeping the change surface as small as practical and balancing efficiency and safety. Never sacrifice correctness, efficiency, or result quality merely to minimize the diff.



\---



# 1\. Core Principles



These principles are mandatory.



## 1.1 Local-first



SaberLab is designed to work primarily with local data.



Prefer:



* local Beat Saber maps
* local `.bsor` replays
* local database/cache
* local deterministic analysis
* local rendering



Network services such as ScoreSaber, BeatLeader, or LLM providers are supplementary.



A network failure must not corrupt local data or prevent core local functionality from working whenever practical.



Do not make a previously local feature network-dependent without an explicit architectural decision.



\---



## 1.2 Deterministic-first



All factual replay and gameplay analysis must be deterministic.



The following belong to deterministic code:



* BSOR parsing
* score calculation
* timing calculations
* swing metrics
* motion telemetry
* path metrics
* fatigue indicators
* map analysis
* statistical calculations
* player history calculations



LLMs must not become the source of truth for numerical gameplay data.



The correct data flow is:



```text

Raw Replay / Map

      ↓

Deterministic Parsing

      ↓

Deterministic Analysis

      ↓

Structured Results

      ↓

AI Interpretation

```



Never reverse this dependency.



The AI may interpret, explain, compare, summarize, or recommend based on structured facts, but it must not invent or silently replace the underlying measurements.



\---



## 1.3 Raw Replay Integrity



Original replay files are read-only source data.



Rules:



* NEVER modify the original `.bsor` file.
* NEVER write derived analysis results into the original replay.
* NEVER use the replay file itself as a cache.
* Store derived information separately.
* Preserve reproducibility: the same replay and analysis version should produce the same deterministic result unless an intentional algorithm change has been introduced.



\---



## 1.4 Preserve Architectural Boundaries



Do not bypass existing architecture merely because a shortcut is easier.



The project is intentionally divided into logical layers.



Typical ownership:



```text

backend/

├── bsor/       Replay parsing and replay-related primitives

├── maps/       Map parsing and map-related primitives

├── analysis/   Deterministic gameplay and motion analysis

├── db/         Persistence and database access

├── services/   External integrations and service logic

├── scoresaber.py  ScoreSaber integration

├── ai/         LLM providers and AI interpretation

├── config/     Configuration schema and configuration handling

├── host.py        Local HTTP host / application integration

└── desktop.py   Desktop/WebView integration



frontend/

└── Main SaberLab UI


plugins/

└── First-party plugins (detected & loaded at startup by convention;
    no third-party plugin interface)


(仓库外) Local-ChroViewer/

└── First plugin instance: independent 3D replay application (external
    GPL-2.0 project, NOT part of this repository; build output loaded
    from plugins/chro/)

```



The exact directory structure is authoritative from the current repository. Do not invent a competing architecture.



\---



# 2\. Layer Ownership



When implementing a feature, determine which layer owns the behavior BEFORE writing code.



## Replay / BSOR



Responsible for:



* reading replay files
* decoding replay structures
* replay-level primitives



Do not place UI logic here.



\---



## Maps



Responsible for:



* map discovery
* map parsing
* map metadata
* difficulty information directly derived from maps



Do not place UI-specific logic here.



\---



## Analysis



Responsible for deterministic computation.



Examples:



* accuracy
* timing deviation
* swing velocity
* angular velocity
* cut distance
* path efficiency
* direction changes
* section statistics
* fatigue-related metrics
* other measurable replay telemetry



Rules:



* No LLM dependency.
* No UI dependency.
* No browser-specific behavior.
* No network dependency unless explicitly required by the algorithm.
* Prefer pure/reproducible functions.
* Keep numerical logic testable.



\---



## Database



Database access must remain centralized.



Current repository conventions must be respected, including the repository/model separation.




<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zircon10086/SaberLab](https://github.com/Zircon10086/SaberLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
