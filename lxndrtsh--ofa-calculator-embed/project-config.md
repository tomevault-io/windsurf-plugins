---
trigger: always_on
description: Hey future-me / Cursor agent 👋
---

# Cursor Project Context — OFA Calculator

Hey future-me / Cursor agent 👋  
This repo is for a **client-facing, embeddable, two-form lead calculator** for **High Performance Providers** (HPP). It's intended to be dropped into sites like **Go High Level** / **HubSpot** / **WordPress** using a tiny `<script>` tag. The embed should **not** expose private API keys and should be configurable from the server.

This file is here so the agent has the full story and doesn’t “optimize” away decisions we already made.

---

## 1. What we’re building

We are building **two separate widgets** (two different embeds):

1. **Impact Analysis (Widget A)**  
   - Input: `# of employees`, optional location, CRM fields  
   - Math is config-driven  
   - Output is a set of opioid/dependence risk indicators for an employer group  
   - On submit → **Go High Level** (GHL) **and** our **Referral Tool** (with referral cookie)

2. **Return-on-Community (Widget B)**  
   - Input: `state`, `county` → try API lookup for population → fallback to manual population  
   - Uses the same CRM fields as Impact  
   - Output is counts/indicators, **not** dollar savings  
   - On submit → **Go High Level** and **Referral Tool**

**Client requirement:** HPP’s marketing team must be able to embed **either one** independently on different pages. That’s why we have **two loader scripts**.

---

## 2. High-level architecture

**Pattern:** host-page script → iframe → React app → server routes

- **Loader scripts (UMD):**
  - `/cdn/leadcalc-impact.min.js`
  - `/cdn/leadcalc-community.min.js`
  - These are built with **Rollup** and live in `public/cdn/` so Next.js can serve them.
  - They expose **one global**: `window.OFACalculator.init(...)`

- **Iframe apps (React, Next.js App Router):**
  - `/embed/impact`
  - `/embed/community`
  - These pages run in an iframe, render the forms, and use `postMessage` back to the parent for:
    - bootstrapping (API base, theme, config version, referral token)
    - dynamic height (no scrollbars on host page)

- **API routes (Next.js):**
  - `GET /api/config?version=...&form=impact|community` → returns math constants
  - `GET /api/lookup/population?state=..&county=..` → TODO: call real Census; right now returns `population: null`
  - `POST /api/submit/impact`
  - `POST /api/submit/community`
  - These 2 POST endpoints are where we will integrate **Go High Level** and the **Referral Tool** securely (server-side secrets).

- **Hosting target:** **Vercel**  
  Next.js → Vercel; static loader bundles live in `public/cdn/*` so they’re served from the same domain.

---

## 3. Local dev setup (current)

Run:

```bash
npm install
npm run dev
# http://localhost:3000
# http://localhost:3000/demo
```

- `npm run dev` runs **Next.js** **and** two **Rollup watchers** (for the two loaders).
- The demo page (`/demo`) simulates a customer website. It loads:

```html
<script src="/cdn/leadcalc-impact.min.js"></script>
<div id="impact-mount"></div>
<script>
  OFACalculator.init('impact-mount', {
    apiBase: window.location.origin,
    iframeBase: window.location.origin,
    configVersion: 'dev',
    theme: 'light',
    referralCookie: 'referral'
  });
</script>
```

If `/cdn/leadcalc-impact.min.js` 404s → Rollup didn’t build. See section 5.

---

## 4. Important files / dirs

- **app/embed/impact/page.tsx**  
  React UI for the employer calculator.  
  - Sends `OFA_CALCULATOR_READY`  
  - Waits for `OFA_CALCULATOR_BOOT`  
  - Renders the form, runs the math, then posts to `/api/submit/impact`

- **app/embed/community/page.tsx**  
  React UI for the county/community calculator.  
  - Same postMessage pattern  
  - Calls `/api/lookup/population` (currently stubbed)

- **app/api/config/route.ts**  
  Returns the canonical math config (see §6). This is because the client said: **all math constants must be configurable via API**.

- **app/api/submit/impact/route.ts** and **app/api/submit/community/route.ts**  
  Stubs right now. This is where we will add the real **GHL** and **Referral Tool** calls.

- **packages/loader-impact/src/index.ts** and **packages/loader-community/src/index.ts**  
  The UMD loaders. These:
  - read the **referral cookie** on the host page,
  - create the **iframe** pointing to `/embed/impact` or `/embed/community`,
  - `postMessage` a boot payload into the iframe,
  - listen for resize events from the iframe.

- **public/cdn/**  
  Where Rollup writes built loader bundles. Next.js serves this statically → becomes `/cdn/...`.

---

## 5. Rollup build notes (we hit this already)

We saw:

> RollupError: Could not resolve entry module "src/index.ts".

This happened because the Rollup config was using a **relative** input and we were running Rollup from the repo root.

**Fix we applied:**

1. Installed the TS plugin:

```bash
npm i -D @rollup/plugin-typescript
```

2. Updated **both** Rollup configs to use **absolute paths** and **TS plugin**:

```js
import path from 'node:path';
import { fileURLToPath } from 'node:url';
import terser from '@rollup/plugin-terser';
import typescript from '@rollup/plugin-typescript';

const __dirname = path.dirname(fileURLToPath(import.meta.url));

export default {
  input: path.join(__dirname, 'src/index.ts'),
  output: {
    file: path.join(__dirname, '../../public/cdn/leadcalc-impact.min.js'),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lxndrtsh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
