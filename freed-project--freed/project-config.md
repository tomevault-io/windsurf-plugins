---
trigger: always_on
description: - **After implementing ANY new features:** Update `docs/PHASE-*.md` immediately — do not wait to be asked. Check every phase whose success criteria or task table is affected and update checkboxes + status lines in the same commit as the feature work.
---

# Agent Instructions

## Rules

- **After implementing ANY new features:** Update `docs/PHASE-*.md` immediately — do not wait to be asked. Check every phase whose success criteria or task table is affected and update checkboxes + status lines in the same commit as the feature work.
- **Roadmap sync:** When `docs/PHASE-*.md` changes, update the roadmap data in `website/src/app/roadmap/RoadmapContent.tsx` to match (`✓ Complete` → `"complete"`, `🚧 In Progress` → `"current"`, else `"upcoming"`). The roadmap is public marketing surface: route the website edit through the `www` lane (`freed-build-www`), not through `dev`.
- **Time estimates:** Machine time only ("one conversation", "~10 min"). Never quote human hours/days.
- **IDs:** Display tail — `...${id.slice(-8)}`.
- **Number formatting:** All user-facing numbers must use `Number.toLocaleString()` (or `Intl.NumberFormat`) — never raw `.toString()` or string interpolation. This ensures locale-appropriate grouping separators (e.g. commas in `en-US`) for counts, totals, and stats.
- **Node toolchain:** Use the repo-pinned Node toolchain from `.nvmrc`. Shell scripts must resolve `node`, `npm`, and `npx` from the same install, never by mixing a global `npm`/`npx` with a different `node` on `PATH`.
- **Debt deferral:** Keep the active delivery plan. Use the smallest check needed to classify an adjacent finding as blocking, deferrable, or speculative. Fix blockers. For deferrable debt, deduplicate open GitHub Issues, create or update one issue with `.github/ISSUE_TEMPLATE/debt.yml`, then resume the plan. Drop speculative findings until evidence exists. After deferral, do not audit adjacent code, implement the debt, expand tests, redesign, or start follow-up work in that delivery. Issues labeled `debt` are the sole backlog. Create a control task only after an issue is selected, and store it as `details.githubIssue: { number, url }`. Scheduled debt discovery follows the stability controller contract and never expands an active delivery. Do not use fixed follow-up quotas.
- **Before creating any new component or hook:** `SemanticSearch` or `Grep` the package for existing code that does the same thing. Duplication is never acceptable — if two surfaces need the same UI or logic, extract a shared primitive and have both import it.
- **Before shipping any feature:** Verify that every exported function/class you added or touched is actually _called_ from an appropriate entry point. Exported-but-never-called code is a bug. Grep for each new export name to confirm it appears in a consumer.
- **Platform copy:** Never write "for Mac", "for Windows", or "for desktop" in user-facing strings. The correct product name is "Freed Desktop". Use that.
- **Buttons and dialog controls:** Do not invent custom CTA styling when the repo already has an established button hierarchy. For dialogs, recovery surfaces, and other utility UI, use standard primary and secondary control treatment with the repo's usual sizing, radius, and typography. Never add hover lift, vertical motion, bounce, or "raise on hover" effects to buttons. Never introduce ad hoc glossy or gradient CTA buttons for utility UI.
- **UI design vocabulary:** Before adding or changing UI, inspect the nearest mature product surface and shared theme primitives first. Match the established vocabulary for tokens, radius, density, typography, borders, shadows, states, and responsive behavior. Introduce new styling only when the existing vocabulary cannot express the required behavior, and keep it compatible with all active themes.
- **Toolbar right-edge controls:** Any new control added to the right edge of a shared toolbar must also map into an existing overflow section or add a new named overflow section at the same time. The control must remain reachable at every supported desktop and mobile toolbar width. When controls collapse into a menu, form controls such as selects, radio groups, segmented controls, sliders, and toggles must fill the menu content width instead of keeping their inline toolbar width. If a control is wrapped by a tooltip or trigger element, that wrapper must also fill the menu content width. Add or update focused e2e coverage for both the inline state and the overflow state, including menu-section width assertions for collapsed form controls, before shipping.
- **Menu scroll bounds:** Any floating menu, command palette, context menu, overflow menu, or dropdown must scroll internally and stay inside the viewport. Use the shared `theme-menu-shell` class with a top or max-height CSS variable instead of raw `overflow-hidden`. Add or update focused e2e coverage when a menu can grow beyond the visible viewport.
- **Async-before-await is synchronous:** Code before the first `await` in an `async` function runs synchronously in the caller's microtask even if the caller doesn't await. Never put O(n) work (e.g. `Array.from(largeUint8Array)`, `A.save()`, serialization) before an `await` in a `subscribe()` callback or any other fire-and-forget async call on a hot path.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freed-project/freed](https://github.com/freed-project/freed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
