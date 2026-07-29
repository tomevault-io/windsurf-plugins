---
trigger: always_on
description: The **keyboard focus halo**: the ring UIKit draws around the focused view during
---

# CLAUDE.md — Halo feature (iOS)

The **keyboard focus halo**: the ring UIKit draws around the focused view during
physical-keyboard / focus-engine navigation (`UIFocusHaloEffect`, iOS 15+). This
folder groups every halo-specific file; see the package-root
[CLAUDE.md](../../../CLAUDE.md) and [ios/CLAUDE.md](../../CLAUDE.md) for the wider
view system.

> iOS only. On Android the "halo" is the OS default focus highlight
> (`defaultFocusHighlightEnabled`), drawn by the framework — there is no
> custom-radius code.

## Structure

```
features/Halo/
  RNCEKVHaloProtocol.h                             contract (the halo props + getFocusTargetView)
  base/      RNCEKVExternalKeyboardHalloBase.{h,mm} view integration (base-chain layer)
  delegate/  RNCEKVHaloDelegate.{h,mm}              per-view effect cache + orchestration
  utils/     RNCEKVFocusEffectUtility.{h,mm}        stateless UIFocusEffect construction
```

| Role | File | What it does |
|---|---|---|
| contract | [RNCEKVHaloProtocol.h](RNCEKVHaloProtocol.h) | Capability the halo reads: `isHaloHidden`, `haloCornerRadius`, `haloExpendX/Y`, `roundedHaloFix` (deprecated), `getFocusTargetView`. |
| `base/` | [RNCEKVExternalKeyboardHalloBase](base/) | The **view-layer integration** — a link in the base chain (subclass of `RNCEKVViewOrderGroupBase`, superclass of `RNCEKVViewGroupIdentifierBase`). Owns one delegate and exposes the halo props + `customFocusEffect` for the pull getter. |
| `delegate/` | [RNCEKVHaloDelegate](delegate/) | Per-view cached builder. Turns the view's explicit halo props into an effect (via the util) and caches it; rebuilds only when bounds / radius / a prop changes. Keeps caching out of the view. |
| `utils/` | [RNCEKVFocusEffectUtility](utils/) | Stateless construction of the `UIFocusHaloEffect` — `emptyFocusEffect` (suppressed) and `getFocusEffect:withExpandedX:withExpandedY:withCornerRadius:` (rounded rect, continuous curve). |

## How it works (control flow)

The halo is **pull-based**: UIKit queries `focusEffect` on the focused view, and the
focused view routes that to the wrapper's `customFocusEffect`, computed on demand.

```
UIKit focuses the view
  → focus target's `focusEffect`
      • wrapper case: a focused child; its category override
        (ios/Extensions/RCTViewComponentView+RNCEKVExternalKeyboard.mm) returns parent.customFocusEffect
      • self-focused case (focusableWrapper=false / hidden): RNCEKVExternalKeyboardHalloBase's
        own `focusEffect` override returns customFocusEffect
  → RNCEKVExternalKeyboardHalloBase.customFocusEffect → RNCEKVHaloDelegate.focusEffect
  → builds (or returns cached) effect for getFocusTargetView via RNCEKVFocusEffectUtility

The halo is drawn on getFocusTargetView — the view the focus engine actually focused
inside the wrapper (RNCEKVFocusDelegate captures it from the focus update, see
ios/Delegates/RNCEKVFocusDelegate/), not a guessed first child.
```

`RNCEKVHaloDelegate.focusEffect` resolves to one of three deterministic results:

- **hidden** (`haloEffect={false}` / `tintType="none"`) → `emptyFocusEffect` (suppressed).
- **no custom settings** (no radius, no expand) → `nil` → UIKit draws its own default
  halo, which already tracks the view's bounds. (The "simple" case.)
- **radius / expand set** → a rounded-rect effect built from the focus target's bounds.

### Pull-only — no layout re-apply

The halo is **purely pull-based**: UIKit queries `focusEffect` at focus-update time
and the delegate computes it then. There is no `layoutSubviews`/`refreshHalo`
write-back — an earlier version re-pushed the effect on every layout to keep a static
effect tracking bounds, but it was a no-op in practice and is gone. Consequences:

- **Default case (nil effect)** — the common one: UIKit's own default halo tracks the
  focused view's bounds **and** corner radius for free, so a ring that should follow a
  rounded card comes from the view's own `borderRadius` + `borderWidth`
  (`layer.cornerRadius`), not from us. Round the card and the halo rounds with it.
- **Explicit `haloCornerRadius` / `haloExpend*` (custom effect)** — a *static*
  `UIFocusHaloEffect` built from the focus target's bounds at focus time. It does not
  re-track bounds if the view resizes *while focused*; a prop change re-invalidates the
  delegate so the next focus query rebuilds. This is deliberate — prefer driving the
  ring off `borderRadius`/`borderWidth` (the default path) over a static custom effect.

## Design notes / history

- **Radius is an explicit prop**, never observed off `layer.cornerRadius`.
  `haloCornerRadius` shapes the halo only; the view's own `borderRadius` (set via
  `containerStyle`) is a separate concern. The two are not inferred from each other.
- **This replaced an older system** that observed the live layer radius
  (`observeCornerRadius:`/`_stableRadius`) and re-armed the effect through a
  coalesced `dispatch_after` loop (`setNeedsHaloRearm`, `flushHaloRearm`,
  `_forceRearm`, plus a new-arch `invalidateLayer` override chasing RN's
  `layer.cornerRadius` flip-flop between its CA and image-border render paths).
  That machinery existed to out-run a transient `cornerRadius` toggle by re-arming
  ~20ms later and de-duping the resulting self-feeding layout loop. Pinning the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArturKalach/react-native-external-keyboard](https://github.com/ArturKalach/react-native-external-keyboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
