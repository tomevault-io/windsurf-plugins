---
trigger: always_on
description: - Never commit directly to `main`.
---

# SOBS Copilot Instructions

## Branch & PR Workflow (Mandatory)

- Never commit directly to `main`.
- Never push directly to `main`.
- Every change must follow: Issue -> new branch -> pull request -> review -> merge.
- Branch naming convention: `issue-<number>-<short-description>` (for example: `issue-321-fix-mcp-key-contrast`).
- PRs must reference the issue number and include a short validation summary.
- If work is started on `main` by mistake, stop and move changes to a new branch before continuing.

These rules apply to humans and automated agents.

## Stack
Flask/Jinja2, Bootstrap 5, chDB. Templates extend `base.html` via `{% block styles %}`, `{% block content %}`, `{% block scripts %}`.

---

## Design Constants

Use these exact values throughout — do not hardcode in individual templates:
- **Mobile breakpoint token:** `{{ mobile_breakpoint_max }}` (currently resolves to `575.98px`, set in `go/cmd/sobs/render.go`)
- **Mobile breakpoint CSS usage:** `@media (max-width: {{ mobile_breakpoint_max }})`
- **Mobile breakpoint JS usage:** `window.matchMedia('(max-width: {{ mobile_breakpoint_max }})')`
- **Test viewports:** 375px (small mobile), 480px (mobile), 575px (trigger point), 992px (tablet), 1440px (desktop)
- **Table border color:** `var(--bs-border-color)` (Bootstrap CSS variable)
- **Secondary text color:** `var(--bs-secondary-color)` (Bootstrap CSS variable)
- **Icon margin utility:** `me-1` (margin-right, Bootstrap) — override with `margin-right: 0 !important` at mobile

---

## Responsive UI Rules (apply to every template change)

### CSS Block Inheritance (MANDATORY)
**Every page template must have this structure — non-negotiable:**
```jinja2
{% block styles %}{{ super() }}<style>
  /* Page-specific CSS here */
</style>{% endblock %}
```
- If you forget `{{ super() }}`, shared stylesheet rules won't apply.
- `base.html` has `{% block styles %}{% endblock %}` before `</head>` — do not remove it.

### Tables
- **Every data table** must support mobile card mode at `≤{{ mobile_breakpoint_max }}`.
- Server-rendered tables: Add a shared CSS class (e.g., `tags-mobile-card-table`) and apply the standard mobile-card pattern:
  ```css
  @media (max-width: {{ mobile_breakpoint_max }}) {
    .tags-mobile-card-table thead { display: none; }
    .tags-mobile-card-table,
    .tags-mobile-card-table tbody,
    .tags-mobile-card-table tr,
    .tags-mobile-card-table td { display: block; width: 100%; }
    .tags-mobile-card-table tr { border: 1px solid var(--bs-border-color); border-radius: 0.5rem; margin-bottom: 0.75rem; padding: 0.5rem 0.625rem; }
    .tags-mobile-card-table td { border: 0; padding: 0.2rem 0; }
    .tags-mobile-card-table td::before { content: attr(data-label); display: block; font-size: 0.72rem; color: var(--bs-secondary-color); text-transform: uppercase; letter-spacing: 0.02em; margin-bottom: 0.1rem; }
  }
  ```
- **Every `<td>` must carry a `data-label="Column Name"` attribute** — this is how the label appears on mobile above the cell value.
- **JS-rendered tables** must also include the mobile-card class on the table and `data-label` attributes on each generated `<td>`. Use string concatenation or template literals to ensure these are present.

### Action Buttons (header/panel buttons)
- **Every action button** (Add, Delete, Edit, etc.) must wrap its label text in `<span class="PAGE-btn-label">Label</span>` and include a `title="Label"` attribute for accessibility.
- Hide labels at mobile (`≤{{ mobile_breakpoint_max }}`), showing only the icon:
  ```css
  @media (max-width: {{ mobile_breakpoint_max }}) {
    .PAGE-btn-label { display: none; }
    .btn:has(> .PAGE-btn-label) i { margin-right: 0 !important; }
  }
  ```
- **JS code that resets button `innerHTML`** (e.g., after async operations) must also use the `<span class="PAGE-btn-label">` wrapper.
- Example button HTML:
  ```html
  <button class="btn btn-sm btn-outline-info" title="Add Tag">
    <i class="bi bi-plus-circle me-1"></i><span class="PAGE-btn-label">Add Tag</span>
  </button>
  ```

### Page Headers
- **Every full page header** must use the shared macro in `templates/_page_header_macros.html` (`render_page_header`) unless there is a documented one-off exception.
- Preferred structure:
  ```jinja2
  {% from "_page_header_macros.html" import render_page_header %}
  {% set page_actions %}
    <a class="btn btn-sm btn-outline-secondary page-help-btn" title="Help">
      <i class="bi bi-question-circle me-1"></i><span class="PAGE-btn-label">Help</span>
    </a>
  {% endset %}
  {{ render_page_header('Page Title', icon_class='bi bi-ICON', icon_text_class='text-COLOR', actions_html=page_actions) }}
  ```
- **Mobile layout (`≤{{ mobile_breakpoint_max }}`)**:
  - Row 1: page icon + title on the left, action icon buttons on the right.
  - Row 2: meta line only (counts, chips, badges, refresh controls).
  - Optional subtext renders below the meta line.
- **Larger layout**:
  - Row 1: page icon + title, then meta line, then action buttons.
  - Row 2: subtext only when the page has explanatory copy.
- **Help button rule**: Help belongs at the far right of the action cluster, always uses the same bordered style (`page-help-btn` / outline-secondary), and uses the same question-circle icon across pages.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abartrim/sobs](https://github.com/abartrim/sobs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
