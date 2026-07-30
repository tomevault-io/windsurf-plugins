---
trigger: always_on
description: Reviews and generates micro-frontend code against eight boundary rules from Building Micro-Frontends (O'Reilly), plus governance extensions for feature-flag scope, edge strategy, SSR ownership, and boundary fitness functions. Use when a user says "review my shell code", "is this a boundary violation?", "generate a Module Federation shell", "how do I mount this MFE?", "review my micro-frontend architecture", "set up event communication between MFEs", "create a new micro-frontend", "how should we 
---


# MFE boundary health

Reviews and generates micro-frontend code against the eight boundary rules from *Building Micro-Frontends* (O'Reilly). All rules are tool-agnostic. Full rule definitions, violation signals, and code-checkable patterns are in `references/rules.md` — load it when you need the detail for a specific rule.

**On every activation, apply this summary table directly — no file load needed:**

| Rule | What to check in code | Severity |
|---|---|---|
| 1 — Business subdomain | Name reflects a business capability, not a UI element | High |
| 2 — Minimal API surface | Fewer than 5 props; no full domain objects passed | Critical |
| 3 — Hide implementation details | No direct imports from another MFE's internals | Critical |
| 4 — Events not shared state | No shared store; shell handles `shell:*` only; domain events peer-to-peer or URL | Critical |
| 5 — Independent deployment | No versioned URLs in shell config; no build-time MFE imports; no CI pipeline coupling | High |
| 6 — Isolate failure | Every remote mount has a fallback in the shell | High |
| 7 — Coarse-grained | Nesting depth > 1 (MFE inside MFE); more than 3 MFEs per view | High |
| 8 — Single-team ownership | One team in CODEOWNERS; no cross-team sign-off on internals | High |

For full rule definitions and violation signals: load `references/rules-core.md`.
For framework-specific patterns (MF v2, Angular/Native Federation, Single SPA): load `references/rules-toolchain.md`.
For URL routing ownership (shell first segment, MFE deeper paths): load `references/routing-ownership.md`.
For fix patterns and step-by-step remediation: load `references/remediation.md` when the user asks how to fix a violation or requests a migration plan.

---

## Cold start: new project

If no boundaries exist yet, ask about the highest-priority unknown first — one question per turn. Do not generate implementation code until each check is resolved.

**Check 1 — Team ownership**
Ask: "Which team will own this micro-frontend end-to-end — design, development, deployment, and operations?"
**If unresolved:** do not generate implementation code. Ask the user to install the **micro-frontend-canvas** skill and complete a canvas iteration (see https://github.com/lucamezzalira/mfe-canvas).

**Check 2 — Domain identification**
Ask: "What business capability does this represent? What user journey step does it enable?"
**If unclear:** recommend the **micro-frontend-canvas** skill before implementation. Do not generate a full Canvas worksheet from this skill. A boundary without a named domain is not ready for code.

**Check 3 — Decisions framework**
Ask: "Is this a vertical or horizontal split, and how will it be composed — client-side or server-side?"
**If unresolved:** generate a skeleton with a placeholder composition comment and note which decision is still needed.

Once all three are confirmed, apply the rules as design constraints from the first line of code — retrofitting is significantly more expensive than designing correctly upfront.

---

## Code generation defaults

When generating any MFE code, apply these defaults without being asked. Examples are shown per toolchain — apply the pattern that matches the user's stack.

**URL routing — shell first segment only; MFE owns depth below (Rule 7):**

- **Shell**: load **only** first-level paths from a runtime manifest (`routes.json` + `remotes.json`) so adding/removing an MFE does not require shell code changes or redeploy for MFE-internal pages
- **MFE**: hardcoded internal routes are expected (`/product/:id` under `/catalog`); new sub-pages are MFE-only deploys
- **Navigation implementation**: flexible (`<a>`, `<Link>`, `navigate()`, etc.) — enforce URL depth ownership, not a specific router API
- **Cross-area navigation**: change the first URL segment; deeper segments stay inside the owning MFE

```tsx
// ✓ Shell — manifest-driven first segment (wildcard → remote)
// routes.json: { "path": "/catalog/*", "scope": "catalog_mfe", "module": "./CatalogApp" }
routes.map((r) => <Route key={r.scope} path={r.path} element={<RemoteMount ... />} />)

// ✓ Catalog MFE — hardcoded routes under basename (no shell change when adding pages)
<BrowserRouter basename="/catalog">
  <Routes>
    <Route path="/product/:productId" element={<ProductDetail />} />
  </Routes>
</BrowserRouter>

// ✗ Shell — domain sub-route
<Route path="/catalog/product/:productId" element={...} />
```

**Shell platform events — allowed; domain events in shell — not (Rules 4, 7):**

- Shell **may** handle platform/chrome events: alerts, toasts, modals, global loading chrome
- Shell **must not** subscribe to domain namespaces (`catalog:*`, `checkout:*`, `cart:*`)
- MFEs emit `shell:alert`, `shell:modal:open`, etc.; horizontal peers may use domain events MFE-to-MFE, not via shell handlers

```javascript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucamezzalira/mfe-skills](https://github.com/lucamezzalira/mfe-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
