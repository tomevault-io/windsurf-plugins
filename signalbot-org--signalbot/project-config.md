---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Signalbot is a Python framework for building Signal bots. It talks to Signal via
[signal-cli-rest-api](https://github.com/bbernhard/signal-cli-rest-api) (run in `json-rpc` mode), receiving
messages over a websocket and sending them over HTTP. This is the `v2.0` branch (v2 rewrite in progress,
pre-release: `version = "2.0.0.dev1"`); `main` still ships v1.

## Commands

Use `uv run <cmd>` for everything (never activate `.venv` directly).

```bash
uv sync --all-groups               # install every dep group (dev+examples+docs) — use this locally, don't
                                    # flip between --group flags mid-session (CI itself only uses --group examples)

uv run pytest                                              # run all tests
uv run pytest tests/unit/test_bot.py                       # single file
uv run pytest tests/unit/test_bot.py::test_name -v          # single test
uv run pytest --cov=src/signalbot --cov-branch --cov-report=xml   # with coverage (matches CI)

uv run prek install    # one-time: install git hooks (ruff-check --fix, ruff-format, ty, yamlfix, uv-lock)
uv run prek run --all-files   # run all lint/format/type-check hooks manually
uv run ty check         # type check only

uv run zensical serve            # serve docs locally
uv run zensical build --clean    # build docs (always pass --clean; build via zensical, not mkdocs)

uv run datamodel-codegen --profile signal-cli-rest-api   # regenerate src/signalbot/_generated/
```

CI (`.github/workflows/ci.yaml`) runs `prek` (all hooks) and `pytest --cov` on every push to `main` and every PR.

## Architecture

### Message flow is bidirectional and file-organized by direction

- **Incoming**: `signal-cli-rest-api` pushes over websocket → `MessagePipeline._produce` (`_pipeline.py`) reads
  raw JSON → `signalbot.messages.parser.parse()` turns it into a typed `ReceivedMessage` → dispatched to a
  queue → `MessagePipeline._consume` pulls jobs and invokes the matching handler method.
- **Outgoing**: bot author code calls a method on one of the `*Actions` classes (`bot.messages`, `bot.reactions`,
  `bot.polls`, ...) → resolves recipients → builds a wire request → calls a method on `SignalAPI` (`_client/`)
  → HTTP request to `signal-cli-rest-api`.

**Extending either direction is a well-defined, multi-file checklist — read
[`docs/06_extending.md`](docs/06_extending.md) before adding a new incoming message type or outgoing action.**
It names every file that needs to change, in order.

### Layering, roughly innermost to outermost

1. `src/signalbot/_generated/` — models generated from signal-cli-rest-api's Swagger/JSON-Schema
   (`_generated/api` = requests, `_generated/data`/`_generated/receive` = responses/incoming). **Never
   hand-edit** — rerun `datamodel-codegen` instead (see `_generated/README.md`).
   `signalbot._generated` types must never leak through the public API, even nested inside a field —
   enforced by `tests/unit/test_public_api_surface.py`. Every top-level domain package (`messages/`,
   `groups/`, `polls/`, `reactions/`, `attachments/`, `contacts/`) wraps the generated type it needs in a
   thin domain subclass (docstring at minimum; real fields/methods only when wire data alone isn't enough).
2. `src/signalbot/_client/` — one file per API section (`messages.py`, `groups.py`, `polls.py`, ...), each
   with a `*URIs` class (endpoint paths) and a `*Client` class (typed to accept generated request models,
   serializes with `model_dump_json(exclude_none=True, by_alias=True)`, calls `self._request(...)`). All
   wired together on `SignalAPI` (`_client/signal_api.py`), internal only — not part of the public
   API surface (`signalbot.client` only re-exports `ConnectionMode`).
3. `src/signalbot/_actions/` — one `*Actions` class per API section (`MessageActions`, `GroupActions`, ...),
   attached to `SignalBot` in `_bot_init.py` as `bot.messages`, `bot.polls`, etc. (`GroupActions` is the
   exception, nested at `bot.groups.actions` since `bot.groups` is already the `GroupRegistry` cache).
   Resolves recipients via `RecipientResolver` (`_recipients.py`), converts domain requests to generated
   ones, calls the client, logs, returns a friendly domain-level result.
4. `src/signalbot/context/` — one `*Context` class per incoming message kind (`DataMessageContext`,
   `ReactionContext`, ...), passed into handler methods. Wraps the `SignalBot` + received message and exposes
   convenience methods (`context.send(...)`, `context.react(...)`) that delegate to the Actions layer, filling
   in fields only knowable once a message has been received (e.g. `recipient`).
5. `src/signalbot/handlers.py` — one `*Handler` ABC per message kind (`DataMessageHandler`,
   `ReactionHandler`, ...) with a single abstract `handle_xxx(self, context)` method that bot authors
   subclass, plus trigger decorators (`text_triggered`, `regex_triggered`, `reaction_triggered`) that filter
   before calling through.
6. `src/signalbot/_pipeline.py` — `_MESSAGE_DISPATCH` is the single map tying a parsed message type to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [signalbot-org/signalbot](https://github.com/signalbot-org/signalbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
