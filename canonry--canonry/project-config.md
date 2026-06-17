---
trigger: always_on
description: The web dashboard follows a dark, professional analytics aesthetic inspired by **Vercel's design system** — clean, minimal, high-contrast, and information-dense. Rival tools like Semrush, Ahrefs, and Profound for data richness, but match Vercel for polish: generous whitespace, sharp typography, subtle borders, no visual noise. Follow these conventions for all UI work:
---

@AGENTS.md

## UI Design System

The web dashboard follows a dark, professional analytics aesthetic inspired by **Vercel's design system** — clean, minimal, high-contrast, and information-dense. Rival tools like Semrush, Ahrefs, and Profound for data richness, but match Vercel for polish: generous whitespace, sharp typography, subtle borders, no visual noise. Follow these conventions for all UI work:

### Layout
- **Sidebar navigation** (persistent left, `w-56`, hidden on mobile with full-screen overlay fallback).
- **Compact topbar** with breadcrumb, health pills, and primary action button.
- **Page container** (`max-w-6xl`, centered) for all page content.
- Pages use a `page-header` (title + subtitle + optional actions) followed by sections separated by `page-section-divider`.

### Color & Theme
- Background: `bg-zinc-950`. Cards/surfaces: `bg-zinc-900/30` with `border-zinc-800/60`.
- Font: **Geist Sans** (400–800 weights) for UI text, **Geist Mono** for code/numerics. Globally enabled OpenType features `cv11`, `ss01`, `ss03` for sharper i/l/I/0 disambiguation. Headings tighten tracking (`-0.015em`, `-0.02em` on h1). `text-zinc-50` primary, `text-zinc-400` secondary, `text-zinc-500`/`text-zinc-600` for labels.
- Tone colors: **positive** = emerald, **caution** = amber, **negative** = rose, **neutral** = zinc.
- No decorative background gradients. Keep it clean and flat.

### Components & Patterns
- **AEO performance hero + metric cards:** the project overview leads with the AEO performance hero — three paired Mention / Cited / Mention-share rows with linear progress bars (stacking below `480px`) — followed by secondary metric cards in a `sm:grid-cols-2 lg:grid-cols-3` grid. Linear bars beat stacked radials when several numbers are read against each other. Keep a single `.metric-grid` / `.metric-card` definition; a duplicate once overrode the column count.
- **Score gauges** (`ScoreGauge`): SVG radial progress rings for a single numeric/text metric (e.g. traffic-source detail). Don't rebuild the overview header as a gauge cluster.
- **Data tables** for evidence, findings, and competitors (not card grids). Tables are more scanable for analysts.
- **Insight cards** with left-border accent color based on tone (`insight-card-positive`, `insight-card-caution`, `insight-card-negative`).
- **Sparklines** for inline trend visualization in overview project rows.
- **ToneBadge** for all status/state indicators. Map tones through helper functions (`toneFromRunStatus`, `toneFromCitationState`, etc.).
- **Filter chips** use `rounded-full` pill style.
- **Health pills** in topbar use `rounded-full` with tone-colored borders.

### Sidebar
- Main nav items use Lucide icons (`LayoutDashboard`, `Globe`, `Play`, `Settings`).
- Projects section shows each project with a colored dot indicating visibility health tone.
- Resources section at bottom with `Rocket` icon for Setup.
- Doc links in sidebar footer.

### Data Density
- Prioritize information density. Analysts want to scan, not scroll through cards.
- Use tables for any list of 3+ structured items (evidence, findings, competitors).
- Use cards only for insights/interpretations where narrative matters.
- Keep eyebrow labels (`text-[10px]`, uppercase, tracking-wide) for section context.

### Text & Tooltips
- **Heavy text belongs in tooltips, not inline.** A data surface shows values, one-line captions, and eyebrow labels — not prose. Multi-sentence explanations (methodology, "what this means", the evidence behind a finding) push the numbers down and break the analyst's scan, so they move into an `InfoTooltip` (`components/shared/InfoTooltip.tsx`) on the relevant heading, label, or row title. The trigger is a real keyboard-reachable button and the copy rides its `aria-label`, so nothing is lost for assistive tech or for tests (`getByRole('button', { name })`).
- **What may stay inline:** the metric value itself, a single-line caption/subtitle, eyebrow section labels, and **empty / onboarding states** (which must instruct — a "connect this integration" empty state is the only content, not heavy text).
- **The test:** if a sentence explains or justifies rather than labels or names, it goes in a tooltip. The section heading gets the info icon; the descriptive paragraph under it should not exist.

### Accessibility
- Skip-to-content link.
- `aria-current="page"` on active nav items.
- `aria-label` on nav landmarks.
- Focus-visible rings on interactive elements.
- Screen-reader-only labels (`.sr-only`) where needed.

### Charting (Critical)

**Recharts is the only charting library.** All charts must use it via `ChartPrimitives.tsx` — never import `recharts` directly in page/section components and never add Chart.js, Highcharts, D3, Plotly, Nivo, or Victory. ESLint enforces this.

- Import chart components and shared constants from `components/shared/ChartPrimitives.js`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Canonry/canonry](https://github.com/Canonry/canonry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
