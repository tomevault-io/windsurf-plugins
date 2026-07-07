---
trigger: always_on
description: Purpose: help AI agents and contributors be productive immediately when working in this repository.
---

# GitHub Copilot / AI Agent Instructions

Purpose: help AI agents and contributors be productive immediately when working in this repository.

Quick Summary

- Single-page React portfolio built with Vite and TailwindCSS.
- **Fully plug-and-play**: All content lives in JSON files (`src/config.json` and `src/data/*.json`). Zero hardcoded content.
- Main application: `src/App.jsx` imports JSON data and passes to data-driven components. `src/main.jsx` bootstraps the app.
- Styling: `src/index.css` contains Tailwind directives and project-specific fallback/custom CSS.
- Build/deploy: Vite (`npm run dev` / `npm run build`) and GitHub Pages via `gh-pages` (`npm run deploy`).

Quick Start (dev + build)

- Install dependencies: `npm install`
- Run dev server: `npm run dev` (uses Vite)
- Build for production: `npm run build`
- Preview production build: `npm run preview`
- Deploy to GitHub Pages: `npm run deploy` (uses `gh-pages -d dist`)

Important Files & Why

**Configuration:**

- `src/config.json` — Central hub for personal info, titles, bio, social links, theme. Edit this first to customize for a new portfolio owner.

**Data (JSON-based):**

- `src/data/aboutMe.json` — Bio sections, competencies, and philosophy. Used by `About.jsx`.
- `src/data/awards.json` — All awards with title, year, quarter, company, description. Used by `Awards.jsx`.
- `src/data/experience.json` — Work history with roles, highlights, key achievements. Used by `Experience.jsx`.
- `src/data/certifications.json` — Certifications with issuers, dates, credentials. Used by `Certifications.jsx`.
- `src/data/techStacks.json` — Technical skills and proficiencies. Used by `TechStack.jsx`.
- `src/data/projects.json` — Portfolio projects with support for featured/regular projects, modular sections (keyFeatures, technologies, quickStart), and data-driven content. Each project controls which sections display via `sections` array. Used by `Projects.jsx` → `ProjectCard.jsx` → `CardSection.jsx`.
- `src/data/mediumArticles.json` — Medium articles feed (auto-fetched by `update-medium.js` or manual). Used by `MediumArticles.jsx`.
- `src/data/education.json` — Education with degrees, institutions, periods, focus areas. Used by `Education.jsx`.

**Components:**

- `src/App.jsx` — Main orchestrator: imports all JSON files and passes them as props to components. Navigation state managed here.
- `src/components/Nav.jsx` — Navigation bar using `config.personal.name`.
- `src/components/Hero.jsx` — Hero section using `config.bio.headline`, `config.bio.subtitle`, personal links.
- `src/components/About.jsx` — About section rendering from `aboutMe.json`.
- `src/components/Awards.jsx` — Awards section (compact centered cards) rendering from `awards.json`.
- `src/components/Experience.jsx` — Experience section rendering from `experience.json`.
- `src/components/Certifications.jsx` — Certifications rendering from `certifications.json`.
- `src/components/TechStack.jsx` — Tech skills rendering from `techStacks.json`.
- `src/components/Projects.jsx` — Projects section orchestrator. Renders featured/regular projects from `projects.json` using `ProjectCard` component.
- `src/components/ProjectCard.jsx` — Reusable project card component. Renders individual projects with dynamic sections based on `sections` array from JSON.
- `src/components/CardSection.jsx` — Modular card section renderer for `keyFeatures`, `technologies`, and `quickStart`. All content data-driven from JSON.
- `src/components/MediumArticles.jsx` — Medium articles rendering from `mediumArticles.json`.
- `src/components/Education.jsx` — Education section rendering from `education.json`.
- `src/components/Footer.jsx` — Footer using `config.personal.name` and links.
- `src/main.jsx` — React bootstrap.

**Other:**

- `src/index.css` — Tailwind directives plus custom CSS (`.glass-card`, gradients, responsive tweaks).
- `vite.config.js` — Vite config; `base` is set to `/sritaj-portfolio/`. Update if repo name changes.
- `package.json` — Scripts and dependencies. Includes `gh-pages` for deployment.
- `tailwind.config.cjs` / `postcss.config.cjs` — Tailwind + PostCSS integration.
- `scripts/update-medium.js` — Auto-fetch Medium articles (optional).
- `CHECKLIST.md` — Visual and styling adjustments; useful for pixel-perfect tweaks.

Architecture & Patterns

- **Single-page app**: No routing library. Navigation is controlled by `activeSection` state in `App.jsx`.
- **Content-as-JSON**: ALL content lives in JSON files (`src/config.json` and `src/data/*.json`). ZERO hardcoded content. Components import JSON and render it.
  - `src/config.json` contains: personal info, titles, bio text, social links, theme settings.
  - `src/data/*.json` contain all section-specific content (about, awards, experience, etc.).
- **Data-driven components**: Each component receives JSON data as a prop from `App.jsx` and renders it. Examples:
  - `<About aboutMe={aboutMe} />` — passes `aboutMe.json` to About component.
  - `<Awards awards={awards} />` — passes `awards.json` to Awards component.
  - `<Experience experience={experience} />` — passes `experience.json` to Experience component.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sritajkumarpatel/sritaj-portfolio](https://github.com/sritajkumarpatel/sritaj-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
