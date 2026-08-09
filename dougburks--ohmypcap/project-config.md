---
trigger: always_on
description: This file contains agent-focused guidance for maintaining SO-CRATES.
---

# AGENTS.md

This file contains agent-focused guidance for maintaining SO-CRATES.

## Updating Vendored Dependencies

SO-CRATES bundles D3 and d3-sankey in `static/` so the application works offline and builds remain deterministic.

### D3

To update to the latest version:

```bash
curl -sL "https://unpkg.com/d3@7/dist/d3.min.js" -o static/d3.min.js
curl -sL "https://unpkg.com/d3-sankey@0.12/dist/d3-sankey.min.js" -o static/d3-sankey.min.js
```

After updating, verify the files load correctly and run the test suite:

```bash
python3 -m unittest discover -v
```

If the copyright year changed, update `static/LICENSE` accordingly.

Check for D3 releases at https://github.com/d3/d3/releases.
Recommended cadence: every 6–12 months, or immediately if a security CVE is announced.

## Backend Architecture

SO-CRATES's backend is split into domain modules. Do not add new logic directly to `socrates.py` — place it in the appropriate module:

| Module | Add here if... |
|---|---|
| `validators.py` | Pure input validation (no HTTP, no I/O). IP/port checks, filename sanitization, URL safety, PCAP magic bytes, ZIP slip prevention. |
| `suricata_analyzer.py` | Anything related to Suricata lifecycle: config setup, rule downloads, spawning subprocesses, processing locks, file extraction. |
| `yara_analyzer.py` | YARA scanning: executable checks, rules download/setup, scanning extracted files, parsing output. |
| `sigma_analyzer.py` | Sigma rule conversion/execution via Zircolite, importing log events, querying Sigma alerts. |
| `file_analyzer.py` | Lightweight file metadata extraction (magic, hashes, strings). |
| `exif_analyzer.py` | EXIF metadata extraction for image/media files. |
| `db.py` | SQLite schema changes, new query functions, index optimization, bulk loading logic. |
| `models.py` | New Suricata event field extraction helpers (parsing JSON fields into typed values). |
| `config.py` | Application-wide constants: size limits, timeouts, thresholds. Adjust here for different deployments. |
| `socrates.py` | Only HTTP handler methods, request/response formatting, and thin orchestration that calls other modules. |

### Handler Conventions

- Use `_send_json(data)` for all JSON responses, `_send_error(code, message)` for errors.
- Extract shared endpoint logic into helper methods on `Handler` (e.g., `_validate_stream_params`).
- Keep `do_GET` and `do_POST` as thin dispatchers via `GET_ROUTES` / `POST_ROUTES` class attributes.

### Frontend Structure

The frontend is split into three files under `static/`:

| File | Content |
|---|---|
| `socrates.html` | HTML shell (one minimal inline theme-restore script in `<head>` to prevent FOUC; otherwise no inline CSS/JS) |
| `static/socrates.css` | All styles |
| `static/socrates.js` | All JavaScript |

`socrates.html` references them via `<link rel="stylesheet" href="static/socrates.css">` and `<script src="static/socrates.js"></script>`.

When updating styles or frontend logic, edit the appropriate split file. Keep `socrates.html` free of inline `<style>` blocks. The single inline `<script>` in `<head>` restores the user's theme before the first paint; keep it minimal and fault-tolerant.

### Theming Conventions

SO-CRATES supports themes via CSS custom properties. The full, current list (name, group, and a short description of each) is the `THEMES` registry in `static/socrates.js` and [Themes](docs/themes.md) — don't duplicate that list here, it goes stale the same way any full list does (see `docs/filtering.md`'s "Column Overlap" note for the same reasoning). As of this writing there are 32: 17 Dark, 5 Light, 10 Fun.

- **Use CSS variables** (`var(--bg-primary)`, `var(--text-primary)`, `var(--accent)`, etc.) instead of hardcoded hex values for all structural/theme colors.
- **Add theme overrides** in the appropriate `[data-theme="<key>"]` block (the theme's registry key in `static/socrates.js`'s `THEMES` object) when a default dark color lacks contrast or does not match the theme's aesthetic.
- **Preserve hardcoded colors** only for functional/data-driven elements (event type colors, severity colors, ASCII transcript direction colors) that must stay consistent across themes.
- **Don't define a variable in every theme block "for completeness" without a real consumer.** `--accent-rgb` and `--filter-bar-bg` were defined identically in all 23 theme blocks but never referenced via `var(--name)` anywhere in CSS/JS/HTML — removed as dead CSS (`test_dead_theme_vars_removed` locks this in). If you add a new per-theme variable, grep for `var(--your-name` before considering it done.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dougburks/ohmypcap](https://github.com/dougburks/ohmypcap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
