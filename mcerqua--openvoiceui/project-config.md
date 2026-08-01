---
trigger: always_on
description: Instructions for AI coding agents working in the OpenVoiceUI repository.
---

# AGENTS.md

Instructions for AI coding agents working in the OpenVoiceUI repository.

This file is the vendor-neutral orientation doc. It does not replace
[CONTRIBUTING.md](CONTRIBUTING.md) (process, branches, PR rules) or
[ARCHITECTURE.md](ARCHITECTURE.md) (how the system is built) — read those too. If anything
here conflicts with CONTRIBUTING.md, CONTRIBUTING.md wins.

---

## 1. What this project is

OpenVoiceUI is the **voice and canvas shell** in front of an agent. It is a Flask backend
plus a vanilla-JavaScript ES-module frontend. The agent itself is not in this repo — it
runs behind a gateway (OpenClaw by default, or any plugin gateway). See
[VISION.md](VISION.md) for positioning and non-goals.

Practical consequence: if a task sounds like "make the model smarter", it is almost
certainly out of scope here. If it sounds like "make the shell hear, speak, show, or
extend better", it belongs here.

---

## 2. Orient yourself first

Before writing code, read in this order:

1. `ARCHITECTURE.md` — the section covering the subsystem you are touching.
2. The module docstring of the file you are about to edit. Almost every Python module in
   this repo opens with a docstring stating its responsibility, its usage pattern, and
   often the reason a design decision was made. Those docstrings are authoritative.
3. The nearest reference doc under `docs/` (`docs/reference/`, `docs/extending/`,
   `docs/customization/`).

Do not infer structure from filenames. `providers/tts/` and `tts_providers/` sound
interchangeable and are not — one is an adapter shim, the other is the live
implementation. That distinction is documented in `providers/tts/__init__.py`.

---

## 3. Repository map (short form)

| Path | What lives there |
|---|---|
| `server.py` | Entry point: registers blueprints, WebSocket routes, startup wiring |
| `app.py` | `create_app()` Flask application factory, returns `(app, sock)` |
| `index.html` | Thin shell that loads `src/app.js` as a module |
| `routes/` | Flask blueprints, one file per feature area |
| `services/` | Backend services (gateways, TTS, plugins, auth, paths, vault, health) |
| `services/gateways/` | `GatewayBase` + the OpenClaw implementation |
| `providers/` | Provider ABCs (`base.py`) + the registry (`registry.py`) for LLM/STT/TTS |
| `tts_providers/` | Canonical TTS implementations — add new TTS backends here |
| `plugins/` | Drop-in plugins: gateways, pages, routes, faces |
| `config/` | YAML/JSON config + `loader.py` with env overrides |
| `profiles/` | Agent profile JSON, schema, and manager |
| `src/` | Frontend: `core/`, `shell/`, `adapters/`, `providers/`, `face/`, `features/`, `ui/`, `styles/` |
| `default-pages/`, `default-faces/`, `default-styles/` | Shipped canvas pages, face template, style presets |
| `runtime/` | Gitignored runtime data. Never commit anything from here |
| `tests/` | pytest suite |

---

## 4. Running and testing

```bash
python3 -m venv venv
venv/bin/pip install -r requirements.txt
cp .env.example .env          # set CLAWDBOT_AUTH_TOKEN and GROQ_API_KEY at minimum
venv/bin/python3 server.py    # serves on http://localhost:5001 by default
```

Tests:

```bash
venv/bin/python3 -m pytest tests/ -q
```

Run the full suite before proposing a change. If your change touches TTS, provider
registration, canvas manifest handling, or the config loader, there is already a test file
for it — extend it rather than creating a parallel one.

Docker (`docker compose up`) is the alternative path; see `README.md`.

**You do not need a gateway to work on most of this repo.** Frontend UI, TTS providers,
canvas pages, config, docs, and tests are all reachable without one. If you need a gateway
that responds, use `plugins/example-gateway/` — it echoes.

Port defaults to 5001 (`config/default.yaml`, override with `PORT`).

---

## 5. Extension points

These are the places designed for new code. Prefer them over editing core files.

| You want to add | Go here | Wiring step |
|---|---|---|
| A TTS backend | `tts_providers/myprovider_provider.py` | add to `_PROVIDERS` in `tts_providers/__init__.py` + a block in `providers_config.json` |
| An STT backend (server-side) | `providers/stt/myprovider_provider.py` | `registry.register(ProviderType.STT, 'id', Class)` + import in `providers/stt/__init__.py` + `stt.modules` in `config/providers.yaml` |
| An STT backend (browser-side) | `src/providers/MySTT.js` | follow `WebSpeechSTT.js` (export STT class + `WakeWordDetector`) |
| A different agent backend | `plugins/my-gateway/gateway.py` | `plugin.json` with `"provides": "gateway"`, class subclasses `GatewayBase` |
| A frontend agent/voice framework | `src/adapters/my-adapter.js` | copy `_template.js`, add to `ADAPTER_PATHS` in `src/shell/adapter-registry.js`, add a profile |
| A face/avatar | `src/face/MyFace.js` or a plugin `faces[]` entry | extend `BaseFace`, add to `src/face/manifest.json` |
| A canvas page | `default-pages/*.html` or a plugin `pages[]` entry | plugin pages are auto-copied and registered in the canvas manifest at load |
| A new API surface | `routes/my_feature.py` | define `my_feature_bp`, register it in `server.py` |
| An agent persona | `profiles/my-profile.json` | validate against `profiles/schema.json` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MCERQUA/OpenVoiceUI](https://github.com/MCERQUA/OpenVoiceUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
