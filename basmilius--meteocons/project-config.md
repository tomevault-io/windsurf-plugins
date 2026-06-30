---
trigger: always_on
description: Monorepo voor Meteocons: animated weather icons. Bevat de export pipeline, publishable icon packages en documentatie.
---

# Meteocons

Monorepo voor Meteocons: animated weather icons. Bevat de export pipeline, publishable icon packages en documentatie.

## Monorepo structuur

```
packages/
├── exporter/     # @meteocons/exporter  — Figma → SVG/Lottie export pipeline
├── svg/          # @meteocons/svg       — Publishable animated SVG icon package
├── svg-static/   # @meteocons/svg-static — Publishable static SVG icon package (no SMIL animations)
├── lottie/       # @meteocons/lottie    — Publishable Lottie icon package
└── docs/         # @meteocons/docs      — Astro docs + marketing website
```

## Commando's

```bash
bun run fetch              # Haal SVGs op uit Figma (gebruikt cache)
bun run fetch --force      # Forceer opnieuw downloaden
bun run export             # Exporteer alle iconen (SVG + Lottie)
bun run export --frame X   # Exporteer één icoon
bun run validate           # Valideer layer-namen en coverage
bun run publish-icons      # Kopieer output naar @meteocons/svg, @meteocons/svg-static en @meteocons/lottie
bun run docs:dev           # Start docs website dev server
```

## Architectuur

```
Figma → fetch → packages/exporter/.cache/svgs/   (statische SVGs)
                    ↓
animations/configs/*.json                         (declaratieve animatie-definitie)
animations/partials/*.json                        (herbruikbare bouwstenen)
                    ↓
config-loader.ts → resolveConfig                  (merge includes + overrides)
                    ↓
svg/processor.ts → SVG <animate>                  (SVG animate/animateTransform injectie)
lottie/generator.ts → Lottie JSON                 (volledige SVG traversal + keyframes)
                    ↓
packages/exporter/output/{style}/svg/*.svg
packages/exporter/output/{style}/lottie/*.json
                    ↓
publish-icons.ts → packages/svg/{style}/*.svg          (animated als beschikbaar, anders static)
                 → packages/svg-static/{style}/*.svg   (altijd static, met snapshot transforms)
                 → packages/lottie/{style}/*.json
```

## Animatie JSON configs

Animaties worden gedefinieerd in `animations/configs/*.json`. Elke config heeft `targets` (welke iconen) en `layers` (welke elementen animeren). Herbruikbare bouwstenen staan in `animations/partials/*.json` en worden via `includes` gemerged.

### Voorbeeld config

```json
{
  "targets": ["partly-cloudy-day-rain", "overcast-day-rain"],
  "includes": ["sun", "clouds", "rain"],
  "overrides": {
    "line": {
      "layers": {
        "Cloud Mask": { "transform": "translateY", "values": [0, -3, 0], "duration": "4s" }
      }
    }
  }
}
```

### Animatie formaat

**Transform animatie:**
```json
{ "transform": "rotate|translateX|translateY|scale|scaleY", "values": [...], "duration": "4s", "easing": "ease-in-out", "origin": "center|bottom|top" }
```

**Property animatie:**
```json
{ "property": "opacity|stroke-dashoffset|y|height", "values": [...], "duration": "2s" }
```

**Samengesteld (meerdere animaties op één element):**
```json
{ "animations": [{ "transform": "translateY", ... }, { "property": "opacity", ... }] }
```

### Speciale velden

- `origin: "bottom"` — schaalt vanuit de onderkant (maxY) van het element (voor water-animaties)
- `origin: "top"` — schaalt vanuit de bovenkant (minY) van het element
- `expandMask: true` — verwijdert viewport-restricties van parent `<mask>` (voor raindrop-measure)
- `dashArray: 50` — zet `stroke-dasharray` (voor wind dashflow)
- `keyTimes: [0, 0.15, 0.85, 1]` — niet-uniforme keyframe verdeling
- `delay: 0.4` — vertraging in seconden (voor stagger)
- `staticValue: 12` — override voor de statische SVG snapshot positie (alleen transforms, niet opacity)
- Wildcards in targets: `"code-*"` matcht code-green, code-yellow, etc.

## Exporter bestandsstructuur

```
packages/exporter/
├── src/
│   ├── fetch.ts              # Figma API → cache
│   ├── export.ts             # Hoofdscript: config laden → processor → output
│   ├── publish-icons.ts      # Kopieer output → svg/lottie packages
│   ├── config-loader.ts      # Laadt/merged JSON configs met includes + overrides
│   ├── validate.ts           # Layer naming lint + coverage rapport
│   ├── cache.ts              # .cache/ lezen/schrijven
│   ├── types.ts              # AnimationType, ElementMapping
│   ├── svg/
│   │   └── processor.ts      # Injecteert SVG <animate>/<animateTransform> elementen
│   ├── lottie/
│   │   ├── generator.ts      # Orchestrator: SVG → Lottie JSON
│   │   ├── animation-helpers.ts # Constants, easing, keyframe helpers
│   │   ├── gradients.ts      # SVG gradiënt → Lottie gradient fill
│   │   ├── shapes.ts         # SVG elementen → Lottie shape groups
│   │   ├── masks.ts          # Mask detectie, conversie en animatie
│   │   ├── types.ts          # Lottie JSON type definities
│   │   ├── colors.ts         # SVG kleur → Lottie RGBA
│   │   └── path-converter.ts # SVG path → Lottie bezier
│   ├── figma/
│   │   ├── client.ts         # Figma REST API client
│   │   └── types.ts          # Figma API types
│   └── utils/
│       └── bbox.ts           # Bounding box + center berekening
animations/                       # (root, buiten packages/exporter)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [basmilius/meteocons](https://github.com/basmilius/meteocons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
