---
trigger: always_on
description: This file provides a summary of my core understanding of the Agora project, which I use as my primary context for our sessions.
---

This file provides a summary of my core understanding of the Agora project, which I use as my primary context for our sessions.

For a detailed plan of future work, see [[ROADMAP.md]].
For a log of completed work, see [[DONE.md]].
For essays and poems on the project's philosophy, see [[PHILOSOPHY.md]].

---

## Memories & Mandates

-   **Feature Flag Protocol:** Always enable new feature flags in `LocalDevelopmentConfig` first. Never enable in `AlphaConfig` or `ProductionConfig` without explicit user instruction and prior local verification.
-   **Git Operations:** The user prefers to handle commits themselves, or explicitly approve them. Always `git status` and `git diff` before asking. **NEVER use `git add .` or `git add -A` under any circumstances, as other agents might be operating concurrently. Always explicitly list the exact files to add: `git add <file1> <file2>`.**
-   **Development:** Run `npm run build` after TypeScript changes. Do not use `./run-dev.sh`; the user manages the server.
-   **Tone:** Be direct and technical. Use metaphors sparingly (e.g., for "Federation").

---

## Understanding the Agora (as of 2025-09-21)

*This is a summary of my understanding of the project's philosophy and technical principles based on our collaboration.*

### The Agora of Flancia's Intents (The "Why")

The project's goal is to create a **Free Knowledge Commons** with a focus on:
-   **Connecting Digital Gardens**: Weaving together individual, user-owned collections of notes into a larger whole.
-   **Collaborative Problem-Solving**: Building a space where knowledge is not just stored but actively used and composed into tools.
-   **Low Barrier to Contribution**: Prioritizing simple, durable formats (like plain text files) to make it easy for anyone to participate.
-   **Playful Experimentation**: Fostering a delightful and evolving user experience.

### The Agora Protocol (The "How")

The protocol is a set of architectural patterns that enable the Agora's vision:
-   **Decentralization**: The filesystem is the ultimate source of truth. The server is a lens, not a silo.
-   **Nodes are Concepts, Subnodes are Utterances**: A key distinction where abstract topics (`[[Calculus]]`) are composed of concrete contributions (`@user/calculus-notes.md`).
-   **Composition over Centralization**: Nodes are built by pulling and combining content from other, more specialized nodes (e.g., `[[170]]` pulling from `[[calc/170]]`).
-   **Everything Has a Place (No 404s)**: Every possible query resolves to a node, turning dead ends into invitations to contribute.

---

## Agora Core Concepts (as of 2025-09-29)

*This is a summary of my understanding of the project's architecture and user experience principles.*

-   **The Default View is a Composite**: The main user experience is not just a single page, but a composition of a central topic and a suite of satellite information panels. The "default view" for any given node is a collection of:
    -   The **Node Proper**: The collection of subnodes (user contributions) that define the node.
    -   **Contextual Sections**: A series of `<details>` summaries that provide automatically-fetched context about the node's topic from various sources.
-   **Nodes are Collections, Subnodes are Resources**: A key architectural pattern is the distinction between nodes and subnodes.
    -   A **Node** (`[[wikilink]]`) represents a topic, a location, or a collection of things. It is an abstract concept.
-   A **Subnode** (`@user/document.md`) is a specific, concrete resource or "utterance" contributed by a user that is associated with a node.

---

## Agora Caching Architecture (as of 2025-12-06)

*This section clarifies the purpose and contents of different cache layers in the Agora.*

### 1. In-Memory Cache (Python `G` object & `cachetools`)

This cache operates within each running Python worker process and provides the fastest data access, as it relies on direct memory lookups. It is cleared on application restarts or worker reloads.

-   **Monolithic Graph (`ENABLE_LAZY_LOAD = False` - Default/Production):
    -   **What**: Stores the entire graph. This includes all `Node` objects, `Subnode` objects, and crucially, the **full text content** (`Subnode.content`) of all markdown/org-mode/myco files, as well as pre-parsed `forward_links` for all subnodes.
    -   **How**: During application startup (or the first request to each worker in `lazy-apps = true` mode), `G.nodes()` and `G.subnodes()` (which call `_get_all_nodes_cached`) either deserialize the `graph_cache` blob from SQLite or perform a full filesystem scan. The resulting Python objects are then stored in `cachetools` LRU caches within the `G` object.
    -   **Impact**: High RAM usage, but subsequent access to any node or subnode property/content is near-instant, as no disk I/O or further parsing is typically needed.

-   **Lazy-Loaded Graph (`ENABLE_LAZY_LOAD = True` - Experimental):
    -   **What**: Stores individual `Node` and `Subnode` objects as they are accessed. `Subnode.content` is still read from disk upon each `Subnode`'s initialization if not already in memory/cache. The `G.node` method also caches individual `Node` objects.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flancian/agora-server](https://github.com/flancian/agora-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
