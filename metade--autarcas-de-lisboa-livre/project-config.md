---
trigger: always_on
description: Jekyll website for the Livre party's elected officials (autarcas) in Lisbon. Hosted on GitHub Pages, with Sveltia CMS for content editing.
---

# Autarcas de Lisboa – Livre

Jekyll website for the Livre party's elected officials (autarcas) in Lisbon. Hosted on GitHub Pages, with Sveltia CMS for content editing.

Deployed at: `https://autarcas-lisboa-livre.metade.org`

## Stack

- **Jekyll 4.3** — static site generator
- **Tailwind CSS 3** — utility-first styling, built via npm CLI before Jekyll
- **Sveltia CMS** — headless CMS loaded from CDN at `/admin/`, GitHub backend
- **GitHub Pages** — hosting via GitHub Actions (not legacy branch deploy)
- **Cloudflare Worker** — OAuth proxy for Sveltia CMS GitHub authentication, deployed at `https://autarcas-de-lisboa-cms-auth.metade.workers.dev`
- **`_plugins/date_filter.rb`** — Ruby plugin providing `date_pt` Liquid filter (Portuguese date formatting)
- **`_data/pt.yml`** — Portuguese month names used by the date filter

## Local Development

```bash
npm install          # install Tailwind
bundle install       # install Jekyll gems
npm run watch:css    # rebuild CSS on changes
bundle exec jekyll serve  # serve at http://127.0.0.1:4000
```

For a one-shot build: `npm run build:css && bundle exec jekyll build`

## Project Structure

```
_autarcas/       One .md per elected official (16 files)
_juntas/         One .md per parish assembly (9 files — Assembleias de Freguesia only)
_propostas/      Per-organ subfolders: _propostas/{organ}/{year}-{slug}.md + co-located PDFs
_tags/           Controlled tag vocabulary (output: false); each file has a `nome` field
_locations/      Controlled location vocabulary (output: false); each file has `nome` + `junta` fields
_pages/          Static pages (registered as a Jekyll collection so Jekyll outputs them)
_layouts/        default, autarca, junta, proposta, propostas_organ, page
_includes/       head, nav, footer, autarca-card, proposta-card
_plugins/        date_filter.rb — date_pt Liquid filter for Portuguese dates
_data/           pt.yml — Portuguese month names
assets/css/      main.css (Tailwind input) → main.min.css (compiled, gitignored)
assets/js/       filter.js — vanilla JS client-side filtering for proposals
admin/           Sveltia CMS entry (index.html) and config (config.yml)
cloudflare-worker/  OAuth proxy source (src/index.js, wrangler.toml)
```

> **Tailwind note:** After adding new utility classes to templates, rerun `npm run build:css` so they are included in the compiled output. Classes not present at build time will be silently dropped.

## Site Structure

| URL | Source |
|-----|--------|
| `/` | `index.md` |
| `/camara-municipal/` | `_pages/camara-municipal.md` (uses `junta` layout) |
| `/assembleia-municipal/` | `_pages/assembleia-municipal.md` (uses `junta` layout) |
| `/juntas/` | `_pages/juntas.md` — lists the 9 parish assemblies |
| `/juntas/:slug/` | `_juntas/*.md` |
| `/autarcas/` | `_pages/autarcas.md` |
| `/autarcas/:slug/` | `_autarcas/*.md` |
| `/propostas/` | `_pages/propostas.md` — filterable listing |
| `/propostas/:organ/` | `_pages/propostas-{organ}.md` — per-organ listing (uses `propostas_organ` layout) |
| `/propostas/:organ/:slug/` | `_propostas/{organ}/*.md` |
| `/sobre/` | `_pages/sobre.md` |
| `/admin/` | Sveltia CMS interface |

**Navigation order:** Câmara Municipal → Assembleia Municipal → Juntas de Freguesia → Autarcas → Propostas

## Content Model

### Autarcas (`_autarcas/*.md`)
One file per person, regardless of how many roles they hold. A person with roles in multiple organs (e.g. João Monteiro) has a single profile page listing all roles.

Key fields:
- `genero` — `m | f | n`, drives grammatical gender in Portuguese role display (o/a convention)
- `juntas` — flat list of junta slugs for Liquid filtering (`where_exp: "a.juntas contains slug"`)
- `cargos` — structured list of cargo objects, displayed on the profile page

Cargo object fields:
- `cargo`, `orgao`, `junta` — required
- `ausente_temporariamente` — boolean; member is temporarily suspended
- `temporario` — boolean; this is a temporary substitution role
- `substitui` — slug of the member being substituted (set on the substitute)
- `substituido_por` — slug of the substitute (set on the suspended member)

Example (substitute member):
```yaml
cargos:
  - cargo: Membro de Assembleia
    orgao: Assembleia de Freguesia de Arroios
    junta: arroios
    temporario: true
    substitui: patricia-robalo
    substituido_por: ''
```

Example (suspended member):
```yaml
cargos:
  - cargo: Membro de Assembleia
    orgao: Assembleia de Freguesia de Arroios
    junta: arroios
    ausente_temporariamente: true
    substituido_por: patrick-sinclair
```

Junta pages render suspended members in a separate faded row below active members.

### Juntas (`_juntas/*.md`)
Only the 9 parish-level assemblies. Câmara Municipal and Assembleia Municipal are standalone pages in `_pages/`, not in this collection.

Key fields: `nome`, `slug`, `tipo`, `descricao`, `foto_junta`

Social link fields (all optional, empty string if unused): `website_oficial`, `facebook`, `x`, `instagram`, `youtube`

### Autarca social fields

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/metade) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
