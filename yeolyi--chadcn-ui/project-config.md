---
trigger: always_on
description: A parody UI library based on shadcn/ui. Same components, same API, terrible UX
---

# chadcn/ui

A parody UI library based on shadcn/ui. Same components, same API, terrible UX
disguised as legitimate features.

## Stack

- **Astro 5** (SSG) + React islands for interactive demos
- **Tailwind v4** via `@tailwindcss/vite`
- **shiki** for code blocks (custom `CodeBlock.astro`, dual theme via media query)
- **MDX** content collections for variant page bodies
- **Radix UI** primitives in `packages/ui`
- **i18n**: Korean default at `/`, English at `/en/` (`prefixDefaultLocale: false`)
- **Theme**: OS preference only — no toggle
- **lucide-react** for all icons

## Project layout

```
packages/
  ui/                              # @chadcn/ui npm package
    src/
      button/
        base.tsx                   # CVA + Radix Slot, no chad behavior
        shy.tsx                    # variant: dodges hover (current escape rename)
        # <slug>.tsx               # one file per variant
      lib/utils.ts                 # cn()
      index.ts                     # top-level Button = first/canonical variant
    tsup.config.ts                 # multi-entry: dist/index + dist/<comp>/<slug>
  docs/                            # Astro site
    src/
      content/
        pages/                     # MDX page bodies (button.<locale>.mdx)
          button.snippets.ts       # code shown in <CodeBlock> (see Footguns)
        config.ts                  # zod schema, custom generateId for .ko/.en
      i18n/
        strings.ts                 # ko + en chrome labels
        locale.ts                  # getLocale, tFor helpers
      layouts/                     # Site, Docs
      pages/                       # ko (default) + en/ mirror, body inline
      components/
        chrome/                    # Site header/footer, mobile nav, icons
        docs/                      # Sidebar, Toc, ExamplePreview, InstallTabs,
                                   #   VariantSelector, CodeBlock
        ui/                        # shadcn primitives (Button) used by chrome
        demos/<comp>/              # variant runtime demos (take {slug})
      lib/
        button-components.ts       # slug → component + variantInfo (tagline,
                                   #   description per locale)
.github/workflows/publish.yml      # OIDC trusted publish on v* tag push
```

## Tone

The single most important rule, and the only one not enforceable by code.

**Every variant is presented as if it were a serious, useful feature.** The
tagline, description, and any prose read like a sincere product pitch — never
break the fourth wall.

- ❌ "도망가는 버튼" / "Catch me if you can"
- ✅ "잘못된 클릭을 막아주는 버튼" / "A button that prevents accidental clicks"

The `slug` (filename, dropdown label) can be playful (`shy`, `whack-a-mole`,
`jealous`) — that's for devs. The user-facing tagline + description must be
deadpan.

## Variant copy persona

Every `tagline` and `description` is written in one voice: **a serious,
methodical PM** who reframes each degraded UX as a thoughtfully engineered
safety, verification, fairness, or preservation feature. The defect is never
acknowledged as a defect.

### Voice rules

- **Subjects**: third-person only — `버튼은`, `사용자가`, `동작이`. No
  first/second person, no addressing the reader.
- **Framing**: reframe the gimmick as a benefit. Lean on a fixed pool of
  feature words — *검증, 보호, 보존, 공평, 신중, 의지, 인내, 가능성,
  활성화, 차단*.
- **Korean tense/mood**: declarative `합니다` throughout. No literary
  truncations (`...이다`, `...것이다`) or aphoristic prose.
- **English tense/mood**: simple present, third person, neutral product copy.
- **Cultural references in body text**: not allowed — *except* when the slug
  itself is the reference (`thanos`, `thanos2`, `benjamin`). The body may
  name it once; the rest of the entry stays in the PM voice (no exclamation,
  no fan-energy, no winking at the reader).
- **Prohibited**: exclamation points, puns on the slug, breaking the fourth
  wall, asking the reader anything.

### Tagline shape

- **Korean**: one sentence ending in `...버튼`. 10–25 chars. States the
  headline benefit, not the mechanism.
- **English**: starts with `A button [that|where|whose|...]`. 4–12 words.

### Description shape

- 1–3 sentences. Mechanism first, benefit framing second.
- **Korean**: 30–120 chars, `합니다` endings throughout the entry.
- **English**: 8–30 words, plain present tense.
- Plain text only — no markdown, no links, no quotes.
- ko/en semantically equivalent, idiomatic per language, never a direct
  translation.

### Slug shape

- Single English word, lowercase `[a-z]+`. Evokes behavior or mood; not a
  literal description.
- Numeric suffix permitted *only* to group a paired variant (`thanos` /
  `thanos2`). One pair max.
- Avoid Korean romanization (`baljak` is the lone holdout and should be
  renamed when convenient).

## Adding a new Button variant

Mechanical, ~10 minutes:

1. **Library component**: `packages/ui/src/button/<slug>.tsx`
   - Import `ButtonBase` from `./base` so the new variant inherits CVA + Radix
     Slot. Add chad behavior via event handlers / style.
   - Add `"use client"` at top (uses hooks).
2. **Library exports**:
   - `packages/ui/tsup.config.ts` — add `"button/<slug>": "src/button/<slug>.tsx"`
   - `packages/ui/package.json#exports` — add `./button/<slug>` block

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yeolyi/chadcn-ui](https://github.com/yeolyi/chadcn-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
