---
trigger: always_on
description: Generates self-contained HTML/CSS animations of app features for walkthroughs, demos, and onboarding. Researches the target app via Chrome, interviews the user, generates a structured brief and animation HTML file, then enters an iterative freeze-inspect-feedback review loop until the user approves. Use when the user says "css animation", "animate this feature", "create a css walkthrough", "animation walkthrough", or wants to create a CSS-based visual demo of an app feature.
---


# CSS Animation Walkthrough Skill

You create polished, self-contained HTML/CSS animations that mimic real app features. These are used for walkthroughs, onboarding demos, and marketing. They are NOT GIFs — they are resolution-independent, tiny file size, and easy to iterate on.

## Two Animation Styles

1. **Feature Demo** — Before → Action → After. Shows a single feature transformation (e.g., clicking "Optimize" and watching seats rearrange).
2. **Carousel** — Multi-view. Cycles through several app screens with cross-fade transitions (e.g., Dashboard → Guest List → Canvas → RSVPs).

Determine which style the user needs during the Interview phase.

## Phases

```
Phase 1: Research → Phase 2: Interview → Phase 3: Generate → Phase 4: Review Loop
```

---

## Phase 1: Research

Before generating anything, deeply understand the target app's visual system by navigating it in Chrome.

### Prerequisites
- Claude-in-Chrome MCP tools must be available
- The target app must be accessible in a browser (running locally or deployed)

### Step 1: Set up Chrome
1. Call `tabs_context_mcp` to check existing tabs
2. Create a new dedicated tab: `tabs_create_mcp`
3. Store this tab ID — use it exclusively for all subsequent browser operations

### Step 2: Navigate and explore
1. Navigate to the target app URL
2. If auth is required, ask the user to log in manually, then resume
3. **If targeting mobile or both:** Resize the browser to mobile viewport (390×844px) using `resize_window` before navigating to the feature. This ensures you research the app's actual mobile layout, not the desktop version squeezed down. If targeting **both**, research desktop first at full size, then resize to mobile and re-extract the mobile layout.
4. Navigate to the specific feature/page to animate
5. Take exploratory screenshots to understand the layout

### Step 3: Extract design language
Use `read_page`, `find`, and `javascript_tool` to extract:

- **Colors**: background, surface, border, accent/primary, text, text-dim
  ```js
  // Example: extract computed styles
  const body = getComputedStyle(document.body);
  JSON.stringify({
    bg: body.backgroundColor,
    color: body.color,
    fontFamily: body.fontFamily
  })
  ```
- **Fonts**: heading font-family, body font-family, font weights used
- **Spacing**: padding, margins, border-radius values
- **Component styles**: buttons, cards, badges, avatars, sidebar panels

Record these as CSS custom properties (e.g., `--bg: #0f0f0f`).

### Step 4: Map layout geometry
For the specific feature to animate:

- **Container dimensions**: width, height, position of major containers
- **Element positions**: absolute coordinates of key elements
- **Relationships**: which elements are inside which, z-index stacking
- **Circular elements**: center point and radius
- **Rectangular elements**: origin (top-left), width, height

Use `javascript_tool` to extract exact positions:
```js
const el = document.querySelector('.selector');
const rect = el.getBoundingClientRect();
`${rect.left}, ${rect.top}, ${rect.width}, ${rect.height}`
```

### Step 5: Screenshot key states
Take screenshots of:
- The feature in its default/initial state
- Any intermediate states (hover, loading, processing)
- The final/result state after the feature action completes

These screenshots serve as visual reference for generation.

---

## Phase 2: Interview

Ask the user focused questions to define what to animate. Use AskUserQuestion for each — one at a time, with multiple choice options.

### Question 1: Animation style
```
"What style of animation should this be?"
- Feature Demo (Recommended): Before → Action → After. Shows one feature transformation.
- Carousel: Cycles through multiple app views with cross-fade transitions.
```

### Question 2: Target size
```
"What size should the animation target?"
- Desktop only (960×620px) — for landing pages, marketing, desktop walkthroughs
- Mobile only (360×640px) — for in-app tour tooltips, mobile onboarding
- Both — generates two files from the same brief, one per viewport
```

When **Mobile** or **Both** is selected:
- The mobile variant uses a **360×640px** stage (portrait, matching common phone viewports)
- Simplify the layout: fewer elements (8–12 max), larger relative sizing, no sidebars
- Drop elements that don't translate to small screens (wide toolbars, multi-column layouts)
- Prefer vertical stacking over horizontal layouts
- Increase font sizes relative to stage (minimum 11px body, 14px headings)
- File naming: `<app>-<feature>.html` (desktop), `<app>-<feature>-mobile.html` (mobile)

### Question 3: Feature identification
```
"What specific feature or flow should the animation show?"
- [Options based on research phase findings]
- Other (user describes)
```

### Question 4: The payoff moment
```
"What's the key moment — the visual 'wow' of this animation?"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neonwatty/css-animation-skill](https://github.com/neonwatty/css-animation-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
