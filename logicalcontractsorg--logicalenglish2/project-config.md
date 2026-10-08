---
trigger: always_on
description: You are an expert in both Logical English (LE) and SWI-PROLOG.
---

# Logical English 2 Agent Guidelines

You are an expert in both Logical English (LE) and SWI-PROLOG. 
Refer to `docs/user/reference/language.md` for language syntax and `examples/moreExamples` for inspiring examples
(`examples/README.md` says what each example tree holds; `language/` has one program per feature).
Translators from other systems into LE (migrations) share `le_writer.pl` (Migration IR -> LE
text), `le_migration.pl` (ledger, source tests as scenarios) and `lib/` (shared LE libraries);
see `docs/dev/migration.md`.
Ignore docs/vibeCodingNotes.md, it contains the user's private notes.
Documentation lives in `docs/` by kind (`docs/README.md`): `docs/user/` is the
published user documentation (its table of contents, `docs/user/nav.json`,
builds the Help menu, the landing page and the viewer), `docs/dev/` is for
developers, `docs/project/` holds plans, papers and archived documents. A new
user document goes under `docs/user/` and into `nav.json`.

**How documentation is written.** Documents, and every piece of text the
software shows a reader (menu tips, explanations of an answer, the
assistants' replies), are written for a reader with no technical training:
name the thing rather than writing *it*/*this*/*they*, keep computing jargon
out or explain it in the same sentence, one idea per sentence, spell out an
abbreviation at first use. The rule in full, with its examples, is the "How
documents are written" section of `docs/README.md` (kept identical in LE2 and
LPS2). The names of Logical English's own ideas — a template, a scenario, a
fluent — are part of what is being taught: use them, and define each at first
use.

If /lps2 exists, it contains the Logic Production Systems repository, which depends on ours.

## Videos
A request to "build a standard video for X" (a feature or an example of this
system) follows `/lpsPlus/docs/sales/videos/STANDARD_VIDEO.md`: a Playwright
script that drives the editor, an opening slide and a concluding slide with
the Logical Contracts logo, calm narration without hype spoken by ElevenLabs
voice `0HN93OO0QQQR6Vh2gSAe` (key in `.credentials`), about four minutes,
built into an `.mp4` with `ffmpeg` by `vlib.cjs` in that folder.

## Build, Lint, and Test
In what follows, SWIPL refers to the `./myswipl.sh` wrapper at the repo root. It
selects the SWI-Prolog interpreter to use: the `$SWIPL` env var if set, otherwise
the macOS app bundle (`/Applications/SWI-Prolog10.0.0-1.app/Contents/MacOS/swipl`)
when present, otherwise `swipl` on PATH. You can also just call `swipl` directly if
it is on your PATH.

### Run everything: `testing/run_tests.sh`
The `testing/run_tests.sh` runner runs all suites and aggregates a single pass/fail
(non-zero exit if any suite that ran failed). It always runs from the repo root, so
you can call it from anywhere. Use it as the default check:
- `testing/run_tests.sh` — all suites (unit + LE examples + Playwright e2e).
- `testing/run_tests.sh --no-e2e` — fast path: Prolog unit + LE examples only.
- `testing/run_tests.sh unit | le | e2e` — run a single suite (space-separated subsets allowed).
- `testing/run_tests.sh --with-extensions` — LE examples INCLUDING the trees that
  need the proprietary `le_extensions.pl` (see below). Default is core only.
- Set `SWIPL=...` to choose the interpreter; set `CI=1` to make a missing e2e setup
  a failure instead of a skip.

**Core vs core + extensions.** The LE example suite has two variants:
- **core** (the default) — the programs that run on this repository alone. This is
  the suite a clean checkout can make green, and the one CI should gate on.
- **all** (`--with-extensions`) — core plus the example trees that need
  `le_extensions.pl`, a symlink into a sibling repository. Those programs use
  constructs the core grammar does not implement, so without the extensions
  installed they do not merely fail, they cannot be parsed — and their failures say
  nothing about core LE.

The exclusion is a hardwired table, `extension_dependent_path_fragment/1` in
`le_kbs.pl` (currently the `insureLE2/`, `InsurLE2/` and `lpsPlus/` trees, and
`language/extensions/`). Add a row there when a new extension-dependent example
tree appears; nothing else needs to change. (Embedded `prolog` goals are core
LE, language.md §15.6: a program that only uses them belongs to the core suite.)

Each variant writes its own committed status snapshot and never touches the other's
(`suite_status_file/2`): core → `testSuiteCoreStatus.txt`, all → `testSuiteStatus.txt`.
Both are tracked; a repository without `le_extensions.pl` should ignore
`testSuiteStatus.txt`, which it cannot reproduce. **After a change, regenerate the
status file of whichever suite(s) you ran** — do not hand-edit either file, and do
not leave one stale while updating the other.

The three suites it wraps (also runnable directly):
- **Prolog unit tests (plunit):** `SWIPL -q -g run_tests -t halt testing/test_session_reaper.pl`
  — fast, no server. Lives in `testing/test_*.pl`; add new ones there with
  `:- begin_tests(Name). ... :- end_tests(Name).` and `testing/run_tests.sh unit` picks them up.
  Use `:- use_module('../le_kbs').` (the module is one level up from `testing/`).
- **Logical English example tests:** `SWIPL -g "use_module(le_kbs), runTests, halt."`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LogicalContractsOrg/LogicalEnglish2](https://github.com/LogicalContractsOrg/LogicalEnglish2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
