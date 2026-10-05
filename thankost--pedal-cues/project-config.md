---
trigger: always_on
description: JUCE plugin (VST3 / AU on macOS / LV2 on Linux / Standalone; Windows, macOS (universal: Intel from 10.13, Apple Silicon from 11; `CMAKE_OSX_DEPLOYMENT_TARGET` 10.13 and the pkg's `os-version min`) and Linux; tested in Reaper, DAW-neutral wording elsewhere) that turns Quad Cortex, Kemper and Whammy V / DT MIDI changes, and tiles for any other MIDI device (custom MIDI devices, beta), into drag-and-drop tiles. The author is Thanasis Kostopoulos (GitHub `thankost`). The repo is public: https://githu
---

# PedalCues: notes for Claude

JUCE plugin (VST3 / AU on macOS / LV2 on Linux / Standalone; Windows, macOS (universal: Intel from 10.13, Apple Silicon from 11; `CMAKE_OSX_DEPLOYMENT_TARGET` 10.13 and the pkg's `os-version min`) and Linux; tested in Reaper, DAW-neutral wording elsewhere) that turns Quad Cortex, Kemper and Whammy V / DT MIDI changes, and tiles for any other MIDI device (custom MIDI devices, beta), into drag-and-drop tiles. The author is Thanasis Kostopoulos (GitHub `thankost`). The repo is public: https://github.com/thankost/pedal-cues. The download site is https://thankost.github.io/pedal-cues/, served from `docs/` on `main`.

## Build and test

```sh
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DPEDALCUES_BUILD_DOCS=ON   # once
cmake --build build                                                       # also installs VST3/AU to ~/Library/Audio/Plug-Ins
build/PedalCuesTests_artefacts/Release/PedalCuesTests                     # unit tests, must print "All tests passed"
build/PedalCuesTests_artefacts/Release/PedalCuesTests --online            # optional: real GitHub update check
build/DocShots_artefacts/Release/DocShots docs/images                     # regenerate every screenshot and diagram
```

- The shell is zsh. Unquoted `$var` does not word-split and `%G?` needs quoting, so use arrays or `bash -c` for loops.
- DocShots renders the real UI components with demo data. Update checks are answered offline there, so the header shows "Up to date".
- To check a dialog (AlertWindow) visually, temporarily add a snapshot of the modal component to DocShots, render it to the scratchpad, then restore the file.

## Every change that touches the UI or behaviour

Update everything that describes it in the same commit:
- `README.md`
- `docs/GUIDE.md` (sections, troubleshooting table, anchors used by README and the site). The website's guide page `docs/guide.html` is generated from it: after editing GUIDE.md run `python3 Tools/make_site_pages.py`
- `docs/index.html` (the website: features, gallery captions, support section)
- `docs/help.html` (Help page: troubleshooting quick answers, report/idea forms) and `docs/pages.css` (styles shared by help.html and the generated guide.html / changelog.html)
- `.github/ISSUE_TEMPLATE/` (problem.yml / idea.yml forms; the app's Report a problem prefills `version`, `os`, `daw` by field id, so keep those ids)
- the quick tour (`Source/Tour.cpp`). Inserting a step shifts the indices in DocShots' `tourShots`.
- screenshots: run DocShots into `docs/images`, then look at the changed images before committing.

Show the user screenshots before pushing a visual change.

## Commits

- Author and committer: `Thanasis Kostopoulos <26544748+thankost@users.noreply.github.com>` (set in the repo's local git config).
- Sign every commit with the SSH key `~/.ssh/thankost` (local config: `gpg.format ssh`, `commit.gpgsign true`). GitHub must show commits as **Verified**.
- **No `Co-Authored-By` trailers** and no other attribution lines.
- Push with thankost's credentials. If `gh auth status` shows another active account, run `gh auth switch -u thankost`, push with
  `git -c credential.helper= -c credential.helper='!gh auth git-credential' push ...`, then switch back.
- Rulesets protect `main` (no deletion or force-push) and `v*` tags (no deletion, moving or force-push). Only the repo admin can bypass them. Rewriting history needs the user's explicit OK.

## Releases

Use the `release` skill (`.claude/skills/release/SKILL.md`). In short:
1. Bump `project(PedalCues VERSION x.y.z)` in `CMakeLists.txt`.
2. Add the version at the top of `CHANGELOG.md`, in plain words for musicians, then run `python3 Tools/make_site_pages.py` to rebuild `docs/changelog.html` and `docs/guide.html` (never edit those pages by hand).
3. Build and run the tests.
4. Commit with a message that works as release notes: the first line is `vX.Y.Z: summary`, then `- bullet` lines. CI copies the tagged commit's message into the GitHub release, and the plugin's update window shows it under "What's new".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thankost/pedal-cues](https://github.com/thankost/pedal-cues) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
