---
trigger: always_on
description: Project guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Project guidance for Claude Code when working in this repository.

## Releasing a new version

The version number lives in **four** places. All must be kept in sync, otherwise the GUI, the Python package metadata, and the Windows EXE properties will disagree.

### 1. Bump the version number

Update **all four** of these files (search for the current version, e.g. `3.1.2`, and replace with the new one, e.g. `3.1.3`):

| File | Line(s) | What to change |
|---|---|---|
| `globalVar.py` | ~103 | `SCIENTOPY_VERSION = "X.Y.Z"` — runtime version shown in the GUI |
| `pyproject.toml` | ~7 | `version = "X.Y.Z"` — Python package metadata |
| `version_info.py` | ~28-29 | `filevers=(X, Y, Z, 0)` and `prodvers=(X, Y, Z, 0)` — Windows EXE binary metadata (tuple form) |
| `version_info.py` | ~45, ~50 | `StringStruct(u'FileVersion', u'X.Y.Z')` and `StringStruct(u'ProductVersion', u'X.Y.Z')` — Windows EXE string metadata |

> Quick check that nothing was missed:
> ```bash
> grep -rn "OLD_VERSION" --include="*.py" --include="*.toml" .
> ```
> Should return zero hits (excluding `doc/release_notes/`).

### 2. Create the release notes

Create a new file `doc/release_notes/vX.Y.Z.md` following the existing format (`doc/release_notes/v3.1.0.md` and `v3.1.2.md` are good templates).

Suggested structure:

```markdown
# ScientoPy vX.Y.Z — Release Notes

## Highlights

One short paragraph describing the headline change.

## <Section per category>

- bullet points...

---

**Full changelog:** [`vPREV...vX.Y.Z`](https://github.com/jpruiz84/ScientoPy/compare/vPREV...vX.Y.Z)
```

Common section headings used in past releases:
- `Performance`
- `Bug Fixes`
- `New Features`
- `Code Signing (Windows)`
- `Infrastructure`

Keep wording user-facing — describe what changed and why it matters to someone running the app, not implementation details.

### 3. Commit, tag, push

```bash
git add globalVar.py pyproject.toml version_info.py doc/release_notes/vX.Y.Z.md
git commit -m "Bump version to X.Y.Z and add release notes"
git push origin master

git tag vX.Y.Z
git push origin vX.Y.Z
```

The `v*` tag push triggers `.github/workflows/build-release.yml`, which:
1. Runs unit + behavioral tests on Linux, macOS, and Windows.
2. Builds PyInstaller bundles for Linux x64, macOS arm64, and Windows x64.
3. Submits the Windows artifact to SignPath for Authenticode signing (only on `v*` tags).
4. Publishes a GitHub Release with all three platform artifacts.

### 4. Verify the release

After the workflow finishes:

- Check the **Actions** tab → all jobs green.
- Open the new release at **Releases** → confirm three assets attached (`ScientoPy-windows-x64.zip`, `ScientoPy-linux-x64.tar.gz`, `ScientoPy-macos-arm64.zip`).
- Download `ScientoPy-windows-x64.zip` → extract → on Windows, right-click `ScientoPyGui.exe` → **Properties** → **Digital Signatures**: signature must be **Valid** with the correct publisher.
- Optional PowerShell check:
  ```powershell
  Get-AuthenticodeSignature .\ScientoPyGui.exe | Format-List
  ```

## Testing

The project has two test suites — both run in CI on every PR and push (`.github/workflows/build-release.yml`, the `test` job, matrix: Linux + macOS + Windows). Run them locally before opening a PR or cutting a release.

### Test suites at a glance

| Suite | File | Type | Coverage |
|---|---|---|---|
| **Unit tests** | `tests/test_unit.py` | pytest | Pure-function logic: `globalVar` constants, `paperUtils` dedup, source counting, open-access normalization, error classes |
| **Behavioral tests** | `tests/run_behavioral_tests.py` | Custom runner | End-to-end pipeline: generates synthetic Scopus + WoS data via `tests/generate_test_data.py`, runs `preProcess.py` and `scientoPy.py`, asserts outputs against the spec in `doc/behavioral_test_spec.md` |

### Running the tests locally

```bash
# Unit tests (~2 s)
python -m pytest tests/test_unit.py -v

# Behavioral tests (~30-60 s — exercises the full pipeline)
python tests/run_behavioral_tests.py
```

Expected counts (as of v3.1.2): **19 unit tests** and **81 behavioral checks**. If counts differ unexpectedly, investigate before assuming tests are passing.

### Which tests to run after a change

Match the change category to the suite — but **always run both before pushing a release tag**.

| You changed... | Run |
|---|---|
| A pure helper in `paperUtils.py`, `globalVar.py`, error classes | Unit tests first; behavioral if the helper feeds into the pipeline |
| Preprocessing logic (`preProcess.py`, `PreProcessClass.py`, `paperIO.py`, dedup, field mapping, OA normalization, country/institution extraction) | **Both suites** — preprocessing is covered end-to-end by behavioral tests |
| Analysis logic (`scientoPy.py`, `ScientoPyClass.py`, criterion handling, year filtering, custom topics, `--saveExtended`, previous-results chaining) | Behavioral suite (Stage 2 covers all `-c` criteria, year ranges, custom topics, dataBase grouping) |
| Export / CLI tools (`exportPapers.py`, `generateBibtex.py`) | Behavioral suite (Stage 3 covers `exportPapers` round-trip) |
| GUI code only (`ScientoPyGui.py`, splash screen, graph theming, dialogs) | Unit tests + **manual GUI smoke test** — there is no automated GUI test harness; launch the app and exercise the affected feature |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jpruiz84/ScientoPy](https://github.com/jpruiz84/ScientoPy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
