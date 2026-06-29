---
trigger: always_on
description: bootstack is a batteries-included Python desktop UI framework. It is **not**
---

# bootstack — Claude Handoff

## Project overview

bootstack is a batteries-included Python desktop UI framework. It is **not**
advertised as a Tkinter wrapper — the goal is to abstract Tkinter away entirely
so that Tkinter's warts, naming conventions, and legacy API are invisible to the
user. Widget names, arguments, methods, and events are designed for modern Python
and ease of use, not compatibility with the raw tk/ttk surface.

**Design philosophy:** Opinionated and configurable within a reasonable range.
Go from nothing to something fast. The user should never need to `import tkinter`.

**Working directory:** `D:\Development\bootstack`
**Branch strategy:** `feat/*` branches off `main`. PRs go `feat/*` → `main`.

---

## Recently completed (all merged to `main`)

Pointers only — these shipped; rationale, detail, and gotchas live in the linked
memories and git history.

- **0.1.2 PATCH SHIPPED — dropdown/context menus dismiss on window move (PR #345;
  2026-06-25).** `bootstack 0.1.2` is on **PyPI** + tagged **`v0.1.2`**. **Bug
  (user-reported, Win10):** an open toolbar `add_menu` dropdown (and any Win/Linux
  `ContextMenu`/`Select`/`MenuButton` popup) stayed pinned at its old screen
  position when you dragged/resized/minimized the window — it "hung in the air."
  The `_ToplevelContextMenu` (overrideredirect Toplevel backend) only dismissed on
  **outside mouse clicks** bound on the owning window; dragging the native title
  bar fires no click, only `<Configure>`/`<Unmap>` on the toplevel, which nothing
  listened for. **Fix:** while shown, also bind the binding-root's
  `<Configure>`/`<Unmap>` → new `_on_window_change` method dismisses (guarded to
  `event.widget is the toplevel` so a child relayout doesn't close it); torn down
  with the existing outside-click cleanup. Shared backend → fixes ALL Win/Linux
  popups at once. Verified decorated AND undecorated (both move via `wm geometry`
  → `<Configure>`); macOS `_NativeContextMenu` untouched (native menu self-dismisses).
  **NB this is the window-MOVE bug — DISTINCT from #207** (the `'break'`-target
  dismiss case, still OPEN; I confirmed a `'break'` toolbar widget already dismisses
  here). Tests in `test_toolbar_menu.py` (window-change dismissal + binding
  teardown). **Process:** empirical self-driving repro (move window, assert
  `winfo_viewable()`) was decisive — static reading missed that no click fires on a
  title-bar drag. Confirmed Py 3.12.10's `unbind(seq, funcid)` removes only that
  funcid (not the unbind-wipes-all of older Tkinter), so binding `<Configure>` on
  the app toplevel is safe. **CHANGELOG gotcha (fixed):** `## [0.1.1]` was a
  bracketed link with NO `[0.1.1]:` definition (dead link); added `[0.1.1]:` +
  `[0.1.2]:` defs. **Release-notes gotcha:** `release.yml` extracts ONLY the
  `## [x]` section, so the bottom link-defs are excluded → `[0.1.2]` renders as
  literal brackets in the GitHub Release body (see Next-up for the fix).
- **0.1.1 PATCH SHIPPED — `pygments` declared as a runtime dependency (PR #344;
  2026-06-24).** `bootstack 0.1.1` is on **PyPI** and tagged **`v0.1.1`** (GitHub
  Release, notes from the CHANGELOG `## [0.1.1]` section). **Bug:** `CodeEditor`
  hard-imports `pygments` (via `_try_install_highlighter`) for syntax highlighting,
  but it was declared as a dependency **nowhere** in `pyproject.toml` — so a clean
  `pip install bootstack` (no incidental pygments) crashed on any
  `CodeEditor(language=...)`, including the bundled `bootstack` demo's editing page
  (`ModuleNotFoundError: No module named 'pygments'`). Fix = add `pygments>=2.15`
  to `[project].dependencies` (hard dep, per the maintainer — pure-Python, no
  transitive deps, and highlighting is core to a top-level `bs.*` widget; matches
  batteries-included). **Full dependency audit done while here — `pygments` was the
  ONLY gap:** every optional extra (`viz`/matplotlib, `parquet`/pyarrow,
  `excel`/xlsxwriter, `hdf5`/pandas+tables, `viz-seaborn`/seaborn) is lazy-imported
  inside functions and gated by `_require(...)`/`_require_matplotlib` (so `import
  bootstack` + core widget construction never crash); `PyInstaller` is a
  packaging-time tool (`try/except` in the CLI); `tomli` is a dead fallback
  (`tomllib` is stdlib on `requires-python >=3.12`); `cycler` is guarded + ships
  with matplotlib. **Release process note:** `bump-my-version` was NOT on PATH /
  importable in any reachable interpreter this session (the maintainer runs it from
  a manually-activated venv); replicated its config by hand — bumped BOTH `version`
  (line 7) and `[tool.bumpversion] current_version` + commit `Release 0.1.1` +
  annotated tag `v0.1.1` (`message`/`tag_message` = `Release {new_version}`).
- **0.1.0 STABLE SHIPPED — ship gate + theme-repaint unification + accent contrast
  (2026-06-24).** `bootstack 0.1.0` is on **PyPI** (`pip install bootstack`, no
  `--pre`) and tagged **`v0.1.0`** (stable GitHub Release, `prerelease=false`).
  Verified on **Windows AND macOS**. **#149 ship gate (PR #335 — MERGED):**
  public-surface audit + lock + CHANGELOG + Release Notes + Roadmap page; the
  folded items shipped in **PR #334** (`text=<Signal>` → `TypeError` guard across

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [israel-dryer/bootstack](https://github.com/israel-dryer/bootstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
