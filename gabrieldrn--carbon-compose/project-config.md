---
trigger: always_on
description: This is a **Kotlin Multiplatform (KMP) Compose library** implementing [IBM's Carbon Design System](https://carbondesignsystem.com/).
---

# carbon-compose — Claude Context

This is a **Kotlin Multiplatform (KMP) Compose library** implementing [IBM's Carbon Design System](https://carbondesignsystem.com/).

**Carbon is a standalone design system. Never use Material3 or any other design system alongside it.**

---

## Module structure

| Module | Purpose |
|---|---|
| `carbon` | All UI components (main library, published) |
| `carbon-common` | Shared semantic utilities (published) |
| `carbon-test` | Shared test helpers (published) |
| `catalog` / `catalog-android` | Demo apps (all platforms) |
| `code-gen` / `doc-parser` | Build tooling — token code generation |

**Targets:** Android, iOS (arm64 + simulatorArm64), Desktop (JVM), WasmJS.

Build convention plugins: `id("carbon.kmp.library")` and `id("carbon.detekt")`.

---

## Never do

- **No Material3 or any other design system** — Carbon is standalone; never import `androidx.compose.material3.*` or equivalent
- **No hardcoded `dp` values** for spacing — always use `SpacingScale.*`
- **No hardcoded colors** — always derive from `Carbon.theme` tokens
- **No `com.gabrieldrn.carbon.foundation.text.Text`** internal wrapper — use `BasicText` from `androidx.compose.foundation.text`
- **No auto-generated resources** — add fonts/drawables/strings manually to the resources directory

---

## Component implementation workflow

1. **Consult the Component Implementation Reference** section below — all patterns are documented. Only read an existing component directly if encountering an edge case not covered there.
2. **Consult spec**: `https://carbondesignsystem.com/components/<name>/usage/`
3. **Implement files** per anatomy below
4. **Always implement the catalog demo screen** in the same task (see Catalog workflow)

---

## Component anatomy

Components live in:
```
carbon/src/commonMain/kotlin/com/gabrieldrn/carbon/<component>/
```

File breakdown is **flexible** — apply Separation of Concerns based on complexity:

| File | Visibility | Purpose |
|---|---|---|
| `ComponentName.kt` | `public` | Main composable(s) + KDoc + `@Preview` region |
| `ComponentNameColors.kt` | `internal` | Color logic derived from `Theme` |
| `ComponentNameSize.kt` | `public expect` | `expect enum class`, actual per platform |
| `ComponentNameTestTags.kt` | `internal` | `object` with `const val` tag strings |
| `ComponentNameImpl.kt` | `internal` | Layout/drawing implementation, separated from public API |

Simpler components may need fewer files; complex ones may split further (e.g., `domain/`, `base/` subpackages).

---

## Component Implementation Reference

### Composable signature conventions

Parameter order (strict):
1. Required state/value params (`value`, `checked`, `selected`, `state`)
2. Required callback params (`onClick`, `onValueChange`, `onToggleChange`)
3. `modifier: Modifier = Modifier`
4. Optional content params (label, placeholder, helperText, iconPainter)
5. Optional type/size/state enums — all with defaults
6. `interactionSource: MutableInteractionSource = remember { MutableInteractionSource() }` — always last

```kotlin
@Composable
public fun Toggle(
    isToggled: Boolean,                          // 1. required state
    onToggleChange: (Boolean) -> Unit,           // 2. required callback
    modifier: Modifier = Modifier,               // 3. modifier
    label: String = "",                          // 4. optional content
    actionText: String = "",
    interactiveState: ToggleInteractiveState = ToggleInteractiveState.Default,  // 5. optional enums
    toggleType: ToggleType = ToggleType.Default,
    interactionSource: MutableInteractionSource = remember { MutableInteractionSource() }  // 6. always last
)
```

---

### Color class pattern

Full canonical template:

```kotlin
@Immutable
internal class ComponentColors private constructor(
    private val theme: Theme,
    private val variant: ComponentVariant,  // add per-color dependencies as constructor params
) {
    // Static colors — computed once from theme tokens
    val containerColor: Color = when (variant) {
        ComponentVariant.Primary -> theme.buttonPrimary
        ComponentVariant.Secondary -> theme.buttonSecondary
    }

    // Dynamic colors — @Composable returning State<Color> via rememberUpdatedState
    @Composable
    fun borderColor(state: ComponentState): State<Color> =
        rememberUpdatedState(newValue = when (state) {
            ComponentState.Disabled -> Color.Transparent
            ComponentState.ReadOnly -> theme.borderSubtleColor(Carbon.layer)
            else -> theme.borderStrongColor(Carbon.layer)
        })

    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (other !is ComponentColors) return false
        if (theme != other.theme) return false
        if (variant != other.variant) return false
        return true
    }

    override fun hashCode(): Int {
        var result = theme.hashCode()
        result = 31 * result + variant.hashCode()
        return result
    }

    companion object {
        @Composable
        fun colors(variant: ComponentVariant): ComponentColors =
            ComponentColors(Carbon.theme, variant)
    }
}
```

Rules:
- `private constructor` always

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabrieldrn/carbon-compose](https://github.com/gabrieldrn/carbon-compose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
