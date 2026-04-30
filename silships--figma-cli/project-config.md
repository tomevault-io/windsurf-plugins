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
| "list variables" | `figma-cli var list` |
| "find nodes named X" | `figma-cli find "X"` |
| "what's on canvas" | `figma-cli canvas info` |
| "export as PNG/SVG" | `figma-cli export png` |
| "show all variants" | `figma-cli combos` |
| "create size variants" | `figma-cli sizes --base small` |
| "create a slot" | `figma-cli slot create "Name"` |
| "list slots" | `figma-cli slot list` |
| "reset slot" | `figma-cli slot reset` |
| "verify creation" | `figma-cli verify` |
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
2. **Never use `eval` to create** - no positioning, overlaps at (0,0)
3. **Never use `npx figma-use render`** - no smart positioning
4. **For multiple frames:** Use `render-batch`
5. **Convert to components:** `node to-component` after creation
6. **Never show terminal commands to users.** Run silently, give friendly feedback.
7. **NEVER delete existing nodes** - users may have components they want to keep!
8. **Always verify after creating:** `figma-cli verify "NODE_ID"`

---

## AI Verification

After creating any component, run `verify` to get a small screenshot for validation:

```bash
figma-cli verify              # Screenshot of selection
figma-cli verify "123:456"    # Screenshot of specific node
```

Returns JSON with base64 image (max 2000px). This is for internal AI checks, not shown to users.

---

## Blocks (Pre-built UI Layouts)

**ALWAYS use `blocks create` for dashboards and page layouts.** Never build them manually.

```bash
figma-cli blocks list                    # Show available blocks
figma-cli blocks create dashboard-01     # Create dashboard in Figma
```

**dashboard-01**: Full analytics dashboard (sidebar, stats cards, area chart, data table). All colors bound to shadcn variables (Light/Dark mode). Block source files: `src/blocks/`

---

## Design Tokens

```bash
figma-cli tokens preset shadcn   # 244 primitives + 32 semantic (Light/Dark)
figma-cli tokens tailwind        # 242 primitive colors only
figma-cli tokens ds              # IDS Base colors
figma-cli var delete-all         # Delete all variables
figma-cli var delete-all -c "primitives"  # Only specific collection
```

- `tokens preset shadcn` = Full system (primitives + semantic with Light/Dark mode)
- `tokens tailwind` = Just the Tailwind color palette (primitives only)
- `var list` only SHOWS variables. Use `tokens` commands to CREATE them.

---

## Variable Binding (var: syntax)

Use `var:name` to bind variables at creation time. Works with `render`, `create`, and `set` commands:

```bash
# JSX render
figma-cli render '<Frame bg="var:card" stroke="var:border" rounded={12} p={24}>
  <Text color="var:foreground" size={18}>Title</Text>
</Frame>'

# Create commands
figma-cli create rect "Card" --fill "var:card" --stroke "var:border"

# Set commands
figma-cli set fill "var:primary"
```

**Available shadcn variables:** `background`, `foreground`, `card`, `primary`, `secondary`, `muted`, `accent`, `border`, `input`, `ring`, and their `-foreground` variants.

---

## Connection Modes

**Yolo Mode (Recommended):** `figma-cli connect` - Patches Figma once, fully automatic.

**Safe Mode:** `figma-cli connect --safe` - Plugin-based, no Figma modification. Then: Plugins > Development > FigCli.

**Safe Mode caveat:** `render-batch` does NOT render text properly. Use `eval` with direct Figma API for components with text (see REFERENCE.md "Safe Mode Component Creation").

---

## JSX Syntax (render command)

```jsx
// Layout
flex="row"              // or "col"
gap={16}                // spacing
p={24}                  // padding all sides
px={16} py={8}          // padding x/y
pt={8} pr={16} pb={8} pl={16}

// Alignment
justify="center"        // main axis: start, center, end, between
items="center"          // cross axis: start, center, end

// Size
w={320} h={200}         // fixed
w="fill" h="fill"       // fill parent
minW={100} maxW={500} minH={50} maxH={300}

// Appearance
bg="#fff"               // fill color
bg="var:card"           // bind to variable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [silships/figma-cli](https://github.com/silships/figma-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
