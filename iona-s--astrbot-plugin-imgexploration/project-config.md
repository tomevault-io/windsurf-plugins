---
trigger: always_on
description: This file applies to the `astrbot_plugin_imgexploration` repository.
---

# AGENTS.md

## Scope

This file applies to the `astrbot_plugin_imgexploration` repository.

The plugin supports image searches from command messages, replied-to images,
and follow-up images submitted during a command wait. It also captures remote
image URLs for session context and exposes optional LLM tools.

Keep changes minimal, compatible with existing adapters, and suitable for
public maintenance. This directory is an independent Git repository even when
checked out under an AstrBot installation. Do not run parent-repository setup,
formatting, release, or commit commands unless the task explicitly includes
that repository.

## Working Rules

- Prefer existing plugin APIs and patterns over new abstractions.
- Keep fixes focused. Separate capture, command interaction, session behavior,
  provider, persistence, and framework changes unless the task requires them
  together.
- Do not modify AstrBot framework or adapter files for a plugin-level fix.
- Treat adapter-specific event data as optional and untrusted.
- Update tests and user documentation when behavior or configuration changes.
- During behavior-preserving refactors, retain useful comments and docstrings;
  edit them only when ownership, names, or behavior make the original wording
  inaccurate.

## Repository Map

- `main.py`: plugin lifecycle, message listener, command flow, and LLM tools.
- `core/image_context.py`: image records, session isolation, retention, and
  retrieval.
- `core/image_sources.py`: component, raw-event, and reply image source
  resolution.
- `core/image_wait.py`: command wait isolation, timeout, cancellation, and
  lifecycle coordination.
- `core/service.py`: search-strategy orchestration and result aggregation.
- `core/result_sender.py`: result formatting, platform delivery, retries, and
  fallback behavior.
- `core/strategy.py` and `core/providers/*_strategy.py`: provider interface and
  implementations.
- `core/utils.py`: image resolution, uploads, downloads, and shared HTTP
  resources.
- `_conf_schema.json`: plugin configuration schema.
- `metadata.yaml`: plugin metadata.
- `tests/test_plugin_imports.py`: plugin entry-point and core-layout import
  smoke checks.
- `tests/test_image_capture.py`: component and raw-event capture behavior.
- `tests/test_image_sources.py`: source validation, ordering, raw extraction,
  and reply fallback.
- `tests/test_command_search.py`: immediate command and reply searches.
- `tests/test_result_sender.py`: result components, retries, and delivery
  fallback.
- `tests/test_image_wait.py`: wait configuration and coordinator state machine.
- `tests/test_image_wait_flow.py`: plugin wait flow, lifecycle, and termination.
- `tests/test_logging.py`: image URL log levels and diagnostic wording.

For capture work, start with `on_message()` in `main.py`. For command behavior,
start with `search_image_cmd()` and `_run_command_search()`. Change
`core/image_context.py` only when the task requires different context or
session behavior. For result delivery, start with
`core/result_sender.py::send_search_results()`. For command-wait state and
lifecycle, start with `core/image_wait.py::ImageWaitCoordinator`.

## Image Source Contract

Command searches resolve sources in this order:

1. The first image attached to the command message.
2. The first image in the replied-to message.
3. The next image submitted through an active command wait.

Within an AstrBot `Image`, validate `url` and `file` independently. Preserve
candidate order, prefer HTTP(S) candidates, and try non-HTTP values through the
existing image resolver only when the relevant flow supports uploads.

The message listener captures context URLs in this order:

1. HTTP(S) `Image.url`.
2. HTTP(S) `Image.file`.
3. HTTP(S) `data.url` from guarded raw OneBot image segments.

Apply these rules:

- Keep the generic AstrBot component path primary for non-OneBot adapters.
- Deduplicate identical candidate strings within one event.
- Do not claim content-level or cross-event deduplication.
- Do not persist local temporary paths in image context unless that contract is
  deliberately extended.
- Missing or malformed raw data must not break generic component extraction.
- Reply chains, forwarded nodes, and file-token resolution are separate
  features and require explicit tests.

Access `event.message_obj.raw_message` defensively. It may be absent, an object,
or a mapping. Inspect only mapping-like image segments with mapping-like `data`
and a valid HTTP(S) `data.url`.

Store event metadata through supported APIs:

```python
message_id = str(getattr(event.message_obj, "message_id", "") or "")
sender_id = str(event.get_sender_id() or "")
```

Missing metadata must not prevent image capture.

## Command And Wait Contract

- Validate requested strategies before starting a search or creating a wait.
- Keep `_run_command_search()` as the shared execution path for immediate and
  waited images.
- Send one user-visible search acknowledgement before conversion or provider
  network requests.
- `_run_command_search()` returns `None` after sending successful results and a
  user-facing string for terminal errors.
- A wait is isolated by `(unified_msg_origin, sender_id)` and retains the
  validated strategy selection.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iona-s/astrbot_plugin_imgexploration](https://github.com/iona-s/astrbot_plugin_imgexploration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
