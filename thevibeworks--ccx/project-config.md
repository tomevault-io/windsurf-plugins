---
trigger: always_on
description: Session viewer for Claude Code. Browse, inspect, search, and export conversations.
---

# ccx

Session viewer for Claude Code. Browse, inspect, search, and export conversations.

## Quick Start

```bash
ccx web                   # Start web UI at localhost:8080
ccx projects              # List all projects
ccx sessions              # List sessions for this workspace
ccx sessions --all        # List sessions across all projects
ccx view                  # View workspace session (interactive picker)
ccx export --format=html  # Export to HTML
```

## Core Principles

### Data Model: Trees, Not Logs

Sessions are **trees**, not flat message lists. This affects everything.

```
message {
  uuid: "abc-123"
  parentUuid: "xyz-789"      // tree structure
  isCompactSummary: true     // context compaction marker
  isSidechain: true          // agent branch
}
```

Key structures:
- `parentUuid` links child to parent (orphans become roots)
- `isCompactSummary` marks compaction boundaries
- `isSidechain` identifies agent task branches
- `agentId` links sidechain messages to agent session

### Read-Only by Design

ccx treats `CLAUDE_CODE_HOME` (~/.claude) as **read-only**. Never modify:
- Session JSONL files
- Claude Code config/settings
- Any Claude Code state

ccx writes only to its own directories:
- `$XDG_CONFIG_HOME/ccx/` - configuration
- `$XDG_DATA_HOME/ccx/` - SQLite (stars)

### Single Binary, Zero Dependencies

All CSS/JS embedded in Go templates. No CDN, no external files.
Pure Go (no CGO) enables true cross-platform compilation.

```bash
CGO_ENABLED=0 GOOS=darwin GOARCH=arm64 go build ./cmd/ccx
```

## Architecture

```
ccx/
  cmd/ccx/main.go        Entry point
  internal/
    cmd/                  Cobra commands
    parser/               JSONL parsing + tree building
    render/               Output formats (HTML, MD, Org, JSON)
    web/                  HTTP server + embedded templates
    db/                   SQLite persistence (stars)
    config/               CLAUDE_CODE_HOME logic
```

### Two-Level Parsing

**Quick parse** (session lists): O(lines), single pass
- Summary from first non-XML user message
- First/last timestamps
- Message/tool/sidechain counts
- Token usage stats

**Full parse** (session view): O(messages)
- Complete tree reconstruction via parentUuid
- All content blocks parsed
- Used only when viewing individual sessions

### Scanner Buffer

Tool results can exceed 64KB. Use 10MB max:

```go
scanner.Buffer(buf, 10*1024*1024)
```

## UI/UX Philosophy

### Clarity Over Cleverness

- Text labels > cryptic Unicode symbols
- Explicit behavior > implicit magic
- Users shouldn't guess what buttons do

```html
<!-- Bad -->
<button>↑</button>

<!-- Good -->
<button>↑ Prev</button>
```

### Terminal-Friendly

- ASCII-compatible where possible
- Keyboard navigation (j/k, /, z, r, d)
- Works without mouse

### Progressive Disclosure

- Large sessions (500+ messages): load last 3 sections initially
- "Load earlier" button for full history
- `?all=1` URL escape hatch

### Two Collapse Mechanisms

```javascript
// Thread fold: CSS class
thread.classList.toggle('folded');

// Details fold: HTML attribute
details.open = !details.open;
```

Search must handle both when auto-expanding.

## Styling

### Colors

```css
:root {
  /* Primary */
  --primary: #da7756;              /* warm terracotta */

  /* Context accents */
  --accent-project: #3b82f6;       /* blue-500 */
  --accent-session: #8b5cf6;       /* violet-500 */
  --accent-conversation: #06b6d4;  /* cyan-500 */
}
```

### Icons

- Geometric SVG icons, not emojis
- Consistent across header, toolbar, turns
- Info panel uses Unicode (kept for density)

### Depth Rendering

Cap visual indent at depth 3. Deeper messages get:
- Dashed left border
- `[depth:N]` badge

```css
.depth-1 { margin-left: 16px; }
.depth-2 { margin-left: 32px; }
.depth-3 { margin-left: 48px; }
.depth-max { border-left-style: dashed; }
```

## Code Quality

### Verify Before Code

1. Inspect actual DOM before writing selectors
2. Read API docs before using unfamiliar functions
3. Trace code path mentally: "What will this actually do?"

### API Selection

| Need              | Wrong                | Right                     |
|-------------------|----------------------|---------------------------|
| Viewport position | offsetTop            | getBoundingClientRect()   |
| Scroll to element | Manual math          | scrollIntoView()          |
| Refresh content   | location.reload()    | fetch() + innerHTML       |

### State Lifecycle

`location.reload()` kills ALL JavaScript state. Prefer fetch + DOM update.

| State Type      | Survives reload? | Preservation            |
|-----------------|------------------|-------------------------|
| Scroll position | No               | sessionStorage          |
| UI toggles      | No               | URL params / localStorage|
| Active section  | No               | URL hash                |

### Accuracy Over Features

If you can't guarantee accuracy, don't display it.

Example: We removed cost estimation and lines-changed tracking because:
- Cost: Claude Code calculates at runtime, not stored in JSONL
- Lines: Agent sidechains in separate `agent-*.jsonl` files

Token counts are accurate (from API `message.usage`), so we kept those.

## Security

### URL Sanitization


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thevibeworks/ccx](https://github.com/thevibeworks/ccx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
