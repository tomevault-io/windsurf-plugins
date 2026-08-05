---
trigger: always_on
description: Guidance for AI agents working in `jupyter-ai` and its subpackages.
---

# AGENTS.md

Guidance for AI agents working in `jupyter-ai` and its subpackages.

## What Jupyter AI is

As of v3, `jupyter-ai` is **not a monorepo**. It is a metapackage that
distributes a working set of subpackages, each living in its own repo under the
[`jupyter-ai-contrib`](https://github.com/jupyter-ai-contrib) org. The
`jupyter-ai` repo itself is mostly empty; the composition is recorded in
`submodules/manifest.json` (a `"pypi_name": "org/repo"` map) and in the
[Contributor Guide](https://jupyter-ai.readthedocs.io/en/latest/contributors/index.html).

## Documentation lives with the code

Documentation is split by audience:

- **User docs** live solely in this repo, under `docs/source/users/`.
- **Contributor and developer docs live in each subpackage's own repo.** The
  main Jupyter AI docs site aggregates them into versioned subpages, so
  everything is readable in one place.

### If you are working in a subpackage repo

When you add or change functionality in a subpackage under `jupyter-ai-contrib/`,
put its contributor/developer documentation **in that subpackage's own repo**,
not here. Create either or both of:

```
docs/source/contributors/index.md   # → surfaces under Contributors on the main site
docs/source/developers/index.md     # → surfaces under Developers on the main site
```

- The `index.md` **H1 becomes the subpage title** — use the repo name by
  convention (e.g. `# jupyter-ai-tools`).
- The two sections are independent; define one, both, or neither.
- You may ship a **full subtree** (nested pages + images); your `index.md`'s
  `{toctree}` structures it.
- These pages flow up to https://jupyter-ai.readthedocs.io automatically once a
  maintainer bumps the doc submodule pins (see below). You do not edit the
  `jupyter-ai` repo to publish subpackage docs.

This is a **gradual rollout** — most subpackages do not have these docs yet, so
adding them is a genuine improvement, not a requirement to match existing repos.

### If you are working in this (`jupyter-ai`) repo

The submodule-docs aggregation infrastructure lives here:

- `submodules/manifest.json` — registry of doc submodules.
- `submodules/<repo>/` — git submodules, sparse-checked-out to `docs/` only.
- `scripts/update-doc-submodules.sh` — re-pins each submodule to the latest tag
  matching its `pyproject.toml` version range (PEP 440 via `packaging`). Run by
  the **Update submodule documentation** workflow (`workflow_dispatch`), which
  opens a PR with the new pins.
- `scripts/rtd-post-checkout.sh` — Read the Docs `post_checkout` hook that
  sparse-initialises the submodules.
- `docs/source/_ext/subpackage_docs.py` — Sphinx extension that copies each
  submodule's `docs/source/{contributors,developers}/` into git-ignored staging
  dirs and injects them into the aggregation toctrees. Missing docs are silently
  skipped; the build stays green.
- `docs/tests/` — integration tests (real `sphinx-build` against synthesized
  fixtures). The **Docs build system** CI workflow runs them on any change to
  the docs build system. Run them locally with `pytest docs/tests`.

When you add a new subpackage dependency that should contribute docs, add it to
`submodules/manifest.json` and run `scripts/update-doc-submodules.sh`.

## Writing release notes

Each release has a page under `docs/source/releases/<version>.md`. The title,
date, and per-subpackage changelog are auto-generated (wrapped in
`AUTO-GENERATED` markers — don't edit those). Your job as a contributor is the
**summary** in the `SUMMARY` markers: a human-readable overview that a reader
sees before the raw changelog. Edit only between the SUMMARY markers; a re-run of
the release-notes workflow regenerates everything else and preserves your
summary.

### Generating the page (Step 0 workflow)

The page is produced by the **"Step 0: Prep release documentation"** workflow
(`.github/workflows/prep-release-docs.yml`), a manual `workflow_dispatch`. Run it
with the GitHub CLI:

```
gh workflow run "Step 0: Prep release documentation" \
  --ref main \
  -f version=v3.1.0 \
  -f target-branch=main
```

- `version` — the release being prepped (e.g. `v3.1.0`).
- `target-branch` — the branch the release is cut from; patch releases of an
  older minor use that minor's maintenance branch (e.g. `3.0.x`) instead of `main`.

The workflow generates the page from the version floors in `pyproject.toml`,
commits it to a `release-docs/<version>` branch, and opens a draft PR against the
target branch. It is **safe to re-run**: it reuses the same branch and open PR,
merges the target branch in to pick up the latest floors and docs, regenerates
the auto sections, and preserves your hand-edited summary. Watch a run with:

```
gh run watch "$(gh run list --workflow 'Step 0: Prep release documentation' \
  -L1 --json databaseId --jq '.[0].databaseId')" --exit-status
```

Write the summary for **users first**, developers second. Use these sections, in
order (drop any that don't apply):

- `## Highlights` — the handful of changes users will care about most.
- `## Major bugs fixed` — notable fixes, in plain "what was broken, now works"
  terms.
- `## API changes` — for extension developers: what they can now build, and the
  methods they'd implement.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jupyterlab/jupyter-ai](https://github.com/jupyterlab/jupyter-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
