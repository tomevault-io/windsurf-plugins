---
trigger: always_on
description: Control Figma via CLI — create shapes, frames, text, components, set styles, layout, variables, export images. Use when asked to create/modify Figma designs or automate design tasks.
---


# figma-use

CLI for Figma. Two modes: commands and JSX.

```bash
# Commands
figma-use create frame --width 400 --height 300 --fill "#FFF" --layout VERTICAL --gap 16
figma-use create icon mdi:home --size 32 --color "#3B82F6"
figma-use set fill 1:23 "$Colors/Primary"

# JSX (props directly on elements, NOT style={{}})
echo '<Frame p={24} bg="#3B82F6" rounded={12}>
  <Text size={18} color="#FFF">Hello</Text>
</Frame>' | figma-use render --stdin --x 100 --y 100
```

## Before You Start

```bash
figma-use status  # Check connection
```

If not connected — start Figma with remote debugging:

```bash
# macOS
open -a Figma --args --remote-debugging-port=9222

# Windows
"%LOCALAPPDATA%\Figma\Figma.exe" --remote-debugging-port=9222

# Linux
figma --remote-debugging-port=9222
```

> Figma 126+ blocks remote debugging. Run `figma-use patch` once to fix, then restart Figma. Click **Always Allow** on the keychain prompt. Re-run after Figma updates.
>
> **Can't patch?** Use `figma-use daemon start --pipe` — launches Figma with debug pipe, no patching needed.

Start Figma with `--remote-debugging-port=9222` and you're ready.

## Two Modes

**Imperative** — single operations:

```bash
figma-use create frame --width 400 --height 300 --fill "#FFF" --radius 12
figma-use set fill <id> "#FF0000"
figma-use node move <id> --x 100 --y 200
```

**Declarative** — render JSX trees:

```bash
echo '<Frame p={24} gap={16} flex="col" bg="#FFF" rounded={12}>
  <Text size={24} weight="bold" color="#000">Title</Text>
  <Text size={14} color="#666">Description</Text>
</Frame>' | figma-use render --stdin --x 100 --y 200
```

stdin supports both pure JSX and full module syntax with imports:

```tsx
import { Frame, Text, defineComponent } from 'figma-use/render'

const Button = defineComponent(
  'Button',
  <Frame bg="#3B82F6" p={12} rounded={6}>
    <Text color="#FFF">Click</Text>
  </Frame>
)

export default () => (
  <Frame flex="row" gap={8}>
    <Button />
    <Button />
  </Frame>
)
```

**Elements:** `Frame`, `Rectangle`, `Ellipse`, `Text`, `Line`, `Star`, `Polygon`, `Vector`, `Group`, `Icon`, `Image`, `Instance`

Use `<Instance>` to create component instances:

```tsx
<Frame flex="row" gap={8}>
  <Instance component="59763:10626" />
  <Instance component="59763:10629" />
</Frame>
```

⚠️ **Always use `--x` and `--y`** to position renders. Don't stack everything at (0, 0).

## Icons

150k+ icons from Iconify by name:

```bash
figma-use create icon mdi:home
figma-use create icon lucide:star --size 48 --color "#F59E0B"
figma-use create icon heroicons:bell-solid --component  # as Figma component
```

In JSX:

```tsx
<Icon name="mdi:home" size={24} color="#3B82F6" />
```

## Images

Load images from URL:

```tsx
<Image src="https://example.com/photo.jpg" w={200} h={150} />
```

## Export JSX

Convert Figma nodes back to JSX code:

```bash
figma-use export jsx <id>           # Minified
figma-use export jsx <id> --pretty  # Formatted

# Format options
figma-use export jsx <id> --pretty --semi --tabs

# Match vector shapes to Iconify icons (requires: npm i whaticon)
figma-use export jsx <id> --match-icons
figma-use export jsx <id> --match-icons --icon-threshold 0.85 --prefer-icons lucide,tabler
```

Round-trip workflow:

```bash
# Export → edit → re-render
figma-use export jsx <id> --pretty > component.tsx
# ... edit the file ...
figma-use render component.tsx --x 500 --y 0
```

Compare two nodes as JSX:

```bash
figma-use diff jsx <from-id> <to-id>
```

## Export Storybook (Experimental)

Export all components on current page as Storybook stories:

```bash
figma-use export storybook                      # Output to ./stories/
figma-use export storybook --out ./src/stories  # Custom output dir
figma-use export storybook --match-icons        # Match vectors to Iconify icons
figma-use export storybook --no-semantic-html   # Disable semantic HTML conversion
```

**Semantic HTML:** By default, components are converted to semantic HTML elements based on their names:

- `Input/*`, `TextField/*` → `<input type="text">`
- `Textarea/*` → `<textarea>`
- `Checkbox/*` → `<input type="checkbox">`
- `Radio/*` → `<input type="radio">`
- `Button/*` → `<button>`
- `Select/*`, `Dropdown/*` → `<select>`

Use `--no-semantic-html` to disable this and keep `<Frame>` elements.

Generates `.stories.tsx` files:

- **ComponentSets** → React component with props + stories with args
- **VARIANT properties** → Union type props (`variant?: 'Primary' | 'Secondary'`)
- **TEXT properties** → Editable string props (`label?: string`)
- Components grouped by `/` prefix → `Button/Primary`, `Button/Secondary` → `Button.stories.tsx`

Example output for Button with variant and label:

```tsx
// Button.tsx
export interface ButtonProps {
  label?: string
  variant?: 'Primary' | 'Secondary'
}
export function Button({ label, variant }: ButtonProps) {
  if (variant === 'Primary')
    return (
      <Frame>
        <Text>{label}</Text>
      </Frame>
    )
  // ...
}

// Button.stories.tsx
export const Primary: StoryObj<typeof Button> = {
  args: { label: 'Click', variant: 'Primary' }
}
```

## Variables as Tokens


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dannote/figma-use](https://github.com/dannote/figma-use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
