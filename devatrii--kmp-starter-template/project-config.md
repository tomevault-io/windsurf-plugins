---
trigger: always_on
description: These rules must be **strictly followed** to maintain code quality, consistency, and best practices in Jetpack Compose development.
---


# Jetpack Compose Development Rules  

These rules must be **strictly followed** to maintain code quality, consistency, and best practices in Jetpack Compose development.  

---

## 1. Enum Handling  
- Always use `Enum.entries` instead of `Enum.values()` when retrieving all entries of an enum.  
- This ensures better performance and readability in newer versions of Kotlin.  

---

## 2. Design Guidelines  
- **Material 3 Compliance**: All UI elements should strictly follow **Material 3** guidelines.  
- **iOS Influence**: UI/UX should follow **iOS Cupertino design principles**, but must maintain a **Material 3 appearance**.  
- **Consistency**: Maintain a **unified design language** throughout the app. Avoid mixing multiple styles.  

---

## 3. Animations  
- **Always prefer `graphicsLayer`** when animating properties such as **scale, alpha, translation, rotation, or shadow**.  
- Example: Instead of using `Modifier.scale()`, use:  
  ```kotlin
  Modifier.graphicsLayer(scaleX = scale, scaleY = scale)
  ```
- **Use `animate*AsState` where applicable** instead of handling animations manually.  
- **Avoid excessive recompositions**: Store animation states in `remember { mutableStateOf(...) }` when needed.  
- **Prefer `Crossfade`** for smooth UI transitions when switching between composables.  
- **Use `AnimatedVisibility`** for enter/exit animations instead of manual show/hide logic.
- **Implement spring animations** with `spring()` for natural motion effects.

---

## 4. Component Organization  
- Every new UI component must have a **dedicated file inside the `components/` folder**.  
- File naming convention: **`ExampleComponent.kt`**  
- Always **include the package name** at the top of the file.  
- Components should be:  
  - **Reusable** across screens.  
  - **Composable** functions without direct dependencies on ViewModels or business logic.  
  - **Self-contained**, handling only UI-related tasks.  
- **Create component previews** for all custom components to aid development.

---

## 5. Icons  
- Do **not** use Material icons that do not exist in **Material 3**.  
- If an icon is missing from Material 3, use its alternative icon from Material 3.
- **Use vector drawables** for custom icons instead of bitmap images.
- **Prefer `Icon` composable** over `Image` for icon rendering.

---

## 6. Screen Structure  
Each screen must follow a **consistent structure**:  

- **`ScreenNameScreenContent`**  
  - Handles all UI elements for the screen.  
  - **Does not handle state management** (except for bottom sheets, dialogs, and popups).  
  - All parameters should be passed via the constructor.  
- **`ScreenNameScreen`**  
  - Wrapper composable that integrates **ViewModel, state management, and UI interactions**.  
- **Preview Composable**  
  - Must have `showSystemUi = true` to provide a full-screen preview in Android Studio.  
  - Example:
    ```kotlin
    @Preview(showSystemUi = true)
    @Composable
    fun ScreenNameScreenPreview() {
        ScreenNameScreen()
    }
    ```

---

## 7. Folder Structure (inside `ui_main/`)  
All files should be **properly categorized** inside the `ui_main/` package of that presentation layer:  
```plaintext
ui_main/
│── components/    # Reusable UI components  
│── screens/       # Screens and UI logic  
│── dialogs/       # Dialogs (alert, confirmation, input, etc.)  
│── bottom_sheets/ # Bottom sheets for additional actions  
│── layouts/       # Custom layout components  
│── modifiers/     # Custom modifier extensions
```  
- **Place each file in the correct directory** based on its purpose.  
- **Avoid keeping too many files in one folder**; refactor when necessary.  

---

## 8. Composables Best Practices  
- **Responsive & Professional UI**  
  - UI elements must **scale properly** across different screen sizes.  
  - Prefer **weight-based layouts** (`Modifier.weight()`) over hardcoded dimensions.  
  - **Test in different screen densities** (e.g., compact, normal, expanded).  
  - **Use `WindowInsets` properly** for edge-to-edge layouts.

- **Consistent Modifier Placement**  
  - **Always** place `modifier` as the **first parameter** in composable functions.  
  - Example:
    ```kotlin
    @Composable
    fun CustomButton(
        modifier: Modifier = Modifier,
        text: String,
        onClick: () -> Unit
    ) { ... }
    ```

- **State Management Best Practices**  
  - **`ScreenNameScreenContent` must not handle state management**.  
  - **State should be lifted** to the parent (`ScreenNameScreen`).  
  - **Use `rememberSaveable` instead of `remember`** where applicable to persist UI state across configuration changes.  

---

## 9. Performance Optimizations  
- **Minimize recompositions**  
  - Use `remember` and `rememberSaveable` to avoid unnecessary recompositions.  
  - Example:
    ```kotlin
    val textFieldState = rememberSaveable { mutableStateOf("") }
    ```
- **Use `LazyColumn` instead of `Column` for large lists**.  
- **Prefer `produceState`, `rememberCoroutineScope` or `LaunchedEffect` for side effects** instead of manual coroutine handling.  
- **Use keys in Lazy lists** (`key = { item.id }`) to avoid recompositions when scrolling.  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DevAtrii/Kmp-Starter-Template](https://github.com/DevAtrii/Kmp-Starter-Template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
