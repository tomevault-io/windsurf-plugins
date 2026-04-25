---
trigger: always_on
description: > **Persona:** Senior React + Tailwind CSS Designer. Every component is production-grade, accessible, dark-mode-ready, fully dynamic, reusable across projects, and aligned with the Aumovio brand guide.
---

# CLAUDE.md — Aumovio UI Component Design System

> **Persona:** Senior React + Tailwind CSS Designer. Every component is production-grade, accessible, dark-mode-ready, fully dynamic, reusable across projects, and aligned with the Aumovio brand guide.

---

## Table of Contents

1. [Configuration & Setup](#1-configuration--setup)
2. [Design Tokens & Theming](#2-design-tokens--theming)
3. [Dark Mode](#3-dark-mode)
4. [Icons](#4-icons)
5. [Optimization Guidelines](#5-optimization-guidelines)
6. [Component Library](#6-component-library)
   - [Accordion](#accordion)
   - [Alerts](#alerts)
   - [Avatar](#avatar)
   - [Badge](#badge)
   - [Banner](#banner)
   - [Bottom Navigation](#bottom-navigation)
   - [Breadcrumb](#breadcrumb)
   - [Buttons](#buttons)
   - [Button Group](#button-group)
   - [Card](#card)
   - [Carousel](#carousel)
   - [Chat Bubble](#chat-bubble)
   - [Clipboard](#clipboard)
   - [Datepicker](#datepicker)
   - [Device Mockups](#device-mockups)
   - [Drawer](#drawer)
   - [Dropdowns](#dropdowns)
   - [Footer](#footer)
   - [Gallery](#gallery)
   - [Indicators](#indicators)
   - [Jumbotron](#jumbotron)
   - [KBD](#kbd)
   - [List Group](#list-group)
   - [Mega Menu](#mega-menu)
   - [Modal](#modal)
   - [Navbar](#navbar)
   - [Pagination](#pagination)
   - [Popover](#popover)
   - [Progress](#progress)
   - [Rating](#rating)
   - [Sidebar](#sidebar)
   - [Skeleton](#skeleton)
   - [Speed Dial](#speed-dial)
   - [Spinner](#spinner)
   - [Stepper](#stepper)
   - [Tables](#tables)
   - [Tabs](#tabs)
   - [Timeline](#timeline)
   - [Toast](#toast)
   - [Tooltips](#tooltips)
   - [QR Code](#qr-code)
7. [Forms](#7-forms)
   - [Input Field](#input-field)
   - [File Input](#file-input)
   - [Search Input](#search-input)
   - [Number Input](#number-input)
   - [Phone Input](#phone-input)
   - [Select](#select)
   - [Textarea](#textarea)
   - [Timepicker](#timepicker)
   - [Checkbox](#checkbox)
   - [Radio](#radio)
   - [Toggle](#toggle)
   - [Range](#range)
   - [Floating Label](#floating-label)
8. [Typography](#8-typography)
   - [Headings](#headings)
   - [Paragraphs](#paragraphs)
   - [Blockquote](#blockquote)
   - [Images](#images)
   - [Lists](#lists)
   - [Links](#links)
   - [Text Utilities](#text-utilities)
   - [HR](#hr)
9. [Charts (ApexCharts)](#9-charts-apexcharts)

---

## 1. Configuration & Setup

### Vite + React + Tailwind v4

```js
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [react(), tailwindcss()],
})
```

```css
/* src/assets/styles/index.css */
@import "tailwindcss";
/* All @theme tokens defined here — see Section 2 */
```

### Required Dependencies

```bash
# Core
npm install react react-dom react-router-dom
npm install @vitejs/plugin-react @tailwindcss/vite tailwindcss

# UI Utilities
npm install @headlessui/react @heroicons/react
npm install @fortawesome/react-fontawesome @fortawesome/free-solid-svg-icons @fortawesome/free-regular-svg-icons @fortawesome/free-brands-svg-icons

# Notifications
npm install react-toastify

# Date
npm install date-fns

# Charts
npm install apexcharts react-apexcharts

# Auth / HTTP
npm install axios js-cookie

# QR
npm install qrcode.react
```

### Environment Variables

```env
# .env
VITE_APP_NAME=YourApp
VITE_API_BASE_URL=http://localhost:3000/api/v1/
VITE_LAYOUT_MODE=top        # "top" | "sidebar"
VITE_THEME=light            # "light" | "dark" | "system"
```

---

## 2. Design Tokens & Theming

All tokens live inside `@theme {}` in `src/assets/styles/index.css`.

### Color Hierarchy

| Role      | Token Family       | Hex Anchor | Usage |
|-----------|--------------------|------------|-------|
| Primary   | `orange-*`/`primary-*` | `#FF4208` | 60% — CTAs, active states |
| Secondary | `purple-*`/`secondary-*` | `#4827AF` | 30% — accents, gradients |
| Blue      | `blue-*`           | `#18A9E7`  | Info, links |
| Turquoise | `turquoise-*`      | `#12CAAE`  | Success alt, tags |
| Yellow    | `yellow-*`         | `#CEC43A`  | Highlights, amber |
| Grey      | `grey-*`           | —          | Neutral surfaces |
| Danger    | `danger-*`         | `#D82822`  | Errors, destructive |
| Warn      | `warn-*`           | `#FFD600`  | Warnings |
| Success   | `success-*`        | `#32CB70`  | Confirmations |

### Spacing Scale (Tailwind defaults, referenced as-is)

Use Tailwind's built-in spacing: `p-1`=4px, `p-2`=8px, `p-4`=16px, `p-6`=24px, `p-8`=32px, `p-10`=40px, `p-12`=48px.

### Border Radius Scale

```
rounded-sm   → 2px
rounded      → 4px
rounded-md   → 6px
rounded-lg   → 8px
rounded-xl   → 12px
rounded-2xl  → 16px
rounded-full → 9999px
```

### Shadow Scale

```
shadow-sm   → subtle card lift
shadow      → default cards
shadow-md   → dropdowns, popovers
shadow-lg   → modals, overlays
shadow-xl   → hero elements
shadow-2xl  → focus accent
```

### CSS Custom Properties (for JS access)

```js
// src/utils/tokens.js
export const TOKENS = {
  primary:   '#FF4208',
  secondary: '#4827AF',
  blue:      '#18A9E7',
  turquoise: '#12CAAE',
  yellow:    '#CEC43A',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jm-Paunlagui) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
