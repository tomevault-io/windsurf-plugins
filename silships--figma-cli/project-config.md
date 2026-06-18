---
trigger: always_on
description: CLI that controls Figma Desktop directly. No API key needed.
---

# figma-ds-cli

CLI that controls Figma Desktop directly. No API key needed.

## Quick Reference

| User says | Command |
|-----------|---------|
| "connect to figma" | `figma-cli connect` |
| "add shadcn colors" | `figma-cli tokens preset shadcn` |
| "add tailwind colors" | `figma-cli tokens tailwind` |
| "show colors on canvas" | `figma-cli var visualize` |
| "create dashboard" | `figma-cli blocks create dashboard-01` |
| "list blocks" | `figma-cli blocks list` |
| "create cards/buttons" | `render-batch` + `node to-component` |
| "create a rectangle/frame" | `figma-cli render '<Frame>...'` |
| "convert to component" | `figma-cli node to-component "ID"` |
| "use the existing X component" / "don't rebuild, instance it" | `figma-cli instantiate "X"` |
| "what component already exists for X" | `figma-cli spec X` (shows the reuse handle) |
| "list variables" | `figma-cli var list` |
| "find nodes named X" | `figma-cli find "X"` |
| "what's on canvas" | `figma-cli canvas info` |
| "export as PNG/SVG" | `figma-cli export png` |
| "extract gradient from image" / "rebuild this gradient" | `figma-cli gradient extract <image>` |
| "apply image gradient to a frame" | `figma-cli gradient extract <image> --apply-to <nodeId>` |
| "match this mesh / blossom / aurora background" | `figma-cli gradient extract <image> --mode mesh --apply-to <frameId>` |
| "create a wallpaper / mesh gradient from these colors" | `figma-cli gradient mesh "#a,#b,#c" --size 1920x1080` |
| "export the design system as markdown" / "create a DESIGN.md" | `figma-cli extract` |
| "export only the tokens" | `figma-cli extract --sections tokens` |
| "extract/document the X page" | `figma-cli extract --pages "X"` |
| "extract what I selected" | `figma-cli extract --selection` |
| "import my tailwind colors" | `figma-cli import tailwind.config.js` |
| "import our css variables" | `figma-cli import src/globals.css` |
| "import design tokens json" | `figma-cli import tokens.json` |
| "export tokens as DTCG / design tokens json" | `figma-cli export dtcg tokens.json` |
| "export tokens as CSS / Tailwind" | `figma-cli export css` / `figma-cli export tailwind` |
| "load our storybook" | `figma-cli import http://localhost:6006` |

**Wallpaper palette tip:** for rich results pass **5-6 hue-diverse colors** (mix warm + cool + a bright accent), not shades of one color. Analogous palettes blend into a flat 2-tone wash. The command auto-adds a depth anchor + focal glow, and `--style auto` rotates compositions (scatter/diagonal/bands/drift/spotlight/corners). For N wallpapers, run it N times with different palettes + styles. Add `--grain` for subtle film-grain NOISE or `--texture` for paper grain over the wallpaper.

**Liquid glass tip (Apple-style):** Figma's native `GLASS` effect (`glass={true}`) reproduces the STATIC optics of Apple Liquid Glass — edge-lensing/refraction (`glassDepth`), specular highlight (`glassLight`/`glassLightAngle`), chromatic dispersion (`glassDispersion`) — but NOT the live material (no motion/scroll adaptation). To make it read as liquid (not frosted): keep `glassRadius` LOW (clear) + `glassDepth` HIGH (strong rim lensing) + put **sharp, detailed content BEHIND** the glass so the lensing is visible. Over a smooth gradient with nothing behind it, any glass looks frosted. Best demo = real UI over a photo-like background (e.g. an iOS Control Center: glass tiles over a vivid wallpaper).
| "show all variants" | `figma-cli combos` |
| "create size variants" | `figma-cli sizes --base small` |
| "make these frames a variant set" / "combine into variants" | `figma-cli variants from <ids> --property Size --values Small,Medium,Large --name Button` |
| "combine existing components into a variant set" | `figma-cli prop combine <ids> --name Button` |
| "create a slot" | `figma-cli slot create "Name"` |
| "list slots" | `figma-cli slot list` |
| "reset slot" | `figma-cli slot reset` |
| "verify creation" | `figma-cli verify` (or render with `--verify` for one roundtrip) |
| "undo that" / "remove what you just made" | `figma-cli undo` |
| "check contrast" | `figma-cli a11y contrast` |
| "color blindness sim" | `figma-cli a11y vision` |
| "check touch targets" | `figma-cli a11y touch` |
| "check text a11y" | `figma-cli a11y text` |
| "full a11y audit" | `figma-cli a11y audit` |
| "show plugins" | `figma-cli plugins` |
| "install voice" | `figma-cli plugins install voice` |
| "setup voice key" | `figma-cli plugins setup voice` (in regular Terminal!) |
| "voice mode" | `figma-cli voice` |
| "chat with figma" | `figma-cli chat "build a button"` |

**Full command reference:** See REFERENCE.md

---

## Key Rules

1. **Always use `render` for frames** - has smart positioning
2. **Never use `eval` to create new visual nodes** - no positioning, overlaps at (0,0), and bypasses every safety guard (auto-split, name dedup, constraints, fills). `eval` is for Plugin API operations that have no equivalent CLI subcommand. Period.
3. **Never use `npx figma-use render`** - no smart positioning
4. **For multiple frames:** Use `render-batch`
5. **Convert to components:** `node to-component` after creation
6. **Never show terminal commands to users.** Run silently, give friendly feedback.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [silships/figma-cli](https://github.com/silships/figma-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
