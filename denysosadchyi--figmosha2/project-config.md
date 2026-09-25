---
trigger: always_on
description: Drive Figma by sending JS code through a local bridge that's connected to a custom plugin running inside Figma Desktop.
---

# Figmosha 2.0 — Claude Code instructions

Drive Figma by sending JS code through a local bridge that's connected to a custom plugin running inside Figma Desktop.

## How to send code

```bash
# Preferred — subcommand-style
python figmosha.py exec "return figma.currentPage.name"
python figmosha.py exec --file script.js

# Shorthand (auto-prepends `exec`)
python figmosha.py "return figma.currentPage.name"

# High-level commands (covered below) save tokens for common operations
python figmosha.py text 185:21880 "Привіт"
python figmosha.py variant 185:21883 "Property 1=Default"

# Quick HTTP (no Python needed)
curl -s -X POST http://localhost:8787/exec \
  -H 'Content-Type: application/json' \
  -d '{"code":"return figma.currentPage.id"}'

# Status
curl -s http://localhost:8787/status   # {"plugin_connected": true/false, "pending": 0}
```

If the bridge isn't running: `bash start-bridge.sh` (runs in tmux `figmosha-bridge`; logs at `/tmp/figmosha-bridge.log`).

If the plugin isn't connected: tell the user — `Plugins → Development → Figmosha Bridge → Run`.

## Helpers (available as `h.*` in every exec)

The plugin runtime exposes a small helper namespace. Use these to keep scripts short:

| Helper | What |
|---|---|
| `await h.bF(node, idx, varOrId)` | Bind fill paint to variable (id or instance) |
| `await h.bS(node, idx, varOrId)` | Bind stroke paint to variable |
| `await h.bN(node, prop, varOrId)` | Bind numeric prop (radius, padding, size...) |
| `h.findByName(root, name)` | First descendant by exact name |
| `h.findAllByName(root, name)` | All descendants by exact name |
| `h.dumpTree(node, {maxDepth, showSize, showText, showLayout})` | Indented tree string |
| `await h.withFonts(root, asyncFn)` | Loads every unique font in subtree, then runs `asyncFn` |
| `await h.setText(node, text)` | Set TEXT node chars with auto font load |
| `h.cloneNext(node, {direction, gap, name})` | Clone + place adjacent (`right`/`left`/`up`/`down`) |
| `await h.variant(instance, props)` | Wrapper around `instance.setProperties(...)` |
| `await h.variantsOf(instance)` | `{ current, groups, all }` for the component set |
| `h.sel()` | Currently selected nodes as `{id,name,type,w,h}` |
| `h.resolve(idOrAlias)` | Node by id, or the aliases `page` / `sel` |
| `h.hex("#1a2b3c")` | Hex to Figma's 0..1 `{r,g,b}` |
| `h.solid("#1a2b3c", opacity?)` | Ready-to-assign paint array |
| `h.frame(parent, opts)` | Frame with auto-layout applied in the right order |
| `await h.node(id)` | Shorthand for `figma.getNodeByIdAsync(id)` |
| `await h.var_(idOrKey)` | Resolve a variable from instance, local id, or library key |
| `await h.importComp(key)` | `figma.importComponentByKeyAsync(key)` |
| `await h.importVar(key)` | `figma.variables.importVariableByKeyAsync(key)` |

**Use them.** Compared to inline boilerplate, helpers save ~70% of the script and avoid common mistakes (frozen `node.fills`, missing `loadFontAsync`, etc.).

### Bad vs good

```js
// Bad — verbose, easy to miss
const f = JSON.parse(JSON.stringify(node.fills));
f[0] = figma.variables.setBoundVariableForPaint(f[0], "color", v);
node.fills = f;

// Good — helper handles freezing + setBoundVariableForPaint
await h.bF(node, 0, v);
```

```js
// Bad — must remember to load fonts first; mixed-font case is silent
await figma.loadFontAsync(node.fontName);
node.characters = "new";

// Good
await h.setText(node, "new");
```

```js
// Bad — manual font collection
const texts = root.findAll(n => n.type === "TEXT");
const fonts = [...new Set(texts.map(t => `${t.fontName.family}|${t.fontName.style}`))];
// ... load each ...

// Good
await h.withFonts(root, async () => {
  // bulk-edit text inside `root` here
});
```

## CLI subcommands (save tokens for common ops)

| Command | Equivalent JS | Use case |
|---|---|---|
| `figmosha doctor` | — | Diagnose bridge → plugin → Figma, with the fix for each break |
| `figmosha sel` | `h.sel()` | What the user has selected right now |
| `figmosha tree <id>` | `h.dumpTree(await h.node(id))` | Explore node structure |
| `figmosha find <id> name=Button` | `(await h.node(id)).findAll(n => n.name === "Button")` | Locate by name |
| `figmosha find <id> name~Btn` | `findAll(n => n.name.includes("Btn"))` | Substring name match |
| `figmosha find <id> type=INSTANCE` | `findAll(n => n.type === "INSTANCE")` | Filter by type |
| `figmosha find <id> text~Привіт` | `findAll(n => n.type === "TEXT" && n.characters.includes(...))` | Find by text |
| `figmosha text <id> "новий"` | `await h.setText(n, "новий")` | Edit text safely |
| `figmosha variant <id> "Property 1=Default"` | `await n.setProperties({...})` | Switch variant |
| `figmosha clone <id> --right --gap 100` | `h.cloneNext(n, {direction:'right',gap:100})` | Duplicate adjacent |
| `figmosha rm <id> [<id>…]` | `n.remove()` | Delete one or more |
| `figmosha icomp <key>` | `(await h.importComp(key)).createInstance()` | Pull from library |

Anywhere an id is taken, `page` and `sel` work too — `figmosha tree sel --layout`
dumps the selected subtree without hunting for its id first.

Use subcommands when the op fits one of these. Fall back to `exec` for anything else.

When the user says "this frame" or "the selected one", call `figmosha sel` — don't

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [denysosadchyi/figmosha2](https://github.com/denysosadchyi/figmosha2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
