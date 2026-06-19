---
trigger: always_on
description: Generate complete, interactive web pages using LLasM (LLM Assembly Language). A framework optimized for LLM code generation, not human developers. Use when the user wants to build a website, web page, web app, landing page, form, dashboard, or any browser-based UI.
---


# LLasM Page Generator

**A framework optimized for LLM code generation, not human developers.**

Generate complete, production-ready web pages with zero build tooling.

## Design Principles (Priority Order)

### Tier 1: SECURITY
- **Safe Binding** - No innerHTML. Sanitize all dynamic content.
- **No UI Cookies** - Auth is server-side only.
- **Zero Trust Input** - Validate all user input and URL params.

### Tier 2: ACCESSIBILITY
- **WCAG Compliant** - WCAG 2.1 AA minimum.
- **Semantic Elements** - Use native HTML5 (nav, main, article, section).
- **I18n Ready** - Locale keys with RTL support.

### Tier 3: QUALITY
- **Lighthouse 90+** - All four categories.
- **SEO Ready** - Meta tags, Open Graph, semantic headings.
- **Self-Booting Pages** - Every page hydrates independently.

### Tier 4: PERFORMANCE
- **LLM-First** - Code for LLMs by LLMs. Human readability is a non-goal.
- **One Way** - Single canonical approach. No alternatives.
- **Terse by Default** - 1-3 char identifiers.
- **Zero Build** - HTML + ES + CSS only.
- **CSS Before JS** - If CSS can do it, don't use JS.
- **Browser-Native** - Only browser APIs. No external libraries.

## REQUIRED CHECKLIST

Every LLasM page MUST include ALL of these. Do not skip any.

- [ ] **Folder per app** - `docs/examples/{app-name}/` with `index.html` as entry
- [ ] **Separate files** - Each view is a separate HTML file (NO hash routing)
- [ ] **Cache-bust import** - `import{l}from"../../llasm.js?v=x9k2m4p7";`
- [ ] **Build date** - `<p class="t1 o5">Built YYYY-MM-DD</p>` in footer
- [ ] **theme-color meta** - `<meta name="theme-color" content="#0066ff">`
- [ ] **Critical CSS** - Inline `<style>` in head with above-fold classes
- [ ] **Dark mode script** - Blocking script in head before body
- [ ] **Semantic footer** - `<footer>` with copyright and build date
- [ ] **No inline styles** - Use utility classes, not `style="..."`

### Dark Mode Script (Required in Head)

```html
<script>try{var d=localStorage.getItem('llasm-dark');if(d==='true'||(d===null&&matchMedia('(prefers-color-scheme:dark)').matches))document.documentElement.classList.add('dark');}catch(e){}</script>
```

### Critical CSS (Required in Head)

```html
<style>
*,*::before,*::after{box-sizing:border-box}
:root{--m-p:#0066ff;--m-s:#6c757d;--m-ok:#28a745;--m-err:#dc3545}
body{margin:0;font-family:system-ui,sans-serif;background:var(--m-bg,#fff);color:var(--m-fg,#212529)}
.f{display:flex}.fc{flex-direction:column}.fi{align-items:center}.fj{justify-content:center}.fb{justify-content:space-between}.fg{flex-grow:1}
.g{display:grid}.gc3{grid-template-columns:repeat(3,1fr)}
.g1{gap:.25rem}.g2{gap:.5rem}.g3{gap:1rem}.g4{gap:1.5rem}
.p2{padding:.5rem}.p3{padding:1rem}.p4{padding:1.5rem}.px3{padding-inline:1rem}.py2{padding-block:.5rem}.py4{padding-block:1.5rem}
.mxa{margin-inline:auto}.xw3{max-width:900px}
.t1{font-size:.75rem}.t2{font-size:.875rem}.t3{font-size:1rem}.t4{font-size:1.25rem}.t5{font-size:1.5rem}.tb{font-weight:700}.tc{text-align:center}
.c1{color:var(--m-p)}.cg{color:#6c757d}.cw{color:#fff}.cb{color:#000}
.b1{background:var(--m-p)}.b2{background:var(--m-s)}.bg{background:#f5f5f5}.bw{background:#fff}
.r{border-radius:4px}.r2{border-radius:8px}.rf{border-radius:9999px}.sh{box-shadow:0 2px 8px rgba(0,0,0,.1)}.bd{border:1px solid #ddd}
.tdn{text-decoration:none}.cp{cursor:pointer}.o5{opacity:.5}
.dn{display:none}.rel{position:relative}.abs{position:absolute}
html.dark{--m-bg:#1a1a1a;--m-fg:#f5f5f5;--m-p:#5c9eff}
html.dark body{background:#1a1a1a;color:#f5f5f5}
html.dark .bg{background:#2a2a2a}
@media(max-width:768px){.sm\:fc{flex-direction:column}.sm\:gc1{grid-template-columns:1fr}.sm\:dn{display:none}}
</style>
```

## Output Structure

Every LLasM page has three parts:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <meta name="description" content="Page description for SEO">
  <title>Page Title</title>
</head>
<body class="p3">
  <!-- 1. SEMANTIC HTML with utility classes -->
  <main class="xw3 mxa f fc g3">
    <h1 class="t6 c1 tb">Welcome</h1>
    <p class="cg" data-m-bind="message"></p>
    <button data-m-on="click:save" class="b1 cw p2 px3 r">Save</button>
  </main>

  <!-- 2. MANIFEST: State, i18n, theme, persistence -->
  <script type="application/llasm+json" id="manifest">
  {"v":1,"r":{"s":{"message":"Hello World"}},"l":{"en":{}}}
  </script>

  <!-- 3. HANDLERS: Event handlers -->
  <script type="module">
  import{l}from"./llasm.js";
  l.h({save:(e,s,L)=>L.t('Saved!','ok')});
  </script>
</body>
</html>
```

## Valid Attributes

Only these `data-m-*` attributes exist. **Do NOT invent new ones.**

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `data-m-bind` | State binding | `data-m-bind="user.name"` |
| `data-m-on` | Events | `data-m-on="click:save"` |
| `data-m-if` | Conditional | `data-m-if="!loading"` |
| `data-m-class` | Conditional class | `data-m-class="active:isActive"` |
| `data-m-enhance` | Enhancements | `data-m-enhance="primary ripple"` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [walkingriver/llasm](https://github.com/walkingriver/llasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
