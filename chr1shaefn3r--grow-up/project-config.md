---
trigger: always_on
description: Project knowledge for an agent working in this repo. `README.md` is the user-facing
---

# Notes for Claude Code

Project knowledge for an agent working in this repo. `README.md` is the user-facing
documentation and is current — read it first for what the tool does. This file covers
what the code cannot tell you: the invariants, the mistakes already made, and the limits
of what you are able to verify.

## Start here

`grow-up` turns the photos of one tagged person in an [Immich](https://immich.app)
library into an eye-aligned face timelapse. Python, 14 modules, no framework.

Every module opens with a docstring stating its design rationale. Read the one for the
area you are touching before changing it — most "why is it like this?" questions are
answered there.

```bash
pytest                    # the whole suite, a few seconds, no network needed
grow-up --help            # the CLI surface
```

## Ground rules

**Credentials are environment-only.** `IMMICH_URL` and `IMMICH_API_KEY`, read in
`config.credentials()`. They never go in `config.toml`, which is why that file is safe to
commit. You will not be given a host or a key — do not plan work whose only verification
is a live library.

**The test suite must keep running with nothing installed.** No network, no Immich, no
model file, no ffmpeg, and no mediapipe or opencv. `.github/workflows/tests.yml` installs
only `pytest numpy httpx`, and that is the contract: a test that imports the heavy stack
ungated will pass locally and break CI on three Python versions. Gate pixel-touching
tests with `pytest.importorskip("cv2")` (see `tests/test_framing.py`), and node-dependent
ones with `pytest.mark.skipif(shutil.which("node") is None, ...)` (see
`tests/test_tuner.py`).

Keeping maths out of the heavy dependencies is a deliberate design choice, not an
accident: `metrics.py` and the transform maths in `align.py` are plain numpy so they stay
testable in a bare environment. Preserve that when adding to them.

**Since 1.0.0, an existing `config.toml` must never break.** This rule reversed at the
release and the old text is preserved here because an agent that finds only the new rule
will not understand why `REMOVED_KEYS` exists. Before 1.0.0 the repo had one user, so a
renamed setting was simply renamed and the old name went into `config.REMOVED_KEYS` to
fail loudly. That table is now **frozen** at its pre-1.0 contents — everything in it
predates the release, so no published config can contain one. Do not add to it.

New settings arrive with a default and a fallback for their absence. `config.sources()`
is the worked example: a file with no `[[immich.sources]]` yields exactly the single
source 1.0.0 assumed, on the same two environment variables, and
`TestTheReleasedConfigStillWorks` in `tests/test_sources.py` is that promise written
down. Schema changes go through `db.ADDED_COLUMNS` so a database from an earlier version
migrates in place.

Settings must also not rot. Two tests in `tests/test_config_example.py` hold the example
file honest: every `[analyze]` key must be one the code actually reads, and every key in
the file must carry a comment.

## Traps

Each of these is a mistake that was already made here, or one the code is shaped
specifically to avoid. They all look like improvements.

| Do not | Why | Guarded by |
|---|---|---|
| Smooth or average `(tx, ty, angle, scale)` across frames | These live in each source photo's own pixel space; across one real library `tx` ranged −962 to −5458. This shipped, and put faces 848–1045px off target — outside the frame entirely. Every single-transform test passed the whole time. | `test_frames_are_solved_independently`, `test_every_eye_stays_inside_the_output_frame` (`tests/test_align.py`) |
| Thread `--since` past the `index` stage | Selection, alignment and encoding must see the whole corpus. Constraining them yields a timelapse of only the last week — a failure that produces a plausible-looking video. | `test_selection_spans_the_whole_corpus_not_just_recent_assets` (`tests/test_select.py`) |
| Switch the sync watermark to `takenAfter` | Photos imported late but taken early — restores, scans, a phone offline for a month — fall permanently behind it. It is `updatedAfter`, stores the run's *start* minus 60s, and commits only on success. | `tests/test_watermark.py` |
| Re-implement a filter rule in the page's JavaScript | `metrics.RULES` is one serialized table interpreted by both Python and `rejects.html`. Two spellings of a rule drift, and a tuner that disagrees with the pipeline is worse than no tuner. | `tests/test_tuner.py` runs the page's own filter under node against the Python one |
| Assume SMT when counting cores | Apple Silicon has no hyperthreading, so halving the logical count gives an M1 Pro 4 workers instead of 8. `analyze.physical_cores()` detects properly per platform. | `tests/test_cores.py` |
| Log `type(exc).__name__` for a failed request | Discarding the HTTP status cost an entire debugging round on a real library. `ImmichHTTPError` carries status, path and body. | `test_keeps_the_status_code` (`tests/test_client_errors.py`) |
| Clear a progress line by padding with spaces | Leaves trailing whitespace in the terminal buffer. Use the ANSI erase-to-end-of-line already in `progress.py`. | `test_summary_line_has_no_trailing_whitespace` (`tests/test_progress.py`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chr1shaefn3r/grow-up](https://github.com/chr1shaefn3r/grow-up) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
