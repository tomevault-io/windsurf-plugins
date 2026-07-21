---
trigger: always_on
description: Instructions for AI agents working on Docker documentation.
---

# AGENTS.md

Instructions for AI agents working on Docker documentation.
This site builds https://docs.docker.com/ using Hugo.

## Project structure

```
content/          # Documentation source (Markdown + Hugo front matter)
├── manuals/      # Product docs (Engine, Desktop, Hub, etc.)
├── guides/       # Task-oriented guides
├── reference/    # API and CLI reference
└── includes/     # Reusable snippets
layouts/          # Hugo templates and shortcodes
data/             # YAML data files (CLI reference, etc.)
assets/           # CSS (Tailwind v4) and JS (Alpine.js)
static/           # Images, fonts
_vendor/          # Vendored Hugo modules (read-only)
```

## URL prefix stripping

The `/manuals` prefix is stripped from published URLs:
`content/manuals/desktop/install.md` becomes `/desktop/install/` on the live
site.

When writing internal cross-references in source files, keep the `/manuals/`
prefix in the path — Hugo requires the full source path. The stripping only
affects the published URL, not the internal link target. Anchor links must
exactly match the generated heading ID (Hugo lowercases and slugifies
headings).

## Vendored content (do not edit)

Content in `_vendor/` and CLI reference data in `data/cli/` are vendored
from upstream repos. Content pages under `content/reference/cli/` are
generated from `data/cli/` YAML. Do not edit any of these files — changes
must go to the source repository:

| Content | Source repo |
|---------|-------------|
| CLI reference (`docker`, `docker build`, etc.) | docker/cli |
| Buildx reference | docker/buildx |
| Compose reference | docker/compose |
| Model Runner reference | docker/model-runner |
| Dockerfile reference | moby/buildkit |
| Engine API reference | moby/moby |
| AI Governance API (`content/reference/api/ai-governance/api.yaml`) | docker/governor-services (private) |

If a validation failure or broken link traces back to vendored content, note
the upstream repo that needs fixing. Do not attempt to fix it locally.

`content/reference/api/ai-governance/api.yaml` is a verbatim copy of the
upstream `openapi.yaml` — do not edit it by hand. Re-vendor it with
`hack/sync-governance-api.sh`, which fetches the latest spec from the private
`docker/governor-services` repo (using your own `gh` auth).

## Writing guidelines

Read and follow [STYLE.md](STYLE.md) and [COMPONENTS.md](COMPONENTS.md).
These contain all style rules, shortcode syntax, and front matter requirements.

### Style violations to avoid

Every piece of writing must avoid these words and patterns (enforced by Vale):

- Hedge words: "simply", "easily", "just", "seamlessly"
- Meta-commentary: "it's worth noting", "it's important to understand"
- "allows you to" or "enables you to" — use "lets you" or rephrase
- "we" — use "you" or "Docker"
- "click" — use "select"
- Bold for emphasis or product names — only bold UI elements
- Time-relative language: "currently", "new", "recently", "now"

### Version-introduction notes

Explicit version anchors ("Starting with Docker Desktop version X...") are
different from time-relative language — they mark when a feature was
introduced, which is permanently true.

- Recent releases (~6 months): leave version callouts in place
- Old releases: consider removing if the callout adds little value
- When in doubt, keep the callout and flag for maintainer review

### Vale gotchas

- Use lowercase "config" in prose — `vale.Terms` flags a capital-C "Config"

### Updating the vocabulary

If Vale flags a legitimate tech term, product name, or compound identifier
as a misspelling, add it to `_vale/config/vocabularies/Docker/accept.txt`.
This is optional — only update when a real new term is missing, not to
silence individual violations.

- Use the canonical form for case-sensitive product names (`PyTorch`,
  `GitHub`, `Kubernetes`, `BuildKit`). `Vale.Terms` enforces that exact
  case across the docs.
- Use `[Aa]bcd` character-class regex for words that legitimately appear
  in multiple cases (e.g., sentence-starting capitalization, or a name
  that's also a generic noun). This covers spelling without enforcing
  a single canonical form.
- Avoid broad regex patterns — entries that match many words at once
  (especially with `(?i)`) suppress other rule checks on every match.
- Don't add a wrong-cased entry to silence one false positive — it
  cascades into `Vale.Terms` violations on every correct usage.

## Alpine.js patterns

Do not combine Alpine's `x-show` with the HTML `hidden` attribute on the
same element. `x-show` toggles inline `display` styles, but `hidden` applies
`display: none` via the user-agent stylesheet — the element stays hidden
regardless of `x-show` state. Use `x-cloak` for pre-Alpine hiding instead.
The site defines `[x-cloak=""] { display: none !important }` in `global.css`.

## Front matter requirements

Every content page under `content/` requires:

- `title:` — page title
- `description:` — short description for SEO/previews
- `keywords:` — list of search keywords (omitting this fails markdownlint)

Additional common fields:

- `linkTitle:` — sidebar label (keep under 30 chars)
- `weight:` — ordering within a section

## Hugo shortcodes

Shortcodes are defined in `layouts/shortcodes/`. Syntax reference is in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [docker/docs](https://github.com/docker/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
