---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **browser environment patching toolkit** for crawler reverse engineering. The core technique: build a fake browser environment (`window`, `document`, `navigator`, etc.) inside Node's `vm` sandbox, load the target's obfuscated JS signing bundle as-is, and capture its output — producing gating signatures offline without a headless browser.

Full documentation and the 7-phase workflow live in [README.md](README.md). This file covers architecture, commands, and constraints that aren't obvious from reading individual files.

## Commands

```bash
# Phase 1: Auto-capture JS bundles from a live page
npx playwright install chromium                                     # one-time setup
node core/capture_sdk.js --url "https://www.<target>.com/"          # basic capture
node core/capture_sdk.js --url "..." --timeout 15000 --min-size 30000 --screenshot
node core/capture_sdk.js --url "..." --cookie "session=abc;token=x" # with cookies
node core/capture_sdk.js --url "..." --pattern "signer|vmp"         # filter JS URLs by regex

# Phase 2: Self-healing trace (auto-generates env stub on stdout)
node core/trace_env.js bundles/signer.js > bundles/fake_env.js
node core/trace_env.js --max-rounds 12 --init bundles/signer.js     # --init is a boolean flag (triggers Phase 4 XHR probe); signer.js is positional
node core/trace_env.js --no-stub bundles/signer.js                  # trace-only, no stub output

# Phase 4-6: One-shot sign (fill in the // TODO placeholders in sign.js first)
node core/sign.js "https://www.<target>.com/api?param=1"            # one-shot (URL is positional)
node core/sign.js --server                                           # persistent JSONL server mode
node core/sign.js --server --ua "Mozilla/5.0 ... Chrome/120.0.0.0"  # server with custom UA

# Phase 7: Validate — must use curl_cffi, not requests/urllib
pip install curl_cffi
python -c "import curl_cffi.requests as cr; r = cr.get(url, impersonate='chrome120'); print(r.json())"

# Persistent signer from Python (starts node core/sign.js --server internally)
SIGN_JS=./core/sign.js TARGET_URL=https://... python core/persistent_signer.py
```

There is no `package.json`, build step, or test suite — this is a template/reference kit, not an installable package.

`bundles/` does not exist at clone time; it is created at runtime by `capture_sdk.js`.

When the user invokes the `fast-spider` skill, follow the 7-phase workflow table defined in `.claude/skills/fast-spider.md`. Skills in `.claude/skills/` must be placed in their own subdirectory with a `SKILL.md` file (e.g., `.claude/skills/my-skill/SKILL.md`). They are auto-discovered — no registration in settings.json needed.

## Architecture: data flow

The 4 core files form a pipeline aligned with the 7 phases:

```
capture_sdk.js               trace_env.js                 fake_env.js
(Phase 1)                    (Phase 2+3)                  (Phase 3 manual alt)
     │                             │                           │
     │ captured JS bundles         │ auto-healing Proxy        │ ~400-line typed stub
     │ saved to bundles/           │ traces all property       │ for vm.createContext
     │ + manifest.json             │ accesses, then            │
     │                             │ generates env stub        │
     │                             │ on stdout                 │
     ▼                             ▼                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                      core/sign.js  (customize this template)      │
│  require()s the env stub, loads SDK in vm.runInContext(),       │
│  calls sign(), outputs JSONL to stdout                          │
└─────────────────────────────────────────────────────────────────┘
                                    │
                                    │ JSONL stdin/stdout
                                    ▼
                          persistent_signer.py
                          (Phase 7 bridge)
```

**Key architectural decisions:**
- `trace_env.js` auto-generates stubs via heuristics in `guessDefault()` (~80 property name patterns). It wraps every object in recursive Proxy handlers (`makeHealer`, depth=2), auto-creates missing properties on access, and re-runs the SDK up to `--max-rounds` (default 8) when it crashes. Diagnostics go to stderr; the final env stub goes to stdout. This replaces the traditional 3-5 round manual iteration.
- `fake_env.js` is the manual alternative — a static ~400-line typed stub you edit by hand. Use it when you want full control or when the auto-healer generates wrong-typed stubs that cause silent failures.
- Both the auto-generated stub and `fake_env.js` export the same signature: `buildFakeBrowser(opts)` → returns a window object with `W.window = W` circular reference. `sign.js` auto-detects which one to use: tries `bundles/fake_env.js` first, falls back to `core/fake_env.js`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alei-xi/fast-spider-skills-kit](https://github.com/alei-xi/fast-spider-skills-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
