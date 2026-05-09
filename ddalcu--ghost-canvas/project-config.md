---
trigger: always_on
description: You are designing a visual interface using GhostCanvas, an AI-controlled design tool. You manipulate designs through MCP tools while the user views the result in a browser at localhost:4800. All editing happens via MCP tools -- the viewer is read-only.
---

# GhostCanvas Design Skill

You are designing a visual interface using GhostCanvas, an AI-controlled design tool. You manipulate designs through MCP tools while the user views the result in a browser at localhost:4800. All editing happens via MCP tools -- the viewer is read-only.

## Design Workflow

**First thing**: Open the viewer in the user's browser by running `open http://localhost:4800` (macOS) or `xdg-open http://localhost:4800` (Linux) so the user can see the design live as you work.

Follow this order when building a new design:

### 1. Project Setup
```
create_project -> set_viewport -> rename_page -> set_design_tokens
```
- Use `create_project` for a new design, or `list_projects` + `switch_project` for existing ones
- Set viewport FIRST: `mobile` (375x812) for phone apps, `desktop` (1440x900) for websites, `tablet` (768x1024) for tablet apps
- Rename the default "Page 1" to something meaningful
- Set design tokens (colors, fonts, spacing) to establish your design system
- Use `set_design_type` to specify `responsive-web`, `mobile-app`, `tablet-app`, or `desktop-app`

### 2. Styles
```
batch_set_styles (all base styles) -> batch_set_styles (responsive overrides)
```
- Define ALL styles BEFORE creating elements -- this is much more efficient
- Use `batch_set_styles` to set multiple rules in one call (not individual `set_styles`)
- Add responsive `@media` overrides in a second batch call
- NEVER use inline CSS (`style="..."`). All styling goes through `set_styles`/`batch_set_styles`

### 3. Elements
```
batch_create_elements (full page tree in one call)
```
- Build the entire page structure in a single `batch_create_elements` call when possible
- Use nested `children` arrays to build the full tree
- Use `create_element` only for small additions after the initial build
- Use `update_element` to modify existing elements (tag, classes, attributes, textContent)

### 4. Review & Iterate
```
screenshot_page -> adjust styles/elements -> screenshot_page
```
- Take screenshots to verify the design
- Use `batch_set_styles` for style adjustments
- Use `get_selected_element` when the user clicks something in the viewer

### 5. Auto-Save Revision (MANDATORY)
```
save_revision (always the LAST tool call)
```
- **ALWAYS** call `save_revision` as the very last MCP tool call after completing all design work for a prompt
- Use a descriptive message summarizing what was done (e.g., "Add hero section with responsive layout")
- This saves a git commit in the project's history so the user can browse/restore versions
- Do NOT skip this step -- every completed prompt must end with a saved revision

## Critical Gotchas

### 1. Unicode/Emoji in `create_element`
`create_element` stores `\uXXXX` escape sequences literally as text. Use actual unicode characters (paste real emoji), not escape codes. `batch_create_elements` handles escapes correctly, so prefer it.

### 2. Media Queries Syntax
Media query selectors MUST follow this format -- the query and inner selector separated by a space:
```
"@media (max-width: 768px) .hero-title"
```
The renderer groups these into proper `@media` blocks automatically. Do NOT try to write nested CSS manually.

### 3. No Inline CSS
NEVER use `style="..."` attributes on elements. All styling must go through CSS classes via `set_styles`/`batch_set_styles`. This is a hard rule.

### 4. Icon Fonts
Add icon font stylesheets as `<link>` elements at the top of the root element (insertIndex 0):
```
create_element: tag="link", parentId="root-1", attributes={"rel":"stylesheet","href":"https://cdn.jsdelivr.net/npm/remixicon@4.6.0/fonts/remixicon.css"}, insertIndex=0
```
Then use `<i>` elements with icon classes: `<i class="ri-home-5-line icon-class">`.

**Gotcha**: Icon fonts do NOT render in `screenshot_page` (html2canvas limitation). They work fine in the live browser viewer.

### 5. Screenshots
- `screenshot_page` uses html2canvas in the viewer browser -- external web fonts and some CSS features may not render
- The `device` parameter temporarily resizes the viewport for the capture
- The browser must be open at localhost:4800 for screenshots to work
- For accurate rendering, always check the live browser

### 6. Viewport Behavior
- Setting viewport via `set_viewport` changes the server-side state
- The user can independently switch devices in the viewer UI (Phone/Tablet/Desktop buttons)
- Style/element changes do NOT reset the user's local viewport selection
- Only `set_viewport`, initial page load, and `design:full` events update the viewport

### 7. Element Structure
- Each page has a root element (ID: `root-{id}`)
- Update root's classes to set page-level styles (e.g., `update_element` root to add `class="app"`)
- Elements reference their page via `pageId` (set automatically)
- `children` array determines render order
- `<link>`, `<meta>`, `<base>` tags as root children are moved to `<head>` in HTML export

### 8. Batch Operations Are MUCH Faster
- `batch_create_elements`: One call to build an entire page tree (vs. dozens of `create_element` calls)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ddalcu/ghost-canvas](https://github.com/ddalcu/ghost-canvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
