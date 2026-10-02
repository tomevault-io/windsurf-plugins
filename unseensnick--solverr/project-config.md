---
trigger: always_on
description: FlareSolverr fork with two solving engines and automatic fallback. Cloudflare/DDoS-GUARD bypass proxy speaking the FlareSolverr `/v1` API on port 8191. Python 3.14, `bottle` + `waitress` (synchronous WSGI). Lives alongside its two upstreams as read-only reference: `../FlareSolverr` (the Chrome engine's origin) and `../Byparr` (the Camoufox stack's origin).
---

# Solverr

FlareSolverr fork with two solving engines and automatic fallback. Cloudflare/DDoS-GUARD bypass proxy speaking the FlareSolverr `/v1` API on port 8191. Python 3.14, `bottle` + `waitress` (synchronous WSGI). Lives alongside its two upstreams as read-only reference: `../FlareSolverr` (the Chrome engine's origin) and `../Byparr` (the Camoufox stack's origin).

## Commands

```bash
docker compose up -d --build         # build + run (image bundles both browsers, ~2.3 GB)
docker logs -f solverr               # logs (set LOG_LEVEL=debug for more)
PYTHONPATH=src uv run --no-project python -m unittest discover -s src -p 'test_*.py' -t src   # browser-free suite, seconds; CI runs it
bash .githooks/tests/run.sh          # the git hooks still reject what they claim to
bash .claude/hooks/tests/run-all.sh  # the Claude Code guard hooks, against their fixtures
uv run --no-project python -m py_compile src/*.py src/engines/*.py   # quick compile check
```

Python only through uv; there is no system Python. `src/tests.py` is upstream's suite: it needs a real browser and live sites. Whether a page still clears a real challenge is `/live-check`, never the unit tests.

## Working approach

- **Memory and `Handoff.md` are hypotheses, not facts.** A memory that names a function, file or flag is true only if it still exists in current code. When one turns out stale, surface it for pruning instead of acting on it.
- **Plan steps carry their check inline**, as `1. <step> -> verify: <check>`, so a step nothing can check is visible before it is built.
- **Reply length.** Default replies are a few sentences: the answer or outcome, the detail that matters, done. A full report is for when the owner asks for one, or for a `/scout` or `/code-research` deliverable, which has its own cap in [.claude/rules/plan-output.md](.claude/rules/plan-output.md).

## Architecture in brief

Two engines behind one interface: `chrome` (Selenium + vendored undetected_chromedriver, the default) and `stealth` (Camoufox via invisible_playwright, on one background event-loop thread). The controller falls back between them and remembers per host which one cleared it. What both engines must do the same way lives once in the shared spine (`assembly.py`, `pipeline.py`, `budget.py`, `sessions.py`); each engine is an adapter over a clearing core derived from its upstream. Several constraints look wrong until you know what they were measured against (the coordinate Turnstile click, no `page.evaluate` against a challenge page, the second look before a challenge counts as cleared, the even `maxTimeout` split, the `quote()` calls in `postform.py`): read [.claude/rules/architecture.md](.claude/rules/architecture.md) before touching any of them. It loads on its own when you work in `src/`.

## Key decisions (WHY)

- **Fork on FlareSolverr, not Byparr.** FlareSolverr's Chrome engine already clears the target sites and has sessions, and its vendored undetected_chromedriver lets the Camoufox/Playwright stack run beside it in one Python 3.14 image.
- **Reliability is dominated by IP reputation, not the tool.** A residential proxy (`PROXY_URL`) is the biggest lever; warm-session cookie reuse is the second.
- **The consuming client keeps one shared session and never destroys it**, so the server-side reaper is what prevents leaked browsers (especially the heavier Camoufox ones).

## Where things live

- `src/flaresolverr.py`: entrypoint. Logging setup (note the `force=True`), server, reaper start.
- `src/flaresolverr_service.py`: controller. `/v1` commands, engine selection and fallback, per-host memory, session commands.
- `src/assembly.py`, `src/pipeline.py`, `src/budget.py`: the shared spine. What a response contains and in what order, the page verdict and the navigate-cookies-reload order, the solve deadline. The first two are sans-io generators (they yield what to read, the engine supplies how) because one engine is synchronous and the other asynchronous; see `.claude/rules/engine-layer.md` before reshaping them.
- `src/engines/`: `base.py` (Engine + SolveResult), `chrome_engine.py`, `stealth_engine.py`. Each is an adapter over an upstream-derived clearing core, which is the one thing the spine never takes over.
- `src/async_runtime.py`, `src/session_reaper.py`, `src/sessions.py`: stealth event loop, idle reaper, and the `SessionStore` both engines use (each holds its own instance; the lifecycle rules live once).
- `src/detection.py` (shared challenge/title/selector lists), `src/geo.py` (browser timezone and language for both engines), `src/config.py` (env, including `env_proxy`), `src/postform.py`, `src/redact.py` (what the `/v1` request and response lines, the Chrome proxy debug line and the geo warning strip before they are written), `src/dtos.py` (request DTOs plus the type validation that makes their annotations binding).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unseensnick/Solverr](https://github.com/unseensnick/Solverr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
