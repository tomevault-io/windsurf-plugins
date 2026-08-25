---
trigger: always_on
description: Working notes for AI coding agents. Read this before touching the code.
---

# AGENTS.md — Comfy Model Fetcher

Working notes for AI coding agents. Read this before touching the code.

## What this is

A ComfyUI extension that **registers no nodes**. It adds a top-bar button, a popup, and HTTP
routes under `/cf_mf`. Everything is wrapped defensively: an error here must never stop ComfyUI
from starting (see `__init__.py`).

## Layout

```
__init__.py          route registration + empty V3 extension shim (no nodes)
fetcher/
  notes_parser.py    pure parsing — no ComfyUI import, no I/O. Keep it that way.
  scanner.py         folder_paths → categories, disk index, classification, relink target
  remote.py          remote size via HEAD (cached)
  downloader.py      queue + single worker thread, .part writes, websocket progress
  routes.py          the /cf_mf/* API
  hf_token.py        token storage/validation + the HuggingFace-only auth header
web/                 frontend, plain ES modules — no build step, no bundler, no npm
  main.js            button, badge, note collection from the graph
  popup.js           the panel and all its state
  relink.js          reads/rewrites loader widgets in the open graph
docs/                README screenshots
tests/               see below — run them
```

## Invariants — do not break these

1. **The token goes to HuggingFace only.** `hf_token.auth_headers(url)` is the *single* place
   that builds an `Authorization` header, and it returns `{}` for any non-HF host. Model URLs
   come from workflow notes, which are untrusted input — a shared workflow must never be able
   to point the downloader at an attacker's server *with your credentials attached*. Never
   reintroduce a global `_auth_headers()`.
2. **Download destinations stay under the models directory.** `scanner.resolve_category()`
   strips `..` and absolute segments; `routes._safe_join()` re-checks containment with
   `scanner.is_under()`. A client-supplied `category` is untrusted. Both layers must stay.
3. **`base_dir` must be a folder already registered for that category** (main + extra paths).
   Never accept an arbitrary path from the client. The allowed set is `scanner.dest_dirs()` —
   the *same* function that builds the UI menu, so what is not offered is not accepted.
4. **Blocking I/O never runs on the aiohttp event loop.** `os.walk` over model directories
   (often network shares) and HEAD requests both go through `asyncio.to_thread`. Blocking the
   loop freezes all of ComfyUI, including websocket progress for the very downloads this
   plugin started.
5. **"Installed" means "the loader resolves the plain filename"** — i.e. the file is at the
   root of *any* of the category's roots, not only `target_dir`. Downloading into an extra
   path is a legitimate choice, and ComfyUI searches every registered root. `_at_dir_root()`
   decides this, and `relink_target()` uses the same predicate on purpose: a row labelled
   *Likely duplicate* must always have something to relink, or it is a dead end (status says
   "you have it", the only button says "download it again").
6. **A row is a grid of two independent lines**, `Link to … Relink` and `Save to … Download`
   (`.cf-mf-ops`, one `.cf-mf-opline` each, `display: contents` so both share the columns).
   `row.action` stays the download slot alone — progress, cancel, error and "✓ Installed" all
   land there — and `row.actionLink` holds the relink button. `flashRow()` takes the slot as an
   argument for the same reason: a message shown next to the wrong button blames it.
7. **Relink state is read from the graph, never cached.** `linkState()` is called on every
   render. A cached "already linked" flag would lie after an undo or a workflow switch. Which
   copy the row points at follows the same rule: `relinkPick()` prefers the user's choice, then
   whichever copy the graph already uses, and only then the server's automatic pick — a popup
   reopened on a relinked workflow must not claim the row is unlinked because it defaulted
   elsewhere. `relink_target()` stays the default — among same-size copies it takes the one on
   the earliest of `cat.all_dirs`, i.e. `folder_paths` order, which `is_default: true` in
   `extra_model_paths.yaml` puts the user's preferred install at the head of; alphabetical
   order only breaks ties within a root — and the invariant test checks it is always one of
   the copies the menu offers.
8. **Popup preferences are keyed per workflow.** Several workflows are open at once; state must
   not leak between tabs. See `currentWorkflowKey()` and its epoch fallback.
9. **Model ids must be unique.** `parse_notes()` disambiguates two different URLs that resolve
   to the same `category/filename`. The id is both the UI row key and the server job id.
10. **A `.part` file is only ever appended to for the same source URL.** `_part_path()` mixes a
   hash of the URL into the name; resume decisions rely on it. Never go back to a fixed
   `<dest>.part`, or two models sharing a filename will be spliced into one corrupt file.
11. **`huggingface_hub` is the HF protocol authority.** File sizes (`get_hf_file_metadata`) and
   token validation (`HfApi.whoami`) delegate to it; both fall back to a direct HTTP call only
   when the import fails. Do not reintroduce a hand-rolled HF HEAD dance — a wrong size flips a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KoLt-Real/ComfyUI-ModelFetcher](https://github.com/KoLt-Real/ComfyUI-ModelFetcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
