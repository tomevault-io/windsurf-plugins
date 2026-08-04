---
trigger: always_on
description: This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.
---

# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

- Add durable project-specific notes here as they are discovered through real work.

## Architecture invariants

- **Disk is the source of truth.** Every mutation rewrites `documents/<id>/info.yaml`
  first, then re-syncs the SQLite index (rebuildable). Write through the helpers in
  `soap/library.py` (`save_document`, `set_review_status`, `set_read_status`,
  `edit_document`, `delete_document`), never the DB directly. Those helpers validate
  document IDs and file references at the library boundary, and metadata YAML
  replacement is atomic; `resolve_file_ref_path` is also the TUI open boundary.
- **Library paths are owner-private; shell exports are quoted.** Anything a
  library owns is forced to `0700`/`0600` after it is created or copied via
  `soap/permissions.py:make_private` (source files keep their original mode
  only until they land inside the library) — never rely on umask or an atomic
  rename to set the mode. Generated shell startup code must go through
  `soap/shell.py:export_line`, which shell-quotes `SOAP_DIR` per shell
  (POSIX/`shlex` for bash/zsh/unknown-fallback, single-quote escaping for
  fish) and rejects NUL bytes and the reserved `# >>> soap >>>` block markers;
  never `f"...{path}..."` a path straight into a config line. Regression
  coverage: `tests/test_security.py`.
- **Link adds download the PDF.** A URL/bare-arXiv-id add resolves metadata *and*
  best-effort downloads the paper's PDF (`soap/ingest/download.py:download_pdf`,
  driven from `soap/ingest/url.py:resolve_url(download=...)`): arXiv's canonical
  `/pdf/<id>.pdf`, a direct `.pdf` URL, or an open-access PDF a DOI exposes
  (Crossref `link`, else the doi.org redirect if it lands on a real PDF). It streams
  to a temp file and keeps it only if the first bytes are the `%PDF` magic marker —
  content-type is advisory, so even `application/pdf` must start with `%PDF` — caps
  size, and hands the temp to `_add_body` which stores+attaches it via the normal
  local-file path (sha256, `attach_file`); the temp is always cleaned up in
  `_add_inner`'s finally. Download is gated `fetch and not dry_run`; a failed/paywalled
  download degrades to metadata-only with a warning — never crashes. The PDF is still
  never parsed (the abstract comes from the metadata API, not the file).
- **Outbound HTTP is SSRF-hardened.** Both metadata fetches (`soap/ingest/fetch.py`)
  and the PDF download (`soap/ingest/download.py`) follow redirects manually one hop at
  a time and route every hop — including the explicit user URL — through
  `soap/ingest/network.py:validate_url`, which rejects non-HTTP(S) schemes, embedded
  credentials, and any host that is or resolves to a loopback/private/link-local/reserved
  address (`MAX_REDIRECTS` cap). Response bodies are size-bounded before a parser sees
  them (`MAX_METADATA_BYTES` for metadata, `MAX_PDF_BYTES` for the PDF). Parsers
  (`parse_crossref`/`parse_arxiv`/`parse_openlibrary`) validate payload shape at the
  boundary and return `None` on malformed/wrong-type provider data, so `add` degrades
  with a warning instead of crashing. Regression-tested in `tests/test_ingest.py` — keep
  these seams injectable and never weaken the address checks. `_bounded_get`
  reconstructs a fresh `httpx.Response` from the *decoded* streamed body, so it
  first strips the provider's now-inaccurate `Content-Encoding`/`Content-Length`
  headers — leaving them makes httpx re-run the gzip decoder on plain bytes and
  raise `DecodingError`, silently turning every gzipped metadata lookup into a
  miss (Open Library/Crossref gzip by default).
- **CLI and TUI share the review core.** The interactive walk lives in
  `soap/library.py:review_inbox` (IO fully injected: `render`/`ask_action`/
  `confirm_delete`/`report`/`prompt_field`) so it is unit-tested without a terminal
  (`tests/test_inbox_review.py`). CLI (`soap/cli/inbox.py`) and TUI
  (`soap/tui/review.py`) are thin shims — keep their review semantics consistent.
- **Inline correction walk:** `soap/library.py:prompt_fields` walks the core fields
  (`CORE_REVIEW_FIELDS`: title/authors/year/type/venue), prefilled, Enter-keeps /
  type-overrides. It **pins the citekey/id** on a review-edit (never renames the
  folder); only a brand-new `add()` derives a fresh key. Shared by the CLI `[c]orrect`
  action, the TUI review form, the TUI browser's `E` in-app edit form
  (`soap/tui/edit.py`), and `soap add --confirm`.
- **Browser edit/delete reuse the review core.** The main browser
  (`soap/tui/app.py`) exposes `E` (in-app core-field edit → `EditScreen` in
  `soap/tui/edit.py`, which drives `prompt_fields`+`save_document`, id pinned) and
  `d` (delete → `ConfirmDeleteScreen` in `soap/tui/confirm.py` → `delete_document`).
  `e` stays the full-YAML `$EDITOR` power option. Both go through the library helpers
  (never the DB); delete is confirm-gated. Pilot coverage:
  `tests/test_tui_browser_edit_delete.py`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GhifariArsa/soap](https://github.com/GhifariArsa/soap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
