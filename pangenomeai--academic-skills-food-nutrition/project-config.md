---
trigger: always_on
description: This file is the contract for anyone (human or AI coding agent — Claude Code,
---

# AGENTS.md — instructions for collaborators and their coding agents

This file is the contract for anyone (human or AI coding agent — Claude Code,
Codex, Cursor, etc.) contributing to **academic-skills-food-nutrition**. Read it
fully before making changes. It is written to be machine-actionable: follow the
rules literally.

## What this project is
A Claude Code / Codex **skill suite** for food & nutrition science research:
multi-subagent skills (`food-research`, `food-deep-research`, `food-paper`,
`food-review`, `food-pipeline`, `food-figure`), a `journal-selector`, and
publisher-tiered journal author-guideline skills under `journals/`. MIT-licensed,
original work.

## GOLDEN RULE — branching
**Never push to `main`.** `main` is protected and release-only.

1. Do all work on the **`development`** branch (or a feature branch off it).
2. Push your commits to `development` (or your feature branch).
3. Open a **pull request** on GitHub to merge `development` → `main`.
4. `main` changes **only** through a reviewed, merged pull request.

```bash
git checkout development
git pull origin development
# … make changes …
git add -A && git commit -m "…"
git push origin development
gh pr create --base main --head development --title "…" --body "…"
```

Do not merge your own PR without review unless you are the maintainer and the
checks below pass.

## ALWAYS keep docs current (required in the same change)
Every change that adds/removes/renames a skill, subagent, reference, or command,
or changes behavior, MUST in the same PR:
- Update **`README.md`** to reflect the new state.
- Add a dated entry to **`CHANGELOG.md`** (newest on top) describing the change.
- Bump the `version` in **both** `.claude-plugin/plugin.json` and
  `.claude-plugin/marketplace.json` (semver: patch = fix, minor = new capability,
  major = breaking).

A PR that changes skills but not README/CHANGELOG will be rejected.

## Releases (automatic on major updates)
A GitHub Release is published **automatically** when a **major update** lands on
`main` — defined as a new **MINOR or MAJOR** version (`x.y.0`, e.g. a new skill,
subagent, or capability). The `.github/workflows/release.yml` workflow runs on
every push to `main` that touches `.claude-plugin/plugin.json` or `CHANGELOG.md`,
and creates release `v<version>` using the matching `CHANGELOG.md` section as the
notes. **Patch releases** (`x.y.z` with `z>0`, e.g. docs/typo fixes) do **not**
create a release.

So, as a developer/agent:
- For a **feature or major change**, bump the version to the next **`x.y.0`** (minor)
  or **`(x+1).0.0`** (major/breaking) and write a matching `## x.y.0 — <date>`
  section in `CHANGELOG.md`. The release publishes itself when the PR merges to `main`.
- For a small fix, use a **patch** bump (`x.y.z`) — no release is created.
- Do **not** create releases by hand for normal changes; let the workflow do it.
  (Manual release if ever needed: `gh release create vX.Y.Z --notes-file NOTES.md --target main`.)

## Verification — run before opening a PR (all must pass)
```bash
# 1. Journal coverage maps resolve (food 60, nutrition list)
python3 scripts/check_journal_coverage.py

# 2. scripts self-test
python3 food-figure/scripts/analyze_data.py --selftest
python3 food-figure/scripts/backend_pref.py --selftest
python3 scripts/verify_citations.py --selftest        # citation/anti-fabrication gate
python3 scripts/privacy_scan.py --selftest            # privacy/confidentiality gate

# 3. Manifests are valid JSON (and, if you have Claude Code, validate the plugin)
python3 -c "import json;[json.load(open(p)) for p in ['.claude-plugin/plugin.json','.claude-plugin/marketplace.json']];print('json ok')"
claude plugin validate .    # optional, if claude CLI is installed

# 4. Every SKILL.md 'name' matches its folder; every frontmatter reference resolves
#    (see the checks in the PR template / repo scripts)

# 5. English only — no CJK or other non-Latin scripts in tracked files
```

## Content rules
- **Grounding — no fabrication (applies to every research/writing/review agent).**
  All output must be written **100% from true information** collected from the
  literature and the user's own data. **Never invent** a reference, DOI, author,
  year, venue, quotation, locator, statistic, effect size, sample size, or result.
  If a source does not report a value, write "not reported"; if a claim cannot be
  traced to a verified source, mark it `[UNVERIFIED]` / `[EVIDENCE GAP]` and route
  it back for sourcing — do not fill it from memory. Every citation must pass the
  four-gate check (exists → identity matches → not retracted → source actually
  supports the claim at a locator). See
  `food-paper/references/faithfulness-and-citation.md` and run
  `python3 scripts/verify_citations.py <cites.json>` on the reference set.
- **Privacy — check before delivering (every skill).** Before handing any
  document to the user (manuscript, brief, report, response letter, figure
  legend, code, PDF/DOCX/LaTeX), ensure it contains **no confidential/local
  information** — absolute filesystem paths (`/Users/<name>/…`, `C:\Users\…`),
  temp/scratchpad paths, usernames, hostnames, IPs, API tokens, or private data.
  Use relative/placeholder paths instead. Scan with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PangenomeAI/academic-skills-food-nutrition](https://github.com/PangenomeAI/academic-skills-food-nutrition) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
