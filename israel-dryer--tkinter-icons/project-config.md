---
trigger: always_on
description: Font-based icons for Tkinter and ttkbootstrap. The library renders glyphs from
---

# tkinter-icons — Claude Handoff

## Project overview

Font-based icons for Tkinter and ttkbootstrap. The library renders glyphs from
icon fonts to Tk-compatible images; the icon sets themselves ship as separate
distributions installed via extras.

**The identity shifted, and this matters for every docs decision.** This started as `ttkbootstrap-icons`, intended to be folded into ttkbootstrap. Instead, Bootstrap icons were built *directly* into ttkbootstrap. So this project's audience is now **people on raw tkinter, or people who want an icon set other than Bootstrap** — not "the way to get icons for ttkbootstrap." Renamed to `tkinter-icons` in 5.0.0 to match.

**bootstack was never connected to this project, and no user-facing text may imply otherwise.** Stated by the owner 2026-08-02. Eight places named it alongside ttkbootstrap in the rename rationale — the two READMEs, both changelogs, and four docs pages — which read as though bootstack were part of why the rename happened. All eight were corrected. bootstack is still a legitimate reference for *conventions* — its `docs/conf.py`, its changelog format, its `release_notes.py` — and those mentions below are fine. The line is between "we copied a pattern from a sibling project" and "this project was ever coupled to it."

**Positioning:** one library, sixteen installable icon packs. The packs are
separate PyPI distributions only because each ships its own font; users should
never have to think about that.

```
pip install "tkinter-icons[material]"
from tkinter_icons import MaterialIcon
```

---

## Environment

**Working directory is `D:\Development\ttkbootstrap-icons`** — the *local folder
name is still the old one*. The GitHub repo is `israel-dryer/tkinter-icons` and
every package inside is renamed; only the containing directory lags. Don't
"fix" it mid-session without updating the remote checkout path.

**Which virtualenv works depends on the Windows account you are logged in as, so do not trust a name written down here — check.** There are two, `.venv` and `.venv-home`, created by two different accounts on this machine, and which of them is live flips with the login. Only the one whose base interpreter belongs to the current account runs at all; the other fails with `Access is denied` on the exe itself, not on a file it wants. This file asserted "use `.venv-home`, not `.venv`" for several sessions and was simply wrong by 2026-08-04, when the login had changed — so read `pyvenv.cfg` and match `home =` against the current user rather than believing the last person who wrote it down:

```bash
head -1 .venv/pyvenv.cfg .venv-home/pyvenv.cfg
```

The same ownership split makes git refuse the repository until you run `git config --global --add safe.directory D:/Development/ttkbootstrap-icons`, and it can make `.pytest_cache/` unwritable, which prints a `PytestCacheWarning` on every run and is harmless.

**Whichever one is live needs all eighteen distributions installed**, plus pytest, the docs toolchain, PyInstaller, and fontTools. If the login has just changed, the newly-live venv is probably bare and needs the whole block — it takes a couple of minutes. Substitute the working venv for `.venv` throughout:

```bash
.venv/Scripts/python.exe -m pip install --no-deps -e packages/tkinter-icons
.venv/Scripts/python.exe -m pip install --no-deps $(printf -- '-e %s ' packages/tkinter-icons-*/)
.venv/Scripts/python.exe -m pip install --no-deps -e packages/ttkbootstrap-icons-shim
.venv/Scripts/python.exe -m pip install twine pytest pyinstaller fonttools -r docs/requirements.txt
.venv/Scripts/python.exe -m pytest -q          # 414 passed (1-2 skip, Tk-ordering)
```

Having every pack installed is worth keeping. The docs build reads each pack's live provider for the packs table and the previews, `generate_metrics --all` needs them, and `verify_packages.py --imports` exercises every entry point.

**Check that the base package is actually editable, because it was not on 2026-08-12 and nothing said so.** All sixteen packs were editable and `tkinter_icons` was a plain copy in `site-packages`, so `pytest` was running against a snapshot rather than against `packages/tkinter-icons/src/`. It surfaced only as new tests failing with `AttributeError` on a method plainly present in the source. Nothing had gone stale — the copy was byte-identical to `HEAD` apart from `tools`, which is correctly excluded from wheels — but a green suite in that state is not evidence about the tree. One line answers it, and the fix is the `-e` install above:

```bash
.venv-home/Scripts/python.exe -c "import tkinter_icons,inspect;print(inspect.getfile(tkinter_icons))"
```

It must print a path under `packages/tkinter-icons/src/`. Anything under `site-packages/` means the suite is testing a copy. Worth running whenever results disagree with the source in front of you.

**`ttkbootstrap` is deliberately *not* installed here**, even though the docs have an integration page for it. Verifying those examples means a throwaway venv (`python -m venv`, then `pip install ttkbootstrap` plus the packs the example uses); that keeps a widget library out of the environment the release is verified in. It is not a large cost — the whole ttkbootstrap 2.0 rewrite was checked that way.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [israel-dryer/tkinter-icons](https://github.com/israel-dryer/tkinter-icons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
