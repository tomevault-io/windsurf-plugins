---
trigger: always_on
description: **Fully expanded specification for code-generation models — Feb 2026, rev 3.5**
---

### **The Definitive Guide to Authoring a RizzyUI Component (Version 3.5 - Documentation Standards)**

**Fully expanded specification for code-generation models — Feb 2026, rev 3.5**

---

---

## Up-Front Non-Negotiables (READ FIRST)

1. **RizzyUI is SSR (server-side rendered) ONLY.**

   * **Do not** write components that require Blazor interactivity.
   * **Do not** introduce Blazor event-based behaviors (`@onclick`, `@onchange`, `@onsubmit`, `@bind`, `EventCallback`, interactive input components that require a Blazor circuit, etc.).
   * All client-side interactivity must be implemented via **Alpine.js** and/or **HTMX** patterns already used in RizzyUI (see `docs/agents/alpine.md`).

2. **AGENTS ONLY — Node install rule**

   * Run `npm install` in **any directory that contains a `packages.json`**.
     *(If the repository uses `package.json` in practice, treat it as the same intent; do not skip installs.)*

3. **Documentation is Mandatory**

   * **Every** new component must have a corresponding documentation page in `src/RizzyUI.Docs/Components/Pages/Components/`.
   * **Every** modified component must have its documentation page updated to reflect API, parameter, or behavior changes.
   * **Every** new component must be added to the navigation menu in `src/RizzyUI.Docs/Components/Layout/ComponentList.razor`.

4. **SSR interaction event policy (mandatory)**

   * For browser-observable interactions in SSR components, prefer Alpine `$dispatch(...)` or `CustomEvent` helpers that emit `rz:` namespaced events.
   * Do **not** use `EventCallback` for browser-only post-render interaction flows.
   * Event payloads must use serializable primitives and stable identifiers.
   * Never emit full `TItem` instances or server object graphs in browser event payloads.
   * For table-like stateful primitives, emit granular events and a table-level aggregate state event when useful (for example `rz:table:on-state-change`).

5. **Accessibility contract for root-level interactive components (mandatory)**

   * Must treat accessibility behavior as a public API contract for every root-level interactive component.
   * During accessibility hardening or refactors, preserve existing compliant behavior before adding or replacing behavior. Inspect the component's Razor, C#, JavaScript/Alpine, tests, and docs before changing it.
   * Must document keyboard behavior in component documentation, including supported keys and the exact effect of each key.
   * Must define explicit ARIA semantics in SSR-safe HTML, including required roles, states, properties, and ID-based relationships (`aria-controls`, `aria-labelledby`, `aria-activedescendant`, etc.) when applicable.
   * Must implement predictable focus management for initial focus, roving focus patterns, focus traps (when applicable), and focus restoration after dismiss/close flows.
   * Must document and implement a screen-reader announcement strategy, including when announcements occur, when they are `polite` versus `assertive`, and when announcements should be suppressed to avoid noise.
   * Must include component tests that prove keyboard behavior, ARIA semantics, focus management, and announcement behavior for every interactive component.
   * Must document known accessibility limitations and assistive technology quirks when discovered, including temporary mitigations where available.
   * For APG-derived widgets, must name the adopted WAI-ARIA Authoring Practices Guide (APG) pattern and explain any intentional deviations.
   * Must not implement accessibility logic with Blazor interactive runtime features such as `EventCallback`, `@onclick`, `@onchange`, `@onsubmit`, or `@bind`.
   * Must implement client interactions using SSR-safe HTML plus shared Alpine/JavaScript primitives defined for RizzyUI Phase 1, while preserving CSP-safe constraints.
   * Must always preserve SSR-only and CSP requirements for accessibility behavior and event handling.
   * Must not create replacement components during accessibility hardening unless the prompt explicitly authorizes new component creation. Update the existing component in place, for example `RzNativeSelect`, `RzCombobox`, `RzNavigationMenu`, `RzAlert`, `RzTooltip`, or `RzDataTable`.

---

## Precedence (Mandatory)

- `AGENTS.md` is the canonical entry point.
- Delegated specification files extend this guide.
- If a conflict exists, `AGENTS.md` takes precedence unless explicitly delegated.

---

## How to Use This File (Execution Flow) (Mandatory)

1. Read all **Non-Negotiables**
2. Identify the task type
3. Load the **Minimum Required Read Set**
4. Perform implementation using delegated specifications
5. Apply the appropriate **Validation Checklists**
6. Produce output following the **Output Contract**

---

## Node Manifest Clarification (Mandatory)

The original repository guidance refers to `packages.json`. In practice, agents should also check for `package.json` and treat it as the same intent.

---

## Delegated Specifications (Routing)

The following files contain expanded rules. Each file focuses on a specific domain. Agents must consult these when working in that area.

- `docs/agents/repository-structure.md`  
  → Repository layout, directory responsibilities, and file placement rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JalexSocial/RizzyUI](https://github.com/JalexSocial/RizzyUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
