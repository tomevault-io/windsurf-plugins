---
trigger: always_on
description: FastAPI (async) + aiosqlite | Jinja2 | HTMX 2 + Alpine 3 | Tailwind v4 | uv + hatchling | pytest + Node
---

# Focus — Agent Reference

## Stack

FastAPI (async) + aiosqlite | Jinja2 | HTMX 2 + Alpine 3 | Tailwind v4 | uv + hatchling | pytest + Node

## Start / Test

- Start: `./start.sh` — vendor sync → tailwind build → `uv run main.py`
- Test: `./test.sh` — `uv run pytest`

## Structure

```
main.py                  # FastAPI app entry
focus/                   # Backend package
  core/                  # DB init, models, utils, macros, segments, media, tracked_fields
  db/                    # CRUD per domain (characters, chats, personas, presets, providers, etc.)
  providers/             # LLM providers (openai_compat, openrouter, deepseek, moonshot, google_*)
  routers/               # Route handlers (pages, chats, stream, presets, providers, tools, backup, etc.)
  tools/                 # Builtin + external tool system, executor, provider adapter
  crud.py, exchange.py, prompt_chain.py, backup.py
templates/               # Full-page Jinja2 templates
partials/                # HTMX fragments (chat/, modals/, personas/, presets/)
static/
  css/                   # Custom CSS modules
  js/
    core/                # state-manager, actions, generation-session, chat-controls, api-paths
    messages/            # Streaming, rendering, editing, pruning
    modals/              # Config forms, editors
    ui/                  # Theme, scroll, lightbox, notifications, etc.
    features/            # char-editor, backup-manager
    utils/               # Small helpers
    vendor/              # HTMX, Alpine, marked, purify, etc. (synced by vendor-sync.py)
assets/                  # User uploads (attachments, characters, personas, tool configs)
data/                    # focus.db, backups/
tools/                   # External tool JSON configs (samples/ shipped)
tests/                   # api/, units/, frontend/ (JS via Node)
```

## Critical patterns

### State flow — `static/js/core/state-manager.js`

Single source of truth for `character_id`, `persona_id`, `preset_id`, `provider_id`, `provider_type`.
- Set via `.setCharacter(id)` / `.setPersona(id)` / `.setPreset(id)` / `.setProvider(id, type)` — auto-persists to DB (chat fields) or localStorage (provider). All accept null.
- Read via `.get('key')` or `.getAll()`.
- React via `.on('event', fn)` — callback gets `{ prev, value }`. Alpine: listen `@event.window`.

### Message identity — `static/js/messages/message-identity.js`

Single source of truth for message id translation. A message is identified by its **bare** id, carried on the node as `data-message-id`. Two derived forms are never written by hand:
- DOM node `#message-<id>` → `MessageIdentity.domId(id)` / `MessageIdentity.node(id)`
- pruned stub `.message-placeholder[data-msg-id="<id>"]` → `MessageIdentity.placeholder(id, root)`

The pruner's `_pruned` map is keyed by bare id (`MessageIdentity.bare`); `_forgetPruned(id)` drops a stale entry. All id reads/writes in `message-pruner.js`, `message-refresh.js`, `stream-events.js`, and `core/actions.js` go through this module.

### Action dispatch — `static/js/core/actions.js`

`data-action="fnName"` on elements → delegated `document` listeners (click, submit, change, input).
- **Form guard**: only `submit` events trigger actions on forms. Never put `data-action` on `<form>`.
- Message toolbar buttons read context from `el.closest('.message')` data-* attributes (no inline JS).

### Modals (`static/js/ui/modal.js`)

- Single lifecycle controller: `ModalController.open(id)` / `close(id, opts)` / `closeTop()` / `isOpen(id)` / `isDirty(id)` / `setDirty(id, bool)` / `refresh(id)` / `onOpen(id, fn)`. Globals `openModal(id)` / `closeModal(id, opts)` are thin wrappers.
- Template: `modal-shell.html` macro → `{% call modal_shell('id', 'Title') %}...{% endcall %}`
- Show/hide: toggle `hidden` class (not `style.display`) — done by the controller, never manually.
- z-index layers: base → sub → editors → overlays → confirm (scale by 10× from 50)
- **Dirty tracking is opt-in via overlay attributes** (rendered by `modal_shell` params `dirty_fields`, `dirty_label`): snapshot is captured automatically on open (fields created before/at open are baselined; fields added later are lazily baselined). `[data-dirty-hint]` gets `.hidden`, `[data-dirty-save]` gets `.disabled` + `.opacity-50`. State changes dispatch `dirty-changed` on `window` (`{id, dirty}`) — used by the rename-preset modal's Alpine UI.
- Closing a dirty modal prompts via confirm; **save handlers must call `closeModal(id, {discard: true})`** (never `captureDirty`-style resets). Use `ModalController.setDirty(id, true)` for non-field changes (e.g. attachment add/delete).
- ESC closes: confirm modal (as Cancel) → topmost modal → lightbox. `option-selected` / `custom-select:set` trigger recompute of open modals.
- Overlays inside htmx-swapped bodies (providers modal) re-register automatically on open.

### Toast system (`static/js/ui/notifications.js`)

- One stack: `#toast-container` in `base.html` (fixed top-center, `z-index: var(--z-max)`, above all modals).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ganon3264/focus](https://github.com/ganon3264/focus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
