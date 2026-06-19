---
trigger: always_on
description: Convert saved HTML snapshots into pixel-perfect Next.js projects with zero external dependencies. Takes browser "Save As" pages with obfuscated CSS/JS and produces a buildable Next.js App Router project — self-hosted fonts, images, icons, and Tailwind config grounded on real extracted CSS values. Use this skill whenever the user wants to convert a saved webpage to Next.js, rebuild a site from snapshots, turn HTML into a React project, recreate a page as a component, or generate a Next.js templat
---


# Snapshot to Next.js

**Convert saved HTML pages into pixel-perfect Next.js projects**

**Mission:** Take saved HTML snapshots — produced by browser "Save As", wget, HTTrack, SingleFile, or any offline capture tool — and convert them into a fully buildable Next.js App Router project with zero third-party dependencies.

The pipeline can also produce intermediate design documentation (Waves 0–2 only) when a full build isn't needed yet.

This is not a summarizer. This is not a screenshot describer. This is a forensic CSS parser that reads every minified rule, decodes every CSS Module class name, resolves every custom property chain, catalogs every `@keyframes` block, and maps every `@media` breakpoint — then builds a production-ready Next.js project from it.

Every design value traces to real CSS/JS from the source snapshot. No approximations. No invented tokens. No "close enough." The downstream builder implements your values **literally** — an unverified guess is worse than an honest gap.

---

## Input Specification

This skill is designed for **saved HTML snapshots** — the kind produced by browser "Save As", wget, HTTrack, SingleFile, or any offline snapshot tool.

### Expected Directory Structure

```
source-pages/
├── {site}-homepage.html              # Full HTML (minified, often 1-2MB)
├── {site}-homepage_files/            # Companion assets folder
│   ├── 06cc9eb5faccd3be.css          # Minified CSS (hashed filenames, 12+ files)
│   ├── 8422-c4149e3ed1566f84.js      # JS bundles (minified)
│   ├── f79251b06e9e...352x352.png    # Images / SVGs / favicons
│   └── Inter-roman.var.woff2         # Font files
├── {site}-pricing.html
├── {site}-pricing_files/
├── {site}-features.html
├── {site}-features_files/
└── ...
```

### Auto-Detection Logic

On first scan of the input directory, detect the input type:

```bash
# Step 1: Find all HTML files
find . -maxdepth 2 -name "*.html" -o -name "*.htm" | sort

# Step 2: Classify each
for f in *.html; do
  base="${f%.html}"
  if [ -d "${base}_files" ]; then
    echo "SNAPSHOT: $f → ${base}_files/ ($(ls ${base}_files/*.css 2>/dev/null | wc -l) CSS files)"
  else
    echo "SINGLEFILE: $f ($(grep -c '<style' "$f" 2>/dev/null) <style> blocks)"
  fi
done
[ -f "package.json" ] && echo "SOURCE REPO: package.json found"
```

### Auto-Detection Table

| Signal | Input Type | Behavior |
|--------|-----------|----------|
| `.html` files + `_files/` directories | Saved snapshot | **Primary mode** — grep minified CSS, decode CSS Module names, extract from companion folders |
| `.html` with `<style>` blocks only, no `_files/` | SingleFile export | **Inline CSS mode** — extract from `<style>` tags within the HTML |
| `package.json` + `src/` directory | Source repository | **Fallback mode** — read component files directly, parse `tailwind.config.*`, follow imports |

### Key Characteristics of Saved Snapshots

- CSS is **MINIFIED** — single-line blobs, 100–400KB each, 12+ files per page
- Filenames are **hashed** (`06cc9eb5faccd3be.css`) — no semantic meaning in filenames
- Class names follow the **CSS Module pattern**: `ComponentName_propertyName__hashCode`
- **CSS custom properties** (`--color-*`, `--font-*`, `--ease-*`) form the design system backbone
- HTML preserves **semantic tags**: `<header>`, `<main>`, `<section>`, `<footer>`, `<nav>`
- Inline `style=""` attributes frequently reference CSS variables via `var(--token-name)`
- Each HTML file represents **one page type** (homepage, pricing, features, about, etc.)
- **Same CSS file** often appears in multiple `_files/` folders — deduplication is mandatory
- JS bundles contain **behavior logic**: scroll animations, intersection observers, dynamic class toggling

---

## Output Specification

The pipeline produces TWO output trees. Extraction-only mode (Waves 0–2) produces the first. Full reconstruction (Waves 0–4) produces both.

### Tree 1: Design Documentation (`.design-soul/`)

```
.design-soul/                                    ← Intermediate extraction docs
├── wave0/                                       ← Per-page deep exploration
│   ├── homepage/
│   │   ├── exploration.md                       # Section inventory + CSS token map + behavior map
│   │   ├── deobfuscated.css                     # Full CSS with semantic class names
│   │   ├── behavior-spec.md                     # JS behaviors as declarative specs
│   │   ├── assets/                              # Downloaded fonts, images, icons, videos
│   │   │   ├── fonts/
│   │   │   ├── images/
│   │   │   └── icons/
│   │   └── done.signal                          # Empty file = this agent completed
│   ├── pricing/
│   │   ├── exploration.md
│   │   ├── deobfuscated.css
│   │   ├── behavior-spec.md
│   │   ├── assets/
│   │   └── done.signal
│   └── {additional-pages}/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [o9-9/skill-snapshot-to-nextjs](https://github.com/o9-9/skill-snapshot-to-nextjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
