---
trigger: always_on
description: Comprehensive audit across 7 parallel review agents examining the full extraction and interaction pipeline for bugs where the agent's perception diverges from reality.
---

# Snapshot & Interaction Pipeline — Audit Results

Comprehensive audit across 7 parallel review agents examining the full extraction and interaction pipeline for bugs where the agent's perception diverges from reality.

## Critical (agent can't complete tasks)

### C1. Iframe element coordinates are iframe-relative, not viewport-relative
**Agent 1, Finding 3** | `layout-extractor.ts:413-458`
Batch path uses `getBoundingClientRect()` inside iframes, returning iframe-local coordinates. `Input.dispatchMouseEvent` expects viewport-relative. Clicks on iframe elements (cookie banners, embedded forms) hit wrong positions.

### C2. Cross-origin iframe elements can't be resolved from main CDP session
**Agent 3, Finding 2** | `element-resolver.ts`, `session-manager.ts:499`
CDP session is created once per page. `backendNodeId` for cross-origin iframe elements can't be resolved via `DOM.resolveNode`/`DOM.getBoxModel` from the main frame session. Cookie consent banners (OneTrust, Cookiebot) are common cross-origin iframes.

### C3. EID collision suffix shifts silently when DOM order changes
**Agent 5, Finding 2+7** | `element-identity.ts:132-143`, `element-registry.ts:67-80`
Collision resolution assigns `-2`, `-3` by iteration order. If elements reorder between snapshots, suffixes shift and the agent silently acts on the wrong element. Common with sorted lists, infinite scroll, and any dynamic reordering.

### C4. Stale element retry picks first label+kind match with no disambiguation
**Agent 5, Finding 6** | `stale-element-retry.ts:64-65`
`nodes.find(n => n.label === node.label && n.kind === node.kind)` returns the first match. For pages with repeated elements ("Add to cart" x10), the wrong element is targeted silently.

## Major (agent works around it poorly)

### M1. `opacity:0`, `clip-path`, `transform:scale(0)` not detected as invisible
**Agent 1, Finding 1** | `layout-extractor.ts:123-140`
`computeVisibility()` only checks `display:none`, `visibility:hidden`, and zero bbox. Invisible elements (loading spinners with opacity:0, off-screen transforms) reported as visible. Agent tries to interact with them.

### M2. Batch vs. fallback coordinate space mismatch
**Agent 1, Finding 4** | `layout-extractor.ts`
Batch path returns viewport-relative coords (`getBoundingClientRect`), fallback returns page-relative (`DOM.getBoxModel`). Mixed coordinate spaces in the same snapshot when some elements fall back.

### M3. `<label>` wrapping hidden inputs dropped from snapshot
**Agent 1, Finding 5** | `types.ts:328-346`, `snapshot-compiler.ts:299-310`
`LabelText` AX role is not in any role set. Labels without click listeners/cursor:pointer are silently dropped. Agent can't see the visual click target for custom radio/checkbox controls.

### M4. Disabled override false positive — analytics click listeners
**Agent 2, Finding 1** | `state-extractor.ts:137-145`
Elements with analytics tracking click listeners (GTM, etc.) are marked `enabled=true` even when truly disabled. Also applies to ancestor delegation listeners (common in React/Vue).

### M5. `aria-pressed` for toggle buttons not mapped
**Agent 2, Finding 2** | `state-extractor.ts` (absent)
No `pressed` field in `NodeState`. Toggle buttons (bold/italic, mute, dark mode) lose their toggled state. Agent can't determine current state.

### M6. `contenteditable` elements indistinguishable from regular inputs
**Agent 2, Finding 4** | `kind-mapping.ts:39`
Both get `kind:'input'`. Agent can't tell a rich text editor from a text field. May be missed entirely if AX doesn't assign `textbox` role.

### M7. `aria-disabled="false"` can't override DOM `disabled`
**Agent 2, Finding 6** | `state-extractor.ts:120-135`
Per WAI-ARIA spec, `aria-disabled="false"` should override native `disabled`. Both AX-first and fallback paths get this wrong.

### M8. Missing `mouseMoved` before click
**Agent 3, Finding 1** | `element-resolver.ts:251-282`
`clickAtCoordinates` sends `mousePressed`+`mouseReleased` without `mouseMoved`. Hover-gated handlers (React `onMouseEnter`, CSS `:hover` gates, tooltip-activated buttons) won't fire.

### M9. Type tool click-then-type race condition
**Agent 3, Finding 3** | `element-resolver.ts:358-374`
`typeByBackendNodeId` clicks to focus, then inserts text. If click triggers navigation/modal/focus steal, text goes to wrong target. No guard between click and insert.

### M10. `selectOption` event not compatible with React/Vue
**Agent 3, Finding 4** | `element-resolver.ts:446-461`
Dispatches `change` but not `input` event. `new Event('change')` is untrusted and bypasses React's value tracking. Vue's `v-model` on selects listens for `input`. Select changes may not be detected by frameworks.

### M11. Coordinate-based click has no scroll guard
**Agent 3, Finding 8** | `interaction-tools.ts:88-98`
Click with raw `x,y` (no eid) has no `scrollIntoViewIfNeeded`. If page scrolled since snapshot, click silently hits the wrong element.

### M12. EID label changes cause phantom add/remove in diff
**Agent 4, Finding 1** | `element-identity.ts:36-47`, `diff-engine.ts:98-112`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lespaceman/agent-web-interface](https://github.com/lespaceman/agent-web-interface) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
