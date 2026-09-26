---
trigger: always_on
description: Zero-dependency hand-drawn UI. Real HTML controls with SVG chrome. A unique seeded sketch per mount. Strokes boil in CSS.
---

# drawably

Zero-dependency hand-drawn UI. Real HTML controls with SVG chrome. A unique seeded sketch per mount. Strokes boil in CSS.

Install Inter yourself if you want the intended type. The library loads no font unless you import the optional `drawably/font.css`.

```
npm i drawably
```

## Vanilla

```js
import {
  drawablyButton,
  drawablyCheckbox,
  drawablyRadio,
  drawablyToggle,
  drawablyInput,
  drawablyTextarea,
  drawablySelect,
  drawablyDivider,
  drawablyCard,
  drawablyBadge,
  drawablyList,
  drawablyUnderline,
  drawablyHighlight,
  drawablyCircle,
  drawablyArrow,
} from "drawably";
import "drawably/style.css";

drawablyButton(document.querySelector("#done"), { variant: "solid" });
drawablyCheckbox(document.querySelector("#check")); // wrapper must contain <input type="checkbox">
drawablyRadio(document.querySelector("#pen")); // wrapper must contain <input type="radio">
drawablyToggle(document.querySelector("#tog")); // wrapper must contain <input type="checkbox">
drawablyInput(document.querySelector("#name")); // wrapper must contain <input>
drawablyTextarea(document.querySelector("#msg")); // wrapper must contain <textarea>
drawablySelect(document.querySelector("#pick")); // wrapper must contain <select>; reserves the widest option's width so picking never shifts layout
drawablyDivider(document.querySelector("#rule")); // <hr> or div
drawablyCard(document.querySelector("#card"));
drawablyBadge(document.querySelector("#tag"), { variant: "scribble" });
drawablyList(document.querySelector("#features"), { marker: "check" }); // <ul> or <ol>
drawablyUnderline(document.querySelector("#word")); // any inline element
drawablyHighlight(document.querySelector("#word"));
drawablyCircle(document.querySelector("#price"));
drawablyArrow(document.querySelector("#from"), document.querySelector("#to")); // two anchors
```

Each attacher throws if the element is missing. Checkbox/radio/toggle/input/textarea/select throw if the inner field is missing. Arrow throws if either anchor is missing. Returns a sketch: `{ resketch(seed?), destroy() }`. Buttons also have `setState(state)`.

## React

Optional peer. Subpath `"drawably/react"`. Client-only (uses `useEffect`).

```jsx
import {
  DrawablyButton,
  DrawablyCheckbox,
  DrawablyRadio,
  DrawablyToggle,
  DrawablyInput,
  DrawablyTextarea,
  DrawablySelect,
  DrawablyDivider,
  DrawablyCard,
  DrawablyBadge,
  DrawablyList,
  DrawablyUnderline,
  DrawablyHighlight,
  DrawablyCircle,
  DrawablyArrow,
} from "drawably/react";
import "drawably/style.css";

<DrawablyButton variant="solid" state="idle" onClick={submit}>Done</DrawablyButton>
<DrawablyButton tone="neutral">Cancel</DrawablyButton>
<DrawablyButton tone="danger">Delete</DrawablyButton>
<DrawablyCheckbox defaultChecked />
<DrawablyRadio name="ink" defaultChecked />
<DrawablyToggle />
<DrawablyInput placeholder="your name" />
<DrawablyTextarea rows={4} />
<DrawablySelect><option>Pen</option><option>Pencil</option></DrawablySelect>
<DrawablyDivider />
<DrawablyCard>…</DrawablyCard>
<DrawablyBadge variant="scribble">new</DrawablyBadge>
<DrawablyList marker="check"><li>…</li></DrawablyList>
<DrawablyUnderline>hand-drawn</DrawablyUnderline>
<DrawablyHighlight>fresh sketch</DrawablyHighlight>
<DrawablyCircle>$0</DrawablyCircle>
<DrawablyArrow from={fromRef} to={toRef} />
```

Native element props pass through. Sketch options are top-level props: `seed`, `roughness`, `boil`, `stroke`, `fill`, `paper`, `width`, plus button `variant`, `state`, and `tone`, badge `variant`, list `marker`. `DrawablyArrow` takes two refs and renders nothing. `DrawablyList` renders a `<ul>`.

## Button

`drawablyButton(el, opts)` → `ButtonSketch`

- `variant`: `"outline"` (default) | `"solid"` | `"scribble"`
- `state`: `"idle"` | `"loading"` | `"error"` | `"success"`
- `tone`: `"neutral"` (warm grey, secondary) | `"danger"` (red)
- `setState(state)` after mount. React: `state` prop.
- hover: lifts 1px and washes the inside with the stroke at 10% (outline/scribble); press: sinks and the outline thickens. Native `disabled` dims it and drops both.
- loading: dimmed, faster boil, `cursor: progress`
- error: `--drawably-error` (default `#d12724`)
- success: `--drawably-success` (default `#188a42`)

## Other controls

- `drawablyCheckbox(wrap, opts)` — checkbox in a wrapper
- `drawablyRadio(wrap, opts)` — radio in a wrapper; scribbled dot when checked. Same `name` groups them.
- `drawablyToggle(wrap, opts)` — checkbox in a wrapper; pill with a sliding ink-blob knob. React sets `role="switch"`.
- `drawablyInput(wrap, opts)` — text input in a wrapper
- `drawablyTextarea(wrap, opts)` — textarea in a wrapper; vertical resize redraws the sketch
- `drawablySelect(wrap, opts)` — select in a wrapper; native arrow hidden, sketched chevron in its place. Width is reserved for the widest option at attach, so changing the value never shifts layout (re-attach if options change). In Chromium the options popup gets a sketched frame too; other browsers show the OS popup.
- `drawablyDivider(el, opts)` — rough line on an `<hr>` or div
- `drawablyCard(el, opts)` — sketched container

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Danilaa1/drawably](https://github.com/Danilaa1/drawably) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
