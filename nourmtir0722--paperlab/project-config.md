---
trigger: always_on
description: Paperlab renders physical, realistic paper as a React component. Content is a texture on a mesh that genuinely bends — text and imagery curl with perfect continuity. This file is the dense reference for agents integrating Paperlab into a project or contributing to this repo.
---

# Paperlab — for coding agents

Paperlab renders physical, realistic paper as a React component. Content is a texture on a mesh that genuinely bends — text and imagery curl with perfect continuity. This file is the dense reference for agents integrating Paperlab into a project or contributing to this repo.

> Reading as a human rather than as an agent? The [reference](https://paperlab.nawwara.studio/docs/) is the same catalogue with everything rendering live.

## Integrating Paperlab into a project

```sh
npm i paperlab three @react-three/fiber gsap
```

Peer requirements: React ≥ 19, three ≥ 0.160. TypeScript types ship with the package.

```tsx
import { Paper } from 'paperlab'

// Simplest: a built-in preset. <Paper> owns its own <Canvas> and fills its
// parent — THE PARENT MUST HAVE A HEIGHT or the canvas renders 0px tall.
<Paper preset="receipt-unroll" />

// Or configure inline (all fields optional, validated by zod):
<Paper
  sheet={{ width: 1, height: 2.6, thickness: 0.3 }}
  stock="thermal"          // printer | thermal | kraft | newsprint | vellum | photo-gloss | sticker
  content={{ type: 'receipt', store: 'acme.dev', items: [{ name: 'Widget', price: 9.99 }] }}
  behavior={{ type: 'unroll', progress: 0.6, tightness: 0.5, sway: 0.3 }}
  surface={{ grain: 0.3, deckle: { edges: ['bottom'], roughness: 0.5 } }}
  physics="none"           // 'none' | float | tumble | dangle | taped | breeze | 'cloth' | 'strip' | {type:…}
  interactive              // drag handles / grab cloth
  autoplay                 // play the behavior loop on mount
/>
```

Inside an existing React Three Fiber scene use `<PaperMesh />` (same props, no Canvas). For galleries use `<PaperField />`:

```tsx
import { PaperField } from 'paperlab'

<PaperField
  images={['/a.jpg', '/b.jpg', '/c.jpg']}
  preset="photo-print"
  layout="ring"            // ring | fan | spread | pile | wall | spill | sweep | book | accordion | rack | colonnade | sheet
  layoutOptions={{ radius: 3, tiltDeg: 8 }}
  motion={{ driver: 'autoplay', speed: 0.5 }}   // autoplay | drag | none
  entrance={{ type: 'rise', stagger: 0.06 }}    // rise | scatter | none
/>
```

### Stage mode — paper as architecture

`<PaperStage>` builds a *space* out of paper: banners hung along a walk, with
a figure walking down it. It is the one mode where the paper is the room
rather than the object.

```tsx
import { PaperStage } from 'paperlab/stage'

<PaperStage
  text="the paper remembers every hand that folded it"   // split across banners, a line each
  count={18}
  stage={{
    path: getWalk('straight'),   // straight | bend | ess | ring | spiral
    shot: { shot: 'follow' },    // follow | lead | low | wide
    lighting: 'nave',            // stage mode is built for this one; the rest are front-lit
    light: { exposure: 0.9, direction: 180, height: 24, studio: 0.55 },  // overrides on the preset
    figure: { model: '/figure/walking.glb', finish: 'shaded' },          // your asset, your URL
    showFigure: true,
  }}
  progress={scrollProgress}      // 0..1 — omit it and the figure walks on its own clock
  quality="auto"                 // auto | low | medium | high — auto adapts to the machine
/>
```

### Content types

`blank`, `image`, `text`, `card`, `receipt` — and any of them can also sit on
the reverse via `content.back`.

**`card` is the paper-artifact type.** One composition — a tracked label, a
hairline rule, a body, and a line of small print — covering the index card,
the library due-date card, the museum wall label, the telegram slip and the
gallery quote sheet, because those are the same object with different parts
present. `{ title, body, note, rule, ruled, align, size, font, color,
padding }`. It exists because `text` sets a block of prose in one size and
one weight, and every artifact above is a *hierarchy*; composing one out of
plain text meant hand-placing newlines and hoping.

**`text` gained `tracking` and `valign`.** Tracking is the control display
type cannot do without — a line set to be read across a room needs it pulled
in, small uppercase needs it pushed out, and neither is reachable by changing
the size. `valign: 'center'` optically centres the block instead of hanging
it from the top edge, which is what a label or a poster wants and what a
letter does not.

**`image.src` may be empty**, and empty renders as bare stock rather than as
a failure. That is what lets `photo-print` and `postage-stamp` be image
presets without the library shipping — or fetching — a photograph; both are
containers for the caller's own art. **No built-in preset touches the network.**

### Lighting is data, not an enum

Eight rigs: `studio`, `window`, `leaves`, `goldenhour`, `noir`, `nave`, and
two built for paper as a material rather than as a surface to print on —

- **`raking`** — a hard key eight degrees above the horizon and well off to
  one side, so it skims ACROSS the sheet instead of landing on it. This is
  how paper is photographed for a swatch book, and it is the only rig that
  turns a crease, a fold or a crumple into relief rather than shading.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NourMtir0722/Paperlab](https://github.com/NourMtir0722/Paperlab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
