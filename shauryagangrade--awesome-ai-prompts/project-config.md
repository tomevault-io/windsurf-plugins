---
trigger: always_on
description: Working notes for coding agents. Everything here was read out of `scripts/` and
---

# AGENTS.md

Working notes for coding agents. Everything here was read out of `scripts/` and
`.github/workflows/`, not out of the prose docs. Where the two disagree, the
scripts win.

## What this repo is

A docs repo: 101 prompt files in 13 category folders. The only real code is
`scripts/`: two Python scripts (the catalog generator and the external-link
checker), three bash linters, and their tests. There is no app, no root
`package.json`, no Makefile, and no test framework beyond `unittest`. 30 tests
cover the scripts.

## Verify before you finish

This is the CI contract. Run it in this order.

```bash
bash scripts/check-prettier.sh
npx --yes markdownlint-cli2@0.17.2 "**/*.md" "#docs/media/demo-app" "#drafts" "#tmp"
bash scripts/check-links.sh
python3 scripts/build-all.py --check
BASE_REF=origin/main bash scripts/check-consistency.sh
typos .
ruff check scripts/
shellcheck scripts/*.sh
actionlint && zizmor .github/workflows/
python3 -m unittest discover -s scripts/tests
```

`ec` (editorconfig-checker) and `gitleaks` are also CI jobs but need a binary
install; skip them unless you are touching line endings or adding a secret.

After changing any prompt file or `README.md`, run `python3 scripts/build-all.py`
(not `--check`) and commit the regenerated `ALL_PROMPTS.html`. CI fails on a
stale catalog.

## Stage new files before running the gates

The gates do not all discover files the same way, so a half-staged tree gives
green-looking results that are not.

| Discoverer                        | Mechanism               | Sees untracked |
| --------------------------------- | ----------------------- | -------------- |
| `build-all.py` `category_order()` | `git ls-files`          | no             |
| `build-all.py` `prompt_files()`   | filesystem glob         | yes            |
| `check-links.sh` section 1        | `git ls-files`          | no             |
| `check-links.sh` sections 2-4     | `find`                  | yes            |
| `check-consistency.sh`            | `find` / glob on `*/`   | yes            |
| `check-prettier.sh`               | `git ls-files --others` | yes            |

Two consequences, both reproduced:

- A new prompt file dropped into an existing category is picked up by the
  catalog even while untracked, because `prompt_files()` globs the filesystem.
  `build-all.py --check` fails with "committed ALL_PROMPTS.html is stale".
- A brand new **category folder** that is untracked is invisible to the catalog
  _and_ to `check-links.sh` section 1. Only `check-consistency.sh` catches it,
  by iterating folders on disk.

So `git add` new files before running the gates, or trust nothing.

## Adding a prompt touches six places

`check-consistency.sh` fails if any of these drifts. `CONTRIBUTING.md` mentions
most of them but not the exact-match requirement.

1. `<category>/<slug>-prompt.md` - H1, usage note, `Keywords:` line, `---`, body.
2. `README.md` Contents line, byte-exact. Section 4 uses `grep -qxF`, a
   whole-line exact match:
   `- [Core coding](#core-coding) (15) <img src="docs/media/spec-badge.svg" alt="spec" style="vertical-align:-3px">`
3. `README.md` section listing:
   `- [<slug>-prompt.md](<category>/<slug>-prompt.md) - one-line description`
4. `<category>/README.md` - one-line entry, plus the `../README.md#<anchor>`
   backlink.
5. `CHANGELOG.md` `[Unreleased]` - matched by basename only.
6. `ALL_PROMPTS.html` - regenerate.

If the prompt is `[spec]`, also add its path to `SPEC_PROMPTS` in
`scripts/check-consistency.sh` and put the badge `img` on its README listing
line. The badge on the Contents line is derived from `SPEC_PROMPTS`, so it is
not a separate manual edit.

### Anchors

Hand-maintained in `meta_for()`. An `&` in a heading vanishes and leaves a
**double** hyphen: `Data & AI` is `#data--ai`, `Code review & quality` is
`#code-review--quality`, `Frontend & UI` is `#frontend--ui`. Guessing a single
hyphen fails only when the link check runs.

## Adding a category touches two hand-maintained lists

Both in `scripts/check-consistency.sh`, which is the gate that fails most often:

- `CATEGORIES` - space-separated folder names.
- `meta_for()` - folder to `README heading|contents anchor`.

Plus the folder with its own `README.md`, and a `##` section plus a Contents
entry in `README.md`. Any other top-level folder must be added to the exclusion
case in section 3 (`scripts | docs | assets | i18n`) or it is reported as a
category with no README section.

Prefer deriving a new check from the files on disk over growing either list.

## House rules

- **No em dashes anywhere.** CI runs `git grep -I -l $'\u2014'`. Use an ASCII
  hyphen.
- **`ALL_PROMPTS.html` is generated.** Never hand-edit it. It is excluded from
  `typos`, `editorconfig-checker`, and the prettier glob.
- **The first `---` is the prompt boundary.** Above it is the intro (title,
  usage note, `Keywords:`); below it is what the user pastes. Nothing above the
  divider may be needed in order to use the prompt.
- **`.prettierrc.json` sets `proseWrap: preserve`.** Prettier will not rewrap
  your prose, so wrap near 79 columns yourself to match the file. It will still
  fix list markers, spacing, and table alignment.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shauryagangrade/awesome-ai-prompts](https://github.com/shauryagangrade/awesome-ai-prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
