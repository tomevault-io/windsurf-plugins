---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Status: Proof of Concept (Alpha)** - APIs may change.

## Project Overview

Atlas is a semantic registry and tooling layer for software architecture. Instead of binding identity to place (files, namespaces), Atlas binds identity to **semantics** - what entities mean and how they relate.

**Key concept:** Every entity has a compound identity like `#{:atlas/execution-function :domain/auth :operation/validate :tier/service}` that IS its semantic meaning, enabling multi-dimensional querying and architectural reasoning.

**Documentation:**
- [Getting Started](docs/getting-started.md) - 5-minute tutorial
- [Core Concepts](docs/concepts.md) - Compound identities, tiers, data flow
- [API Reference](docs/api-reference.md) - Function reference
- [Examples](docs/examples.md) - Annotated patterns
- [Emacs Integration](docs/emacs-integration.md) - IDE support via CIDER
- [Visual Explorer](docs/visual-explorer.md) - Browser-based graph UI (PoC)

## Clojure REPL Evaluation

The command `clj-nrepl-eval` is installed on your path for evaluating Clojure code via nREPL.

```bash
# Discover nREPL servers
clj-nrepl-eval --discover-ports

# Evaluate code
clj-nrepl-eval -p <port> "<clojure-code>"

# With timeout (milliseconds)
clj-nrepl-eval -p <port> --timeout 5000 "<clojure-code>"
```

The REPL session persists between evaluations. Always use `:reload` when requiring namespaces to pick up changes.

## Development Commands

### Running Tests
```bash
# Run all tests with Kaocha
clojure -M:test

# Run specific test namespace
clojure -M:test --focus atlas.registry-test

# Run tests via nREPL
clj-nrepl-eval -p <port> "(require 'atlas.registry-test :reload) (clojure.test/run-tests 'atlas.registry-test)"
```

### REPL Development
```bash
# Start REPL
clojure -M:repl

# Load example app:
(require '[app.calendar-availability :as app])
(app/init-registry!)
```

### Visual Explorer (PoC)

```clojure
(require '[atlas.atlas-ui.server :as ui])
(ui/start!)  ; Opens browser at http://localhost:8082
(ui/stop! 8082)
```

## Code Architecture

This is a **monorepo** with three libraries:

```
atlas/
├── deps.edn           # Root aggregator (dev/test aliases)
├── core/              # atlas - production library
│   ├── deps.edn
│   ├── build.clj
│   ├── build-and-deploy.sh
│   └── src/atlas/
├── dev-tools/         # atlas-dev - dev tooling (depends on core)
│   ├── deps.edn
│   ├── build.clj
│   ├── build-and-deploy.sh
│   └── src/atlas/
├── ui/                # atlas-ui - visual explorer (depends on core)
│   ├── deps.edn
│   ├── build.clj
│   ├── build-and-deploy.sh
│   ├── shadow-cljs.edn
│   ├── package.json   # npm dependencies (React, shadow-cljs)
│   └── src/
└── test/              # Tests (run from root)
```

### Core Library Structure

```
core/src/atlas/
├── core.cljc          # Public API entry point
├── registry.cljc      # Semantic kernel, compound identity algebra
├── query.cljc         # Querying by aspects
├── entity.cljc        # Runtime entity resolution
├── invariant.cljc     # Architectural validation
├── ontology.cljc      # Tools, templates, discovery
├── graph.cljc         # Graph algorithms
└── datalog.cljc       # Datascript backend
```

### UI Dev Tool Structure

```
ui/src/
├── atlas/atlas_ui/    # Server-side (clj)
├── atlas_ui/          # Client-side (cljs/cljc)
└── atlas_ui_v2/       # V2 client
```

## Building & Publishing

### Atlas Core (production library)

```bash
cd core

# Quick: Use build script
./build-and-deploy.sh              # Build JAR
./build-and-deploy.sh --install    # Build and install to ~/.m2
./build-and-deploy.sh --deploy     # Build and deploy to Clojars

# Manual: Direct commands
clojure -T:build jar         # => target/atlas-0.1.0-SNAPSHOT.jar
clojure -T:build install     # Install to local Maven
clojure -X:deploy            # Deploy to Clojars (uses ~/.lein/credentials.clj)
```

### Atlas UI (dev tool)

**First time setup:**
```bash
cd ui
npm install  # Install React, shadow-cljs, etc.
```

**Build:**
```bash
cd ui

# Quick: Use build script (recommended)
./build-and-deploy.sh              # Compile ClojureScript + build JAR
./build-and-deploy.sh --install    # + install to ~/.m2
./build-and-deploy.sh --deploy     # + deploy to Clojars
./build-and-deploy.sh --jar-only   # Skip ClojureScript compilation (faster rebuilds)

# Manual: Step by step
npx shadow-cljs release atlas-ui       # Compile ClojureScript v1
npx shadow-cljs release atlas-ui-v2    # Compile ClojureScript v2
clojure -T:build jar                   # Build JAR
clojure -T:build install               # Install to local Maven
clojure -X:deploy                      # Deploy to Clojars
```

### Clojars Credentials

The deployment scripts (`--deploy` flag) automatically use credentials from `~/.lein/credentials.clj`:

```clojure
{#"https://repo.clojars.org"
 {:username "your-username"
  :password "your-deploy-token"}}
```

Alternatively, set environment variables:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [semantic-namespace/atlas](https://github.com/semantic-namespace/atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
