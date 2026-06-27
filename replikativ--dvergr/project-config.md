---
trigger: always_on
description: The `io.aviso.ansi` library has a transitive dependency that **hangs indefinitely** on `:reload-all`. Always use `:reload` instead:
---

# CLAUDE.md - Dvergr Development Notes

## ⚠️ CRITICAL: Never use :reload-all

The `io.aviso.ansi` library has a transitive dependency that **hangs indefinitely** on `:reload-all`. Always use `:reload` instead:

```clojure
;; GOOD - use this
(require '[dvergr.core :as r] :reload)
(require '[dvergr.agent :as agent] :reload)

;; BAD - will hang forever, requires killing the REPL
(require '[dvergr.agent :as agent] :reload-all)  ; DON'T DO THIS
```

## Project Overview

Dvergr is a Clojure-based AI agent harness supporting multiple LLM providers (Anthropic, OpenAI, Fireworks). It provides:
- Multi-provider SSE streaming
- Tool calling with parallel execution
- Session management with EDN persistence
- Automatic retry with exponential backoff (429, 5xx errors)

## Quick Start

### Start REPL
```bash
clj -Sdeps '{:deps {nrepl/nrepl {:mvn/version "1.3.0"}}}' -M -m nrepl.cmdline --port 0
```

### Run an Agent Task
```clojure
(require '[dvergr.core :as r] :reload)

;; Simple task
(r/run "List all Clojure files and count them"
       :provider :fireworks
       :model "accounts/fireworks/models/qwen3-coder-480b-a35b-instruct"
       :max-turns 5)

;; With Anthropic
(r/run "Explain what core.clj does"
       :provider :anthropic
       :model "claude-sonnet-4-5-20250514")
```

### Testing

```clojure
;; Quick test
(require '[dvergr.core :as r] :reload)
(r/run "Say hello" :provider :fireworks :model "accounts/fireworks/models/qwen3-coder-480b-a35b-instruct")
```

## Recommended Models

### Fireworks.ai (tested, fast, cost-effective)

```clojure
;; Qwen3 Coder 480B - Best for code tasks
:model "accounts/fireworks/models/qwen3-coder-480b-a35b-instruct"

;; Kimi K2 Thinking - Advanced reasoning
:model "accounts/fireworks/models/kimi-k2-thinking"
```

## Architecture

```
dvergr.core       - Public API facade (discourse algebra + personas + proposals)
dvergr.chat.agent - Agent loop, turn execution
dvergr.model.*    - LLM API calls, SSE streaming, retries (model/api/{anthropic,openai,claude_code})
dvergr.tools      - Tool registry, execution
dvergr.chat.context - Per-chat state (signals + datahike + sci)
```

## Provider Support

| Provider | Streaming | Tools | Status |
|----------|-----------|-------|--------|
| Anthropic | Yes | Yes | Tested |
| OpenAI | Yes | Yes | Implemented |
| Fireworks | Yes | Yes | Tested |

## Error Handling

- Automatic retry (3 attempts) for: 429, 500, 502, 503, 504
- Exponential backoff with jitter
- Clear error messages in agent output

## Future Integration

See **DESIGN.md** for full architecture document.

Planned integration with:
- **SCI** for sandboxed REPL isolation (each session gets isolated ctx)
- **Datahike** for REPL history storage and querying
- **Yggdrasil** for CoW branching (fork/merge sessions)
- **Scriptum** for versioned source code + fulltext search
- **Beichte** for purity analysis enabling JIT compilation

## Key Design Decisions

1. **SCI by default** - All evals go through SCI for isolation
2. **State in datahike** - REPL history, conversations, ctx snapshots
3. **Branches = Sessions** - Each session maps to a yggdrasil branch
4. **Pure functions can be promoted** - JIT from SCI to JVM when safe

## Spindel Integration (../spindel)

Spindel is the reactive runtime powering agent composition. Key concepts:

### Core Abstractions

**CRITICAL DESIGN PRINCIPLE**: Spindel is an **FRP (Functional Reactive Programming) system** for composable reactive computations, NOT a future/promise library.

- **@deref is ONLY for REPL convenience** - DO NOT use `@spin` inside other spins!
- **Compose spins using `await` inside `spin` macro** - builds reactive dependency graphs
- **Spins are cached and automatically re-execute** when dependencies change
- **The goal is composition**, not one-off blocking calls that lose reactivity

```clojure
;; CORRECT - Composable reactive pattern
(def result-spin
  (spin
    (let [a (await spin-a)  ; Non-blocking suspension
          b (await spin-b)]
      (+ a b))))  ; Re-executes when spin-a or spin-b change

;; WRONG - Breaks composition
(defn broken []
  (let [a @spin-a  ; Blocking deref - loses reactivity!
        b @spin-b]
    (+ a b)))

;; REPL/Test boundary - @deref is OK here
(binding [rtc/*execution-context* runtime]
  @result-spin)  ; Fine - this is the REPL boundary

;; Spin - reactive computation unit (like React hooks or SolidJS signals)
(ns ... (:require [is.simm.spindel.spin.cps :refer [spin]]
                  [is.simm.spindel.effects.await :refer [await]]
                  [is.simm.spindel.effects.track :refer [track]]
                  [is.simm.spindel.runtime.core :as rtc]
                  [is.simm.spindel.spin.combinators :as comb]))

;; Create a spin - automatically CPS-transformed
(spin
  (let [x (await some-other-spin)
        y (await another-spin)]
    (+ x y)))

;; Lower-level API (no CPS transformation)
(spin-core/make-spin
  (fn [resolve reject]
    ;; Manual CPS - call resolve/reject when done
    (resolve 42))
  :my-spin-id)

;; Dynamic bindings - CRITICAL for all spindel operations
(binding [rtc/*execution-context* runtime-ctx
          rtc/*spin-id* current-spin-id]
  ;; Spindel code here
  )
```

### Two Categories of Abstractions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [replikativ/dvergr](https://github.com/replikativ/dvergr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
