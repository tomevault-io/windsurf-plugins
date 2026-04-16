---
trigger: always_on
description: Hugo portfolio + professional services site for Bryan Chasko. Fully custom theme
---

# Copilot Instructions – bryanchasko.com

Hugo portfolio + professional services site for Bryan Chasko. Fully custom theme
(`bryan-chasko-theme`) with 7 WebGL scenes, glassmorphism UI, and animated
backgrounds. Deployed to S3 + CloudFront via Perl deploy script + Terraform.

## Agent Squad

See `.squad/` for the full squad configuration:

- **Team roster:** `.squad/team.md`
- **Coordinator:** Harald — routes all work, manages scope and priority
- **Specialists:** Stratia (Site Architecture), Orin (Hugo Dev Environment),
  Solan (Build & Deploy), Myrren (AWS/CDN/Terraform), Kade Vox (Playwright/WebGL
  Tests), Scribe (Logger), Ralph (Work Monitor)
- **Agentic instruction fragments:** `agentic_instructions/` — use the matching
  fragment for your task area (CSS, setup, CI/CD, etc.)

---

## Project Overview

| Property | Value |
|---|---|
| Site URL | https://bryanchasko.com |
| Framework | Hugo |
| Theme | `themes/bryan-chasko-theme` — fully custom (not a PaperMod overlay) |
| Hosting | AWS S3 + CloudFront |
| IaC | Terraform (`terraform/`) with modules: acm, cloudfront, iam, route53, s3, ssm |
| CI/CD | GitHub Actions: `deploy.yml`, `terraform.yml`, `webgl-tests.yml` |
| Testing | Playwright (visual regression, WebGL, cross-browser) |

## Key Commands

```bash
hugo server --config config.dev.toml   # local preview
hugo --minify                          # production build
perl scripts/deploy.pl                 # build + S3 sync + CloudFront invalidation
npm test                               # Playwright (all browsers)
npm run test:chromium                  # Chrome only
npm run test:update-baselines          # regenerate visual baselines after intentional changes
cd terraform && terraform plan         # preview infra changes
cd terraform && terraform apply        # apply infra changes
```

## Theme Architecture

The custom theme lives entirely in `themes/bryan-chasko-theme/`:

```
assets/
  css/
    core/        ← variables.css, typography, reset
    components/  ← home.css, terminal.css, navigation.css, cards.css,
                    social-feed.css, github-calendar.css, github-dashboard.css,
                    contact.css, notes-enhancements.css, cloudcroft.css
    extended/    ← nebula.css (animated backgrounds, color palettes)
    layouts/     ← page layout CSS
    includes/    ← shared partials CSS
    common/      ← utility CSS
  js/
    webgl-scenes/  ← BaseScene.js, OrbitScene.js, RippleScene.js,
                      ShimmerScene.js, SkillsNetworkScene.js,
                      TransitionScene.js, SceneInitializer.js
    constellation.js, webgl-monitor.js, notes-tilt.js, fastsearch.js
```

## Conventions

- **`themes/bryan-chasko-theme/` IS the theme** — it is not a submodule and IS
  edited directly. `themes/PaperMod/` is the submodule — never touch it.
- **CSS variables only** — colors from `variables.css` or `nebula.css`; never
  hardcode hex values
- **WebGL:** edit scenes in `themes/bryan-chasko-theme/assets/js/webgl-scenes/`,
  then sync to static:
  `cp themes/bryan-chasko-theme/assets/js/webgl-scenes/*.js themes/bryan-chasko-theme/static/js/webgl-scenes/`
- **Deploy:** `perl scripts/deploy.pl` — not a shell script
- **Never deploy with `buildDrafts = true`**
- **Terraform:** changes go through `terraform plan` → review → `terraform apply`;
  state is remote (backend.tf)
- **Agentic instructions:** see `agentic_instructions/README.md` for task-specific
  guidance fragments

---

# Agent Instructions: Home Page CSS (Fragment)

## Purpose

Guidance for AI agents working specifically on CSS for the home page (including css terminal simulator, builder card, social icons, and animated backgrounds).

---

## Key Files

- `themes/bryan-chasko-theme/assets/css/components/home.css` ← Home page layout, terminal, builder card, social icons
- `themes/bryan-chasko-theme/assets/css/extended/nebula.css` ← Animated backgrounds, color palettes
- `themes/bryan-chasko-theme/assets/css/core/variables.css` ← CSS custom properties (colors, spacing, typography)

## Principles

- **Use MCP CSS tools** for analysis, browser compatibility, and documentation before making changes
- **Do not modify PROTECTED rules** in `header.css` (logo sizing, mobile nav)
- **All home page CSS must be responsive** (mobile, tablet, desktop)
- **Respect dark mode**: Use `[data-theme="dark"]` selectors for overrides
- **Animated backgrounds** must respect `prefers-reduced-motion`
- **Do not hardcode colors**: Use CSS variables from `variables.css` or `nebula.css`

## Workflow

1. Analyze current CSS with `mcp_css_analyze_css` or `mcp_css_analyze_project_css`
2. Check browser compatibility for any new features (e.g., `:has`, container queries)
3. Reference MDN docs for syntax (`mcp_css_get_docs`)
4. Make changes in `home.css` for layout, builder card, terminal, or social icons
5. For background/animation changes, edit `nebula.css` only
6. Test on all breakpoints and both light/dark mode
7. Validate with MCP CSS tools after changes

## Home Page Structure

- **Terminal**: `.terminal-wrapper`, `.entry-header` (animated, glassmorphism)
- **Social Icons**: `.terminal-social-reveal` (revealed after terminal animation)
- **Builder Card**: `.builder-card` (glassmorphism, WebGL overlay)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chasko-labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
