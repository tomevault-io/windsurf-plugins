---
trigger: always_on
description: Quick orientation for an agent (or a person) working in this repo.
---

# Resume — Agent Knowledge

Quick orientation for an agent (or a person) working in this repo.

## Repo layout

| Path | What it is |
|---|---|
| `template/*.tex` | The resume content and layout. Compiled with XeLaTeX. |
| `template/resume.yml` | Per-branch build config: `variant`, `label`, `author`, `template`, `output` |
| `template/fonts/*.ttf` | Ubuntu font family, loaded by `fontspec` via `Path=fonts/`. Must stay in the same directory as the `.tex` file. |
| `template/icons/*` | Logos/icons referenced from the `.tex`, e.g. `\iconentry{icons/foo.png}{...}{...}` |
| `template/index.html` | Source for the Pages viewer. Copied to `public/index.html` during CI, with `{{OWNER}}`, `{{REPO}}` substituted in. |
| `.github/workflows/build-resume.yml` | The entire pipeline — compile, release, and Pages deploy — in one job, one file |
| `skills/*.md` | Optional guidance modules for content-authoring tasks. Not read by CI — see **Skills** below. |

CI never commits back to the repository. Build outputs (the compiled PDF, the
dated archive copy, the generated site) exist only as release assets or a
Pages deployment — never as git history.

## Workflow trigger

```yaml
on:
  push:
    branches:
      - main
      - 'resume/**'
    paths: ["template/**"]
  workflow_dispatch: {}
```

`permissions: contents: write, pages: write, id-token: write` (needed for releases
and the Pages OIDC deploy). `concurrency: group: build-resume, cancel-in-progress: false`
— overlapping pushes queue instead of racing each other's release edits.

## Branch-per-variant architecture

Each resume variant lives on its own branch. The branch name follows the
convention `resume/<variant-id>` (e.g. `resume/facebook-de`, `resume/amazon-sde`).
The `main` branch holds the general/default variant.

Every branch carries its own `template/resume.yml`:

```yaml
variant: facebook-de
label: Facebook — Data Engineer
author: "Sadig Akhund"
template: "Resume.tex"
output: "SadigAkhund_Facebook_DE"
```

- **`variant`** — a short slug used in release tag names (must be filesystem-safe, e.g. `facebook-de`)
- **`label`** — human-readable name shown in the viewer's profile selector
- **`template`**, **`output`**, **`author`** — same as before

When you push to any matching branch, CI builds that branch's `.tex` and
uploads the PDF to namespaced releases. Pages is deployed **only from `main`**.

## Release tags

Each variant gets its own release tag family, namespaced by variant ID:

| Tag pattern | Created | Lifecycle | Purpose |
|---|---|---|---|
| `latest-{variant}` | every run (deleted + recreated) | recreated each run to freshen `created_at` | stable link to newest PDF for this variant |
| `resume-{variant}-YYYY-MM-DD` | only when PDF content changes vs `latest-{variant}` | deleted + recreated per change; `--prerelease` | dated archival snapshot for this variant |

Examples:
- `latest-general`, `resume-general-2026-07-28`
- `latest-facebook-de`, `resume-facebook-de-2026-07-28`
- `latest-amazon-sde`, `resume-amazon-sde-2026-07-28`

Each variant's release history is independent — pushing to `resume/facebook-de`
only touches the `*-facebook-de-*` tags; `general` and `amazon-sde` releases
are never affected.

## Build steps (in order)

1. Checkout
2. Cache the `ghcr.io/xu-cheng/texlive-full` Docker image, keyed on its remote
   manifest digest, so most runs skip re-pulling the full TeX Live image
3. Read `template/resume.yml` with PyYAML → `VARIANT`, `VARIANT_LABEL`,
   `AUTHOR`, `TEMPLATE`, `OUTPUT` env vars
4. Compile `template/${TEMPLATE}` via `xu-cheng/latex-action@v3`
   (`latexmk_use_xelatex: true`, `extra_fonts: template/fonts/*.ttf`) — **must stay
   XeLaTeX**, since `fontspec` + the bundled TTFs won't work under `pdflatex`
5. Normalize the compiled PDF to `<OUTPUT>.pdf` at the repo root, and copy it into
   a build-only `Archive/<OUTPUT>_<DATE>.pdf` (gitignored — only used as a release
   asset within this run)

### Release upload logic

1. **SHA256 change detection** — downloads the current `latest-{variant}` PDF,
   SHA256-compares against the freshly built one. `CHANGED=1` if different, or if
   `latest-{variant}` doesn't exist yet.
2. **If changed** — delete old `resume-{variant}-YYYY-MM-DD` tag and recreate it
   with the archive PDF; then recreate `latest-{variant}` (deleted + recreated).
3. **If unchanged** — find most recent dated release for this variant and update
   notes to "Verified unchanged"; recreate `latest-{variant}` with the existing
   PDF.

The `latest-{variant}` release is **always deleted and recreated last** so its
`created_at` timestamp is always the most recent. GitHub sorts releases by
`created_at` descending, so `latest-{variant}` shows up first in the release
list and the viewer's version dropdown.

No `--latest` flag is set on any release — GitHub's repo-wide "latest release"
flag would race between variants if set, and the viewer doesn't use it.

### Pages deployment

Pages steps only run when `github.ref_name == 'main'`:
- `template/index.html` → `public/index.html`
- `sed` substitutes `{{OWNER}}`, `{{REPO}}`
- Deployed via `actions/upload-pages-artifact` + `actions/deploy-pages`

The viewer discovers all variants dynamically from the release list API —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AS-FOSS/git-resume](https://github.com/AS-FOSS/git-resume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
