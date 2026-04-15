---
trigger: always_on
description: Before modifying or creating any module, service, or feature, read the relevant documentation under:
---

# FTG Racing Manager — Claude Development Guide

## 1. Mandatory Pre-Development Review

Before modifying or creating any module, service, or feature, read the relevant documentation under:

```
frontend/src/routes/admin/docs/ai/
```

| Document | When to read |
|---|---|
| `standards.md` | Always — governs code patterns |
| `architecture.md` | Always — state machines, critical fallbacks |
| `services.md` | When touching services, stores, or Cloud Functions |
| `database_schema.md` | When touching Firestore reads/writes |
| `weekend_pipeline.md` | When touching race simulation or scheduled functions |
| `human/postmortem_r2.md` / `human/postmortem_r3.md` | When modifying `functions/index.js` |
| `human/postmortem_admin_qualy_wipe.md` | When creating or modifying any admin tool |

---

## 2. Technology Stack

This is a **Svelte 5 + Firebase** web application. The Flutter codebase (`lib/`) is **deprecated and must not be modified**. All new development happens in `frontend/`.

```
frontend/        ← SvelteKit 5 (active — all development here)
functions/       ← Firebase Cloud Functions v2 (Node.js 20, strict mode)
lib/             ← DEPRECATED Flutter code. Do not touch.
```

**Frontend stack:** SvelteKit · Svelte 5 Runes · TypeScript · Tailwind CSS · Firebase v10
**Backend:** Firestore · Cloud Functions v2 · Firebase Auth · Firebase Hosting

---

## 3. Core Development Rules

### 3.1 Reactivity (Svelte 5 Runes — Non-negotiable)

- **Use `$state()`** for reactive data from Firestore.
- **Use `$derived()`** for all computed values — formatting, filtering, transformations.
- **Use `$effect()`** sparingly — only for DOM integrations or Firebase SDK synchronization.
- **Never use** Svelte 4 `writable` / `readable` stores for local state.
- **Expose reactive state** via class getters, not raw store exports.

### 3.2 Architecture: Service / Store Separation

```
src/lib/services/*.svelte.ts  →  Stateless logic, pure Firebase API calls
src/lib/stores/*.svelte.ts    →  UI reactivity, Firestore onSnapshot listeners
src/routes/**/*.svelte        →  UI only — NO direct Firestore calls
```

**Hard rule:** Never call `doc()`, `getDoc()`, `setDoc()`, or `collection()` directly from a `.svelte` component file.

### 3.3 Transactional Safety

Every mutation that affects **budget**, driver/team **ownership**, or **economic fields** (`budget`, `value`, `salary`) **must** use `runTransaction` or a server-side Cloud Function.

Silent writes outside transactions are forbidden for these fields.

### 3.4 Error Handling

- Empty `catch` blocks are **forbidden**.
- All errors must be logged with namespace context:
  ```typescript
  console.error('[SponsorService:negotiate] Failed to write transaction:', e);
  ```
- Use `try/catch` at the boundary where recovery or user feedback is possible, not at every line.

### 3.5 UI/UX Standards

- **No browser dialogs.** `alert()`, `confirm()`, and `prompt()` are prohibited. Use `uiStore` and custom modal components.
- **Tailwind CSS only.** No inline styles, no raw hex codes. Custom colors reference variables defined in `app.css`.
- **Design tokens** — All surfaces and colors use CSS variables (`--app-primary`, `--app-surface`, `--app-bg`, etc.). Never reference design values by literal color.
- **Mobile First.** Design for mobile, then scale up to desktop dashboards.
- **Design aesthetic:** Premium Dark Gold — high contrast, subtle gradients, `backdrop-blur` for overlays.

### 3.6 Anti-Hardcoding

Every business value must be centralized:

- **Text:** Use the i18n utility (`src/lib/utils/i18n.ts`) for all user-facing strings.
- **Business constants:** Reference `reglas_negocio.md` and centralize in `src/lib/constants/`. Never embed fees, thresholds, or multipliers directly in logic.
  - Examples: Team rename fee ($500k), HQ maintenance (Level × $15k), XP threshold (500 XP), Fibonacci upgrade multipliers.
- **Firestore config:** Prefer Firestore-stored config for values that change over time.

---

## 4. Cloud Functions — Critical Safety Rules

> **Context:** Functions run in Node.js strict mode. Violations are silent — they fail in Firebase logs only, not in local console.

### 4.1 Variable Declaration (Postmortem R2/R3 — Critical)

**Always declare variables with `let` before any conditional assignment.**

```js
// WRONG — throws ReferenceError in strict mode
if (teamRole === "ex_driver") { extraCrash = 0.001; }
const crashed = Math.random() < (accProb + extraCrash);

// CORRECT
let extraCrash = 0;
if (teamRole === "ex_driver") { extraCrash = 0.001; }
const crashed = Math.random() < (accProb + extraCrash);
```

### 4.2 Array Guard Pattern (Postmortem R2/R3 — Critical)

**Always check `array.length > 0`, not just array existence. Empty arrays are truthy.**

```js
// WRONG — [] is truthy, causes false skip
if (rSnap.data().qualyGrid) { continue; }

// CORRECT
if (rSnap.data().qualyGrid?.length > 0) { continue; }
```

### 4.3 Deploy Verification Rule

After ANY fix to `functions/index.js`, remind the user to:
1. Run `firebase deploy --only functions`
2. Verify in Firebase Console → Functions that the deployment timestamp is current


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FGT-felipe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
