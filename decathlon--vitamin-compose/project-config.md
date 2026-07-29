---
trigger: always_on
description: This page describes guidelines that all contributions need to follow to be accepted in the codebase.
---

# Compose Contract convention

This page describes guidelines that all contributions need to follow to be accepted in the codebase.
Please, read it and apply all points in your contribution.

## Naming conventions

**Glossary**
* `Component` is the name of the component from the design specification on the Vitamin Figma project.
* `Property` can be colors, sizes, borders or any concept which can customize the component from a data class.
* `Variant` is a sub component. You need at least one variant to use it inside your module component.

**Object access**
Each component must be accessible from an object with the following specification for the name
`Vitamin<Component>s`. e.g. `VitaminButtons`

If your component can be customized by properties, you must create an object per property with the
following specification for the name `Vitamin<Component><Property>s`. e.g. `VitaminButtonColors`

If you component have only one variant and if there isn't any name proposed by the specification
in the Figma project, the composable inside the object access must be named `Primary`. e.g. for
a component with only one variant `VitaminRadioButtons.Primary`.

When you create a common component to centralize the UI logic, it must be `private` and with the
following specification for the name `Vitamin<Component><Variant>?Impl` where the variant
is used when you can't have only one common component. e.g. `VitaminProgressBarLinearImpl`.

**Colors and sizes object access**
When you need to create an object access for colors configurations but you have only one variant,
it should be named by `primary`, excepted if the design specification defines a name in the Figma
project.

When you need to create an object access for sizes configurations but you have only one variant,
it should be named `medium`, excepted if the design specification defines the size in the Figma
project.

For colors and sizes, functions for each variant shouldn't contains `Colors` and `Sizes` keywords.
e.g. `Vitamin<Component>Colors.primary()` for colors and `Vitamin<Component>Sizes.medium()` for 
sizes. Functions arguments should be suffixed with `Color`, e.g. `backgroundColor`



## Composable variants

A component can be declined in multiple variants. There are two options: create
a component by variant or create one component which can be customized in all variants. If it is
the case, follow these rules to know which one you need to create:

If you need to change dynamically from one variant to another, you should have only one component.
e.g. the top appbar which can be changed from primary to contextual and vice versa.

```kotlin
@Composable
fun Primary(
    /* parameters */
    colors: TopBarsColors = VitaminTopBarsColors.primary(),
)
```

If not, you create a component for each variant to avoid any confusion from the developer. But for
this option, you can create a private common component. This component should follow the naming
convention described in the adapted section and should be declared outside the public object.

## Parameter ordering

When you build a component, it should follow Vitamin guidelines described in the Figma project
but to be compatible with real life issues, it should be fully customizable and easily. To allow
this kind of think, you need to define the specification as default value in composable parameters.

These parameters need to follow an ordering to avoid differences in the contract of each component.
Find here this ordering:

* **Required parameters**: There isn't default value and the developer need to specify a value for them. Try to have as little as possible.
* **Modifier**: Recommended by Compose itself, the modifier should be after required parameters. The name of the parameter is `modifier`.
* **Data and settings parameters**: Some internal components can use parameters with default value because it is specified by Vitamin.
* **Colors**: If you need to apply a set of colors on your component, you need to define `colors` parameter with a default value to the adapted color.
* **Sizes**: If you need to apply a padding, a text style or a width/height inside your component, you need to define `sizes` parameter with a default value to the phone size.
* **Shapes**: If you need to apply a shape on some internal components, you need to define `shapes` parameter with a default value from `VitaminTheme.shapes`. 
* **Borders**: If you need to apply a border around your component, you need to define `borders` parameter with a default value to the adapted default border.
* **Ripple**: If your component need to have a user feedback with a ripple effect, you need to define `ripple` parameter with a default value from `VitaminTheme.ripples`.
* **Elevation**: If your component need an elevation to display a shadow below the component, you need to define `elevation` parameter with a default value in `Dp`.
* **Callbacks**: For each event defined in the specification and where the developer can be interested, you need to define a callback by event.
* **Compose Slotting**: For each api slot you define in your component, follow the api slotting section to know how to create them. 

> **Note:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Decathlon/vitamin-compose](https://github.com/Decathlon/vitamin-compose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
