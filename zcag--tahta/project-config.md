---
trigger: always_on
description: <!-- GENERATED from layouts.json + variants.json by scripts/gen-agents.mjs — do not edit by hand. -->
---

<!-- GENERATED from layouts.json + variants.json by scripts/gen-agents.mjs — do not edit by hand. -->
# tahta — authoring contract for agents

Generate a Slidev deck with `slidev-theme-tahta`. **No CSS, `<style>`, grids, or layout HTML** — pick a `layout` per slide and fill its frontmatter. The theme renders kicker, title, footer (auto page numbers), background, type, color, spacing, and motion.

## Rules
1. Pick the layout that matches the content shape; fill its frontmatter fields. Do not write CSS, grids, or layout HTML.
2. One idea per slide.
3. Titles/subtitles may contain <span class="accent2">highlight</span> (accent color) or <em>highlight</em> (italic accent — the editorial emphasis). Nothing else needs HTML.
4. For varied, designed decks vary the composition: open with lead, punctuate with bigtype, and use metric/agenda/define/columns/panels/reference/vs for teaching content — not every slide as a centered title+body.
5. Choose a variant deliberately. themeConfig.variant is required — pick the one whose 'best for' fits the talk's tone, audience, and medium (see the Variants table); never omit it to coast on a default. When you hand back the deck, tell the user which variant you chose and why, and that they can switch it — the visual direction is a shared decision.
6. Layouts are your default — the design is built in, so reach for the one that matches the content shape (definition→define, comparison→vs, numbers→stats, process→steps). But don't force every slide into a preset: default/two-cols bodies (and the space under a statement) are a canvas — compose components there (<Callout>, <Stat>, <Plot>, <Terminal>, <Kbd>, <Figure>, <Meter>, <Tags>, …) to enrich a slide, and reach for them whenever no layout cleanly fits. A deck that mixes designed layouts with a few composed slides reads richer than one that only fills templates.
7. Use components liberally AND with variety — they are the main source of richness and make each deck feel bespoke rather than templated. Map content to the component it invites: a number→<Stat>/<Meter>, a status→<Badge>, a stack/skill list→<Tags>, a person→<Person>, a command/artifact→<Terminal>/<Plot>/<Kbd>, an aside→<Callout>. Aim for a component on most content slides, and reach for several different ones across the deck — leaning on a single type (e.g. only <Callout>) still reads templated. A deck where nearly every slide carries a fitting component reads custom; one that only fills layout fields reads generic.
8. Fence every slide's frontmatter with --- above and below it (--- / layout: … / ---); the separator between two slides is just the first slide's closing --- followed by the next slide's opening ---. Most tahta slides are body-less (cover, section, statement, stats, define, steps, panels, vs, metric, end…), so between two of them you'll see the closing --- then the next opening --- — two --- lines, and that is correct. Two mistakes break a deck: (a) sharing ONE --- between two frontmatter blocks — bare layout:/title: keys right after a --- with no opening fence — which makes Slidev render those keys as body text and mis-parse the rest; (b) on a slide that HAS a markdown body, a stray --- after the body before the next slide's frontmatter ---, which doubles up into a blank slide. lint_deck / `npx tahta-lint` catch both.
9. The opening --- block is both the deck headmatter and the first slide's frontmatter — don't set the same key twice in it. A deck-level title plus the first slide's title collide (duplicate YAML key) and break slidev export; let the cover's title stand as the deck title, or start with a headmatter-only block and put the cover next.
10. Before finishing, validate the deck: run `npx tahta-lint slides.md` (ships with the theme) and fix what it reports — it catches empty slides, unclosed frontmatter, missing required fields, and bad enum values.
11. class: dropcap on a default slide sets a drop cap on the first paragraph.
12. Footer label auto-fills from the deck title (override per slide with foot:). Never add page numbers.
13. Keep numeric values bare; put the symbol in unit (value: 80, unit: "%").
14. For cover/section/statement/end/fact, the title comes from frontmatter — leave the slide body empty.
15. Inside { ... } flow rows, quote any value containing a comma or colon (before: "$4,200").
16. ghost: (on default/section/stats/steps/fact) prints a faint giant background glyph.
17. Entrance motion is automatic, themeable per variant, and disabled in print + reduced-motion.
18. Keep slides scannable, not prose: at most ~6 short bullets, written as phrases not sentences. If the idea is a structure or a flow (architecture, a pipeline, a data structure, who-calls-whom), reach for the `diagram` layout (themed Mermaid) instead of describing it in bullets; numbers→`stats`/`metric`, comparisons→`vs`/`compare`. lint_deck warns on dense slides (too many or too-long bullets, the same layout 3× in a row, an empty diagram).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zcag/tahta](https://github.com/zcag/tahta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
