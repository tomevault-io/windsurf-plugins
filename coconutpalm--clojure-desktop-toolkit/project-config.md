---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project type: Native desktop library (SWT/CDT)

This project is a **native desktop UI toolkit** built on Eclipse SWT. It has no web UI.
The following gstack skills do not apply and should not be invoked:
- `/qa`, `/qa-only`, `/browse`, `/canary`, `/benchmark` — require a browser; SWT windows are not web pages
- `/design-review`, `/design-html`, `/design-shotgun` — generate or inspect web/HTML UI, not SWT

For UI testing, use the REPL screenshot workflow documented in `SWT-UI-RULES.md` instead of `/qa`.
For all UI code, **always read `SWT-UI-RULES.md` first** (see section below).

## SWT UI Rules — REQUIRED READING

**ALWAYS read [`SWT-UI-RULES.md`](SWT-UI-RULES.md) before writing any SWT user interface code.** It contains mandatory rules on UI threading, init function composition, event handling, resource management, layout helpers, platform quirks, and REPL testing workflow.

## Project Overview

Clojure Desktop Toolkit is a library that wraps [Eclipse SWT](https://eclipse.dev/eclipse/swt/) with an idiomatic functional Clojure API for building cross-platform native desktop apps. A key feature is that SWT platform libraries are bundled as ZIP resources inside the JAR and loaded automatically at runtime — users do not install SWT separately.

## Clojure Development

### REPL-First Workflow (Non-Negotiable)

Use `clj-nrepl-eval` (installed on PATH) for all REPL evaluation. **Never edit code files when the REPL is unavailable** — stop and ask the user to restore REPL first.

Before ANY file modification:
1. Read the whole source file
2. Test current behavior in REPL with sample data
3. Develop fix interactively in REPL
4. Verify with multiple test cases
5. Only then write to files

After editing files, reload the namespace: `(require 'my.namespace :reload)`

### Data Structure Conventions

- **Flat structures**: Avoid deep nesting; use namespaced/synthetic keywords (`:foo/something`)
- **Destructure in parameter lists**: `[{:user/keys [id name] :config/keys [timeout]}]`
- **Avoid shadowing core fns**: Don't bind `map`, `name`, `type`, `count`, `set`, `str`, `get`, `filter`, `reduce`, `merge`, `update`, `key`, `first`, `rest`, `keyword`, `symbol`, `class`, `empty?`
- **Use `str` not `.toString`**: `(str val)` is idiomatic Clojure; never call `(.toString val)`

### Alignment Rule

Always align multi-line data structure elements vertically — the bracket balancer depends on consistent indentation:

```clojure
;; ✅ Correct
(select-keys m [:key-a
                :key-b
                :key-c])
```

### Debugging: Inline Def over Println

```clojure
(defn process [data]
  (def data data)   ; inline def — keeps value inspectable in REPL
  ...)
```

### Rich Comment Forms (RCF) and Inline Tests

This codebase uses [Hyperfiddle RCF](https://github.com/hyperfiddle/rcf) for inline tests co-located with source code. The `tests` macro is the standard way to document and verify function behavior — prefer it over plain `(comment ...)` blocks when you want executable examples. Tests are activated via `(rcf/enable!)` in `dev/user.clj`.

Inline test syntax:
```clojure
(ns example
  (:require [hyperfiddle.rcf :refer [tests tap %]]))

(tests
  (inc 1) := 2
  {:a :b} := {:a _}                    ; wildcard
  {:a :b, :b [2 :b]} := {:a ?x, ?x [2 ?x]}  ; unification
  (assert false) :throws AssertionError)
```

Async tests with `tap`/`%`:
```clojure
(tests
  (future (tap 1) (tap 2))
  % := 1
  % := 2)
```

Use plain `(comment ...)` only for exploratory scratch work that you don't intend to keep as verified examples.

### Architectural Integrity

- **Fail fast, fail clearly**: config/service failures → explicit errors, never `(or real-config fallback)`
- **No architectural violations**: functions must not call `swap!`/`reset!` on global atoms; business logic must be separate from side effects
- **Definition of Done**: architectural integrity verified + REPL testing done + zero compilation warnings + zero linting errors + all tests pass

### Dynamic Dependency Loading

```clojure
(require '[clojure.repl.deps :refer [add-libs]])
(add-libs '{some/library {:mvn/version "1.0.0"}})
```
Requires Clojure 1.12+.

## Build Commands

```bash
make jar      # clojure -X:jar :version '"0.4.4"' → clojure-desktop-toolkit.jar
make deploy   # ./deploy.sh → deploy to Clojars
make clean    # rm -f *.jar
```

**Development REPL:**
```bash
clojure -M:dev   # starts REPL with src, resources, dev, and examples on classpath
```

**Tests** use [Hyperfiddle RCF](https://github.com/hyperfiddle/rcf) — tests are inline in source files and activated via `(rcf/enable!)` in `dev/user.clj`. Run them by evaluating the test forms in the REPL.

## Architecture

### Core Pattern: Init Functions

The central abstraction is an **init function**: `(fn [props parent] ...)`. All widget constructors accept a vararg list of init functions (plus some sugar forms) that are called in order on the constructed widget.

- **`props`** — an atom of shared mutable state threaded through the entire UI tree, used for data binding and cross-component references
- **`parent`** — the SWT widget being initialized


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coconutpalm/clojure-desktop-toolkit](https://github.com/coconutpalm/clojure-desktop-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
