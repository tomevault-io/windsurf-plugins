---
trigger: always_on
description: |
---


# Pretty Mermaid

Render stunning, professionally-styled Mermaid diagrams with one command. Supports SVG for web/docs and ASCII for terminals.

## Quick Start

### Render a Single Diagram

**From a file:**
```bash
node scripts/render.mjs \
  --input diagram.mmd \
  --output diagram.svg \
  --format svg \
  --theme tokyo-night
```

**From user-provided Mermaid code:**
1. Save the code to a `.mmd` file
2. Run the render script with desired theme

### Batch Render Multiple Diagrams

```bash
node scripts/batch.mjs \
  --input-dir ./diagrams \
  --output-dir ./output \
  --format svg \
  --theme dracula \
  --workers 4
```

### ASCII Output (Terminal-Friendly)

```bash
node scripts/render.mjs \
  --input diagram.mmd \
  --format ascii \
  --use-ascii
```

---

## Workflow Decision Tree

**Step 1: What does the user want?**
- **Render existing Mermaid code** → Go to [Rendering](#rendering-diagrams)
- **Create new diagram** → Go to [Creating](#creating-diagrams)
- **Apply/change theme** → Go to [Theming](#theming)
- **Batch process** → Go to [Batch Rendering](#batch-rendering)

**Step 2: Choose output format**
- **SVG** (web, docs, presentations) → `--format svg`
- **ASCII** (terminal, logs, plain text) → `--format ascii`

**Step 3: Select theme**
- **Dark mode docs** → `tokyo-night` (recommended)
- **Light mode docs** → `github-light`
- **Vibrant colors** → `dracula`
- **See all themes** → Run `node scripts/themes.mjs`

---

## Rendering Diagrams

### From File

When user provides a `.mmd` file or Mermaid code block:

1. **Save to file** (if code block):
   ```bash
   cat > diagram.mmd << 'EOF'
   flowchart LR
       A[Start] --> B[End]
   EOF
   ```

2. **Render with theme**:
   ```bash
   node scripts/render.mjs \
     --input diagram.mmd \
     --output diagram.svg \
     --theme tokyo-night
   ```

3. **Verify output**:
   - SVG: Open in browser or embed in docs
   - ASCII: Display in terminal

### Output Formats

**SVG (Scalable Vector Graphics)**
- Best for: Web pages, documentation, presentations
- Features: Full color support, transparency, scalable
- Usage: `--format svg --output diagram.svg`

**ASCII (Terminal Art)**
- Best for: Terminal output, plain text logs, README files
- Features: Pure text, works anywhere, no dependencies
- Usage: `--format ascii` (prints to stdout)
- Options:
  - `--use-ascii` - Use pure ASCII (no Unicode)
  - `--padding-x 5` - Horizontal spacing
  - `--padding-y 5` - Vertical spacing

### Advanced Options

**Custom Colors** (overrides theme):
```bash
node scripts/render.mjs \
  --input diagram.mmd \
  --bg "#1a1b26" \
  --fg "#a9b1d6" \
  --accent "#7aa2f7" \
  --output custom.svg
```

**Transparent Background**:
```bash
node scripts/render.mjs \
  --input diagram.mmd \
  --transparent \
  --output transparent.svg
```

**Custom Font**:
```bash
node scripts/render.mjs \
  --input diagram.mmd \
  --font "JetBrains Mono" \
  --output custom-font.svg
```

---

## Creating Diagrams

### Using Templates

**Step 1: List available templates**
```bash
ls assets/example_diagrams/
# flowchart.mmd  sequence.mmd  state.mmd  class.mmd  er.mmd
```

**Step 2: Copy and modify**
```bash
cp assets/example_diagrams/flowchart.mmd my-workflow.mmd
# Edit my-workflow.mmd with user requirements
```

**Step 3: Render**
```bash
node scripts/render.mjs \
  --input my-workflow.mmd \
  --output my-workflow.svg \
  --theme github-dark
```

### Diagram Type Reference

For detailed syntax and best practices, see [DIAGRAM_TYPES.md](references/DIAGRAM_TYPES.md).

**Quick reference:**

**Flowchart** - Processes, workflows, decision trees
```mermaid
flowchart LR
    A[Start] --> B{Decision}
    B -->|Yes| C[Action]
    B -->|No| D[End]
```

**Sequence** - API calls, interactions, message flows
```mermaid
sequenceDiagram
    User->>Server: Request
    Server-->>User: Response
```

**State** - Application states, lifecycle, FSM
```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Loading
    Loading --> [*]
```

**Class** - Object models, architecture, relationships
```mermaid
classDiagram
    User --> Post: creates
    Post --> Comment: has
```

**ER** - Database schema, data models
```mermaid
erDiagram
    USER ||--o{ ORDER : places
    ORDER ||--|{ ORDER_ITEM : contains
```

### From User Requirements

**Step 1: Identify diagram type**
- **Process/workflow** → Flowchart
- **API/interaction** → Sequence
- **States/lifecycle** → State
- **Object model** → Class
- **Database** → ER

**Step 2: Create diagram file**
```bash
cat > user-diagram.mmd << 'EOF'
# [Insert generated Mermaid code]
EOF
```

**Step 3: Render and iterate**
```bash
node scripts/render.mjs \
  --input user-diagram.mmd \
  --output preview.svg \
  --theme tokyo-night

# Review with user, edit diagram.mmd if needed, re-render
```

---

## Theming

### List Available Themes

```bash
node scripts/themes.mjs
```

**Output:**
```
Available Beautiful-Mermaid Themes:

 1. zinc-light
 2. zinc-dark
 3. tokyo-night
 4. tokyo-night-storm
 5. tokyo-night-light
 6. catppuccin-mocha
 7. catppuccin-latte
 8. nord
 9. nord-light
10. dracula
11. github-dark
12. github-light
13. solarized-dark
14. solarized-light
15. one-dark

Total: 15 themes
```

### Theme Selection Guide

**For dark mode documentation:**
- `tokyo-night` ⭐ - Modern, developer-friendly
- `github-dark` - Familiar GitHub style

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imxv/Pretty-mermaid-skills](https://github.com/imxv/Pretty-mermaid-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
