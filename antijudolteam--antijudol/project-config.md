---
trigger: always_on
description: AntiJudol is a Bun/Express proxy server that intercepts streaming donation overlay widgets (Saweria, Tako, BagiBagi, Sociabuzz), inspects donation messages in real-time, and replaces blocked content (gambling promotion) at the data source before the overlay renders it. Blocked donations also have their TTS audio stripped.
---

# AGENTS.md — AntiJudol

## Project Overview

AntiJudol is a Bun/Express proxy server that intercepts streaming donation overlay widgets (Saweria, Tako, BagiBagi, Sociabuzz), inspects donation messages in real-time, and replaces blocked content (gambling promotion) at the data source before the overlay renders it. Blocked donations also have their TTS audio stripped.

## Architecture

```
Browser/OBS → localhost:3000/overlay → src/server.js → src/routes/proxy/overlay.js → platform HTML
                                                                ↓
                                                 public/inject.js (bundled from client/)
                                                                ↓
                                                     hooks WS/fetch/XHR in browser
                                                                ↓
                                                 Sync XHR to /check (src/routes/api/check.js)
                                                                ↓
                                                 Modify WS/fetch data → overlay renders filtered content
```

### Repository Layout

The project is a multi-service monorepo. Each service lives under `services/<name>/`.

```
AntiJudol/
├── services/
│   ├── proxy/                Bun/Express proxy — interception, injection, decision routing
│   │   ├── src/ client/ public/ data/ scripts/ tests/
│   │   ├── package.json, bun.lock
│   │   └── Dockerfile
│   └── filter/               FastAPI classifier — ML-based gambling detection
│       ├── app/              FastAPI app code
│       ├── model/            ML model artifacts (download separately, not in git)
│       ├── requirements.txt
│       └── Dockerfile
├── docker-compose.yml        Orchestrates all services
├── .dockerignore
├── .env.example              Single source of truth for env vars (namespaced PROXY_*/FILTER_*)
├── Makefile                  Local Bun + kill-switch shortcuts
├── AGENTS.md / CLAUDE.md / README.md
```

Run commands from a service directory (`cd services/proxy && bun dev`) or via `docker compose` from the repo root. Paths in the **Proxy Service Structure** section below are relative to `services/proxy/`.

### Filter Service (services/filter/)

FastAPI classifier exposing `/api/v1/classify/predict` and `/api/v1/classify/predict/batch`. Loads a Hugging Face model from `model/` at startup. Used by proxy when `PROXY_FILTER_METHOD=classifier`. See `services/filter/README.md` for setup, endpoints, and model download instructions.

The proxy chooses between the in-process algorithm (`src/filter/judolFilter.js`) and the classifier service via `PROXY_FILTER_METHOD`:

- `algorithm` (default for local dev) — uses the built-in pattern/wordlist filter, no Python needed.
- `classifier` — proxy POSTs each donation to `PROXY_FILTER_URL` and uses the model's verdict directly. Falls back to algorithm on classifier failure.
- `both` (recommended in Docker) — runs both. The classifier's `gambling` probability is mapped to ±5 score points and folded into `decide()`'s existing scoring, so a confident classifier can either push a borderline case over the block threshold or pull it back. A `gambling >= 0.85` reading additionally short-circuits the default-allow path with stage `classifier-confident`. Algorithm "block" verdicts are never downgraded by the classifier — signature matches always win. Anti-judol context is also never overridden.

### Proxy Service Structure

```
src/                          Server-side code (Bun/Node)
  server.js                   Entry point — validates config, pre-downloads impersonate binary, starts Express
  config.js                   Environment-driven runtime config
  constants.js                Wire-protocol constants
  platforms.js                Platform configuration registry
  routes/
    index.js                  Route aggregator + JSON body parser
    api/
      check.js                POST /check — donation filter endpoint
    proxy/
      overlay.js              GET /overlay — HTML proxy + injection + native path rewrites
      backend.js              /backend/:platform/* — API proxy
      assets.js               Catch-all — static asset proxy
    web/
      static.js               express.static over public/ (serves index.html + bundled inject.js)
  antibot/
    index.js                  fetchViaAntibot facade — forwards to impersonate
    impersonate.js            cuimp (curl-impersonate) client with cookie jar
  filter/
    judolFilter.js            decide(donator, message) → {action, stage, reason}
    classifier.js             HTTP client for the FastAPI filter service
    normalizeText.js          Homoglyph/leet/decoration normalization with variant expansion
    typoMatcher.js            Fuzzy Indonesian dictionary matcher (canonicalization, near-miss scoring)
    textUtils.js              Shared regexes (chains, leet) + foldLeet, tightenIntraWord, levenshtein

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AntiJudolTeam/AntiJudol](https://github.com/AntiJudolTeam/AntiJudol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
