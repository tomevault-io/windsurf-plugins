---
trigger: always_on
description: Forward-chaining expert system shell in Common Lisp (Rete algorithm, CLOS/MOP, certainty factors). Integrated with Claude via tool-use for natural-language medical diagnosis (MYCIN rulebase).
---

# Lisa — Lisp-based Intelligent Software Agents

Forward-chaining expert system shell in Common Lisp (Rete algorithm, CLOS/MOP, certainty factors). Integrated with Claude via tool-use for natural-language medical diagnosis (MYCIN rulebase).

## Build & Load

Requires SBCL with Quicklisp. From the SBCL REPL at project root:

```lisp
;; Load Lisa core
(load "lisa.asd")
(asdf:load-system :lisa)

;; Load the MYCIN rules (classes + rules, no assertions)
(in-package :lisa-user)
(load "examples/mycin.lisp")

;; Load and start the LLM bridge
(asdf:load-system :lisa-bridge)
(lisa-bridge:start)  ; starts on port 8090
```

To stop: `(lisa-bridge:stop)`

### Choosing a belief system

The bridge honors `LISA_BELIEF_SYSTEM` at startup. **Dempster-Shafer is the
default** — it exposes ignorance intervals `{bel, pl, ignorance}` that the
LLM can narrate meaningfully. Override with the env var:

```bash
LISA_BELIEF_SYSTEM=cf sbcl ...   # certainty factors (Shortliffe-Buchanan)
LISA_BELIEF_SYSTEM=ds sbcl ...   # Dempster-Shafer (default)
```

Per-session overrides ride on `POST /reset` with body `{"belief_system":
"cf" | "ds"}`. `/conclusions` echoes the active system in its response
and emits `{bel, pl, ignorance}` payloads under DS.

## Project Structure

```
lisa.asd              — Core system definition (depends on log4cl)
lisa-bridge.asd       — Bridge system (depends on lisa, hunchentoot, jzon, bordeaux-threads)
src/
  core/               — Rete engine, rules, facts, conflict resolution
  belief-systems/     — Pluggable belief-system protocol
    protocol.lisp     — Generic function surface + dispatcher + use-system
    certainty-factors/— Shortliffe-Buchanan CF implementation
    dempster-shafer/  — Simplified [Bel, Pl] interval implementation
  rete/reference/     — Rete network nodes and compiler
  llm/bridge/         — HTTP bridge for LLM integration
    package.lisp      — :lisa-bridge package
    session.lisp      — Entity registry, session reset
    server.lisp       — Hunchentoot start/stop; LISA_BELIEF_SYSTEM env var
    handlers.lisp     — REST endpoints (belief-system-aware)
  llm/claude/         — Claude tool-use integration
    driver.py         — Python client: tool-call loop; transcript capture
    tools.json        — Tool schemas (assert_fact, run_inference, get_conclusions, etc.)
    system-prompt.md  — Clinical diagnostic system prompt (18 rules, CF/DS output)
examples/
  mycin.lisp          — MYCIN rules (18 rules incl. 3 disconfirming; culture-1, culture-1a, culture-2, culture-3)
bin/
  test-culture-1.sh   — End-to-end bridge test (curl-based)
  run-mycin.sh        — Same as test-culture-1.sh (legacy name)
```

## Bridge Endpoints (port 8090)

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/health` | GET | Health check |
| `/assert-fact` | POST | Assert a fact: `{fact_type, value, entity?, entity_class?, confidence?}` |
| `/run-inference` | POST | Fire rules (captures rule trace) |
| `/conclusions` | GET | Get organism-identity results + belief factors |
| `/rule-trace` | GET | Get which rules fired last run |
| `/reset` | POST | Clear working memory and entity registry |

## Testing the Bridge

```bash
# Start the bridge first (see Build & Load above), then:
./bin/test-culture-1.sh
```

Expected: culture-1 scenario produces pseudomonas (0.6) and enterobacteriaceae (0.8).

## Running the Test Suite

Dependency-free golden-master + belief-algebra tests live in `tests/` as the
`lisa/test` ASDF system (no external framework). From an SBCL REPL with Lisa loaded:

```lisp
(asdf:test-system :lisa)                 ; runs the suite; errors on any failure
;; or, for the raw report / interactive use:
(asdf:load-system "lisa/test")
(lisa-test:run-all)                      ; => T iff all pass; prints pass/fail counts
```

Coverage: both belief algebras (CF and DS) directly, all four `culture-*` scenarios
under each system with hand-verified golden values, the DS clamp / total-conflict /
malformed-input edge cases, **each of the 18 rules fired in isolation** (confirming
rules contribute exactly their `:belief`; disconfirming rules drop plausibility below
1.0), and behavioral properties (confirmatory evidence keeps
`pl = 1.0`; conflicting evidence drops it below 1.0; CF and DS agree without conflict
and diverge with it). Golden values were captured from the 4.1.0 engine; if a belief
computation changes intentionally, re-capture and update `tests/scenarios.lisp`.

## Key Packages

- `lisa` / `lisa-user` — Core engine and user-facing DSL (defrule, assert, run, reset)
- `belief` — Certainty factor operations (belief-factor, combine-beliefs)
- `lisa-bridge` — HTTP bridge (start, stop, reset-session)

## LLM Integration Status

Both phases are complete:

- **Phase 1 — HTTP Bridge**: Hunchentoot server exposing Lisa's inference engine as REST endpoints (assert-fact, run-inference, conclusions, rule-trace, partial-matches, reset). Belief-system-aware: startup-configurable via `LISA_BELIEF_SYSTEM` and per-session overridable via `/reset`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [youngde811/Lisa](https://github.com/youngde811/Lisa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
