---
trigger: always_on
description: Use this file as a `.cursorrules` attachment, system prompt, or `@SKILL.md` reference to teach an AI assistant how to work with the Objs library.
---

# Objs v2.0 — AI Skill File

Use this file as a `.cursorrules` attachment, system prompt, or `@SKILL.md` reference to teach an AI assistant how to work with the Objs library.

---

## Library basics

### Loading

```html
<!-- Browser -->
<script src="objs.js"></script>
```

```js
// npm / bundler — correct file chosen automatically by package.json exports
import o from 'objs-core'; // resolves to objs.built.js
```

### The `o()` function

```js
o('#id')           // → ObjsInstance wrapping all matching elements
o('.class')        // → ObjsInstance wrapping all matching elements
o(domElement)      // → ObjsInstance wrapping one DOM element
o([el1, el2])      // → ObjsInstance wrapping element array
o(2)               // → ObjsInstance from o.inits[2] (previously inited component)
o()                // → empty ObjsInstance, used to start init chains
o.first('#id')     // → ObjsInstance, single element, same as querySelector
self.select(e)     // → select the element in state action or render with self in the parameters, returns Objs instance with e.target (e.g. the row); then .refs, .el apply to that row
```

---

## Running Objs in Node (SSR)

In Node, **o.D** is **o.DocumentMVP** — there is no real `document` or `window`. You can run Objs in Node to render components to HTML (e.g. for SSR or for verification): use `o.init(states).render()`; the result is a tree of plain objects; serialize with the same SSR path the app uses (e.g. `o.D.parseElement`). To self-check generated Objs code, you can run a Node script that requires Objs, calls `o.init(...).render()`, and inspects the output or HTML string — no browser or user review required for structure verification. Call **`.html()`** (no arguments) on the rendered ObjsInstance to get that HTML string (in Node it uses `o.D.parseElement` under the hood).

---

## Component model

A component is a **states object** passed to `o.init()`. Every key becomes a method on the component instance.

### State object keys

| Key | Meaning |
|---|---|
| `name` | Component name string — used for `o.autotag` data attribute |
| `render` | Reserved: defines the DOM element to create (tag, attributes, html) |
| `tag` / `tagName` | HTML element type (default: `div`) |
| `html` / `innerHTML` | Inner HTML of the element |
| `class` / `className` | CSS class (`className` is a React-familiar alias) |
| `style` | Inline style string or object |
| `dataset` | Object of `data-*` attributes |
| `events` | Object of `{eventName: handler}` added on creation |
| any other key | HTML attribute set via `setAttribute` |

**ObjsInstance properties after init:**

| Property | Description |
|---|---|
| `refs` | Object of `{ name: ObjsInstance }` — auto-populated from `ref="name"` child elements on `init` |
| `store` | Plain object for storing child components and other per-instance data |

### State function signature

Every non-render state is a function:

```js
stateName: ({ self, o, i, parent }, data) => {
  // self   — the ObjsInstance (has all methods: .first(), .html(), .attr(), .on(), etc.)
  // o      — the o() function for creating new instances or querying the global DOM
  // i      — index of current element in multi-element instances
  // parent — the ObjsInstance this component was appendInside() into, or null
  // data   — argument passed when the state is called: component.stateName(data)
}
```

Inside a state function, `self` is the ObjsInstance. Use `self.first()`, `self.html()`, `self.attr()` etc. directly — no need to re-wrap it with `o(self)`.

### Creating a component

```js
const buttonStates = {
  name: 'SubmitButton',                          // sets data-qa="submit-button" if o.autotag = "qa"
  render: { tag: 'button', class: 'btn', html: 'Submit' },
  disable: ({ self }) => { self.attr('disabled', 'true'); },
  enable:  ({ self }) => { self.attr('disabled', null); },  // null removes the attribute
  setLabel:({ self }, text) => { self.html(text); },
};

const btn = o.init(buttonStates).render();  // creates the DOM element
btn.appendInside('#form');                  // inserts it
btn.disable();                              // calls the disable state
btn.setLabel('Saving...');                  // calls with data
```

### Shorthand for simple elements

```js
// Render a single element without explicit states object
o.initState({ tag: 'span', class: 'badge', html: '3' }).appendInside('.nav');
```

---

## Selecting and querying elements

```js
o('.card').first('h3').html('New title');    // find first h3 inside each .card, set text
o('.card').find('button').on('click', fn);   // find all buttons inside all .cards
o('.card').select(0).addClass('featured');   // operate only on first .card

const el = o.first('.card').el;             // raw DOM element
const els = o('.card').els;                 // raw DOM array
```

---

## Events

```js
o(component).on('click', handler);
o(component).on('click, mouseover', handler);     // multiple events
o(component).off('click', handler);
o(component).offAll();                            // remove all listeners

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [foggysq/objs](https://github.com/foggysq/objs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
