---
trigger: always_on
description: This file provides guidance for AI coding agents (Claude Code, Cursor, Aider, OpenAI Codex CLI, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance for AI coding agents (Claude Code, Cursor, Aider, OpenAI Codex CLI, etc.) when working with code in this repository.

## Project Overview

kube-image-keeper (kuik) v2 is a Kubernetes operator providing container image routing, mirroring, and replication. Built with Go and Kubebuilder v4 (controller-runtime). It intercepts Pod creation via a mutating webhook and rewrites container images to the first available alternative from a prioritized list defined via CRDs.

## Common Commands

The Makefile is the source of truth — run `make help` for the full list. A few non-obvious things:

- `go test ./internal/controller/kuik -run TestName -v` — run a single unit test
- `make test-e2e` needs a running Kind cluster (see [`docs/guides/development.md`](./docs/guides/development.md))

For the documentation site (Astro / Starlight), commands live in [`website/`](./website/) — see its [README](./website/README.md). Local preview: `cd website && npm install && npm run dev`.

## Documentation

User-facing documentation lives under [`docs/`](./docs/) at the repository root and is published at **[kuik.enix.io](https://kuik.enix.io)**. The markdown files are the single source of truth (read them alongside the code when working on the corresponding areas):

- [`docs/crds.md`](./docs/crds.md): CRD reference with all fields and semantics
- [`docs/concepts/image-routing.md`](./docs/concepts/image-routing.md): deep dive into the priority system and webhook routing logic
- [`docs/configuration.md`](./docs/configuration.md): all config fields with defaults and examples
- [`docs/concepts/resource-filtering.md`](./docs/concepts/resource-filtering.md): image / namespace / pod filtering semantics
- [`docs/guides/development.md`](./docs/guides/development.md): local development setup and workflow

The same files render both on GitHub (browse `docs/`) and on the Astro Starlight site. Author them in GitHub-flavored markdown; the website build converts the GitHub-specific bits to Starlight at build time (see [How the docs site is built](#how-the-docs-site-is-built)).

### Markdown conventions for the docs site

Write for GitHub first; the build adapts. When editing or adding docs under `docs/`:

- Give every page its title as a leading `# H1` (the first body line), not a frontmatter `title:`. It reads naturally on GitHub, and the `sync-docs` build lifts that H1 into the Starlight frontmatter `title:` and strips it from the body (so Starlight does not render a duplicate heading). Add a `description:` in frontmatter; it doubles as the SEO `<meta>` description and is shown on the use-cases index cards. A page with only a `description:` (and no `title:`) is expected; GitHub renders that small frontmatter table above the `# H1`. Website-only overlay pages keep a frontmatter `title:` and are left untouched by the lift.
- Use relative markdown links between docs: `[Label selectors](./resource-filtering.md#label-and-annotation-selector-syntax)`. They work on GitHub, and the build rewrites them to site routes (`/resource-filtering/#...`). Do NOT write rendered routes like `/resource-filtering/`; they break on GitHub.
- Use GitHub alert syntax for callouts: `> [!NOTE]`, `> [!TIP]`, `> [!WARNING]`, `> [!IMPORTANT]`, `> [!CAUTION]`. They render natively on GitHub, and the build converts them to Starlight asides (note / tip / caution / danger). Do NOT use Starlight's `:::note` syntax; it shows as raw text on GitHub.
- New use-case files dropped under `docs/use-cases/` are automatically picked up by the use-cases index page and the sidebar on the next build.

### How the docs site is built

The site (in [`website/`](./website/), Astro Starlight) does not read `docs/` directly. A `sync-docs` integration in [`website/astro.config.mjs`](./website/astro.config.mjs) copies `docs/` and [`website/src/content/overlay/`](./website/src/content/overlay/) into `website/src/content/docs/` (gitignored, generated) before content collections load, and also generates each archived version's pages there (see versioning below). The overlay holds website-only pages that must not live in `docs/` (currently the use-cases landing `use-cases/index.mdx`); it is copied last (so it wins on path conflicts) into the current version **and** into every archived version. While copying, `sync-docs` also lifts each page's leading `# H1` into the frontmatter `title:` Starlight requires (and removes it from the body), so docs can carry their title as a GitHub-friendly H1; pages that already define a frontmatter `title:` are left as-is.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enix/kube-image-keeper](https://github.com/enix/kube-image-keeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
