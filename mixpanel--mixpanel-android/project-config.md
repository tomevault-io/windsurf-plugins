---
trigger: always_on
description: User-facing docs live in [README.md](README.md). This file captures internal
---

# Mixpanel Android Session Replay — Internal Notes

User-facing docs live in [README.md](README.md). This file captures internal
design decisions for the wireframe/AI-summary work so they aren't re-litigated.

## Wireframe Capture Notes (2026-07-29)

**The `mp_wireframe` payload shape is shared across Android, Flutter, and iOS.**
The supported `role` values are not a cross-platform contract: each SDK reports
the semantic roles its platform exposes. This section records how the Android
implementation behaves and where the reasoning isn't obvious from the code.

- **Android currently emits ten semantic roles:** `text`, `button`, `input`,
  `image`, `link`, `header`, `checkbox`, `switch`, `radio`, and `tab`
  (`WireframeType`). The vocabulary may evolve independently on each platform.
  Layout/containers (`ViewGroup`/`LinearLayout`/
  `ConstraintLayout`, Compose `Box`/`Row`/`Column`) are never emitted —
  `classifyAndroidView` returns null for them and `collectWireframeForNode` hits
  `else -> return`. The payload is a flat list of `{role, text?, bounds}`, not a
  view hierarchy.
- **Every collected element is emitted, even when textless.** A textless
  `button`/`input`/`image` is meaningful structure — e.g. two textless `input`
  shells + a `Log in` button reads as a login form. Existence + position + role
  is not customer content, so an element is never dropped merely for lacking
  text. (Input fields are always textless by security design — `TEXT_ENTRY`.)
- **Text population depends on screenshot masking, not element type:**
  - *Not screenshot-masked* → `text` = visible text; if absent, fall back to the
    platform accessibility label (Android `contentDescription` / Compose
    `ContentDescription`; Flutter `semanticLabel` / tooltip), unless
    `WireframesOptions.useAccessibilityLabelFallback` is off. Then run user
    `SensitiveRule`s over the result. Rationale: if it's visible it's already in
    the unmasked screenshot; the customer's lever is masking the view.
  - *Screenshot-masked* (explicit mask, auto text/image mask, geometric overlap,
    input `TEXT_ENTRY`) → keep the `role + bounds` shell with `text = null`.
    Nothing hidden on the screenshot leaves the device.
- **Masking vs. rules are distinct.** A *screenshot mask* grays pixels → drop
  text (shell kept). `SensitiveRule`s are a wireframe-only text filter (pixels
  still visible) → element kept; strip nulls text, redact rewrites it.
- **Accessibility labels are opt-in (reversed 2026-08-24; was opt-out, decided
  2026-08-14).** `WireframesOptions.useAccessibilityLabelFallback`, default
  **`false`** on all three platforms. The residual risk is what decided it: a
  label can describe more than what's visible (an icon whose
  `contentDescription` holds PII), and because it is never drawn, the "mask what
  you can see in the replay" workflow can't reach it — so the customer has no
  way to audit it, and a default that ships un-auditable text is the wrong
  default. The cost is understood and accepted: with the fallback off every
  icon-only control (nav bars, `IconButton`, FABs, toolbar actions) is a bare
  shell, and naming those is much of what an AI summary wants from a toolbar.
  Customers who want them named set it `true`, or describe elements with
  `mpWireframeText(...)`, which the flag deliberately does not gate — declared
  text is authored, not scraped. `SensitiveRule`s run over label text either
  way, and screenshot-masked views never expose labels.
  - **The internal mirrors default off too**, so "nobody set it" and "the
    customer took the default" behave the same: `SensitiveViewManager
    .useAccessibilityLabelFallback` and what `deinitialize()` resets it to
    (Android), `SensitiveViewManager.shared` (iOS), and the `MaskDetector` /
    `ScreenshotCapturer` parameters (Flutter).
  - **The golden harnesses force it on** (Android `resetMaskingState`, iOS
    `WireframeGoldenTestUtils`, Flutter `golden_test_utils`) and the
    `*_fallbackOff_*` cases turn it back off per-case, so the fixtures still
    cover both tiers and none of them had to be re-recorded.
  - The label is only ever the *third* tier: declared `wireframeText` → visible
    text → label. That ordering is what makes "fallback" accurate even for
    images, which have no visible text of their own.
  - With the fallback off, a **Compose** node whose only content is a label is
    dropped rather than emitted as an empty text shell — the label was the sole
    evidence it was content at all, so keeping a shell would emit the labeled
    *containers* the recognized-role rule above excludes. Nodes carrying a role
    (`Role.Image`, `Role.Button`) still emit textless, as does every Android
    view, since `classifyAndroidView` never consults the label.

## Icon-glyph normalization (added 2026-08-17)

`WireframeEmitter.isHumanReadable` nulls text made *entirely* of Unicode
private-use-area codepoints (U+E000–U+F8FF), where icon fonts live — a Material
icon `TextView` would otherwise ship its glyph as element text and hand the
summarizer garbage. The element keeps its role + bounds shell; only the text is
dropped. Any single readable character keeps the whole string, so "Settings ⚙"
survives.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mixpanel/mixpanel-android](https://github.com/mixpanel/mixpanel-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
