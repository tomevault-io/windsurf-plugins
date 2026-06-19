---
trigger: always_on
description: Enforces WCAG 2.2 accessible coding patterns when writing SwiftUI — labels, traits, Dynamic Type, contrast, touch targets, focus management, orientation, and more. Based on the ios-swiftui-accessibility-techniques project with 37 static analysis rules across 19 WCAG criteria.
---


# iOS SwiftUI Accessibility Coding Rules

Follow these rules when writing or reviewing SwiftUI code. Each rule maps to a WCAG 2.2 success criterion.

## Images (WCAG 1.1.1)

- Every `Image(systemName:)` and `Image("name")` must have `.accessibilityLabel("description")` describing what the image conveys.
- For decorative images that add no information, use `Image(decorative:)` or add `.accessibilityHidden(true)`.
- Never include words like "image", "icon", "graphic", or "button" in an `.accessibilityLabel` — VoiceOver already announces the element's trait.
- Describe what the image shows, not the file name or technical details.

```swift
// Good
Image(systemName: "heart.fill")
    .accessibilityLabel("Favorite")

// Good — decorative
Image(decorative: "background-pattern")

// Bad — no label
Image(systemName: "trash")

// Bad — includes role
Image(systemName: "heart.fill")
    .accessibilityLabel("Heart icon")
```

## Buttons (WCAG 4.1.2)

- Icon-only `Button` views must have `.accessibilityLabel("action")` describing what the button does.
- Never include "button" in the `.accessibilityLabel` — VoiceOver announces the button trait automatically, so users hear "Delete button, button".
- Prefer `Button` over `.onTapGesture`. If you must use `.onTapGesture`, add `.accessibilityAddTraits(.isButton)` so VoiceOver announces it as a button.
- Visually disabled buttons (using `.opacity()` or `.tint(.gray)`) must also use `.disabled(true)` so assistive technology knows the button is disabled.

```swift
// Good
Button(action: { deleteItem() }) {
    Image(systemName: "trash")
}
.accessibilityLabel("Delete")

// Bad — label includes role
Button(action: {}) {
    Image(systemName: "trash")
}
.accessibilityLabel("Delete button")

// Bad — looks disabled but isn't semantically
Button("Submit", action: {})
    .opacity(0.5)
// Fix:
Button("Submit", action: {})
    .disabled(true)
```

## Accessibility Label (WCAG 4.1.2, 2.5.3)

- Every interactive control must have a meaningful accessible name — either from visible text content or `.accessibilityLabel`.
- Labels should be concise (ideally 1–3 words), begin capitalized, and not end with a period.
- The `.accessibilityLabel` must start with the visible label text so Voice Control users can activate the element by saying what they see (Label in Name — WCAG 2.5.3).
- Never include the control type in the label ("button", "tab", "link", "image").

```swift
// Good — specific label includes visible text first
Button("Add to cart") {}
    .accessibilityLabel("Add to cart, Wireless Headphones")

// Bad — accessible name doesn't contain visible text
Button("Add to cart") {}
    .accessibilityLabel("Purchase Wireless Headphones")
```

## Accessibility Value (WCAG 4.1.2)

- Use `.accessibilityValue` on custom controls to convey their current state or value (e.g., "3 out of 5", "Step 2 of 4", "enabled"/"disabled").
- Pair adjustable custom controls with `.accessibilityAdjustableAction` so VoiceOver users can swipe up/down to change the value.
- Use `.accessibilityValue` on tab bar items to convey badge notification counts (e.g., "3 notifications").

```swift
// Good — custom star rating
HStack {
    ForEach(1...5, id: \.self) { star in
        Image(systemName: star <= rating ? "star.fill" : "star")
    }
}
.accessibilityElement(children: .ignore)
.accessibilityLabel("Rating")
.accessibilityValue("\(rating) out of 5")
.accessibilityAdjustableAction { direction in
    switch direction {
    case .increment: rating = min(5, rating + 1)
    case .decrement: rating = max(1, rating - 1)
    @unknown default: break
    }
}
```

## Accessibility Hint (WCAG 3.3.2)

- Hints are optional for standard buttons and taps — VoiceOver users can turn them off. Only add a hint when the result of activating an element is not obvious from the label alone.
- **Hints are required** for elements with non-obvious interactions: `.onLongPressGesture`, `.onDrag`, `.onDrop`, `.contextMenu`, `.swipeActions`, `DragGesture`, and `LongPressGesture`. Without a hint, VoiceOver users won't know these interactions exist.
- Use third-person singular verb describing the result: "Adds this item to your favorites." NOT "Add this item to your favorites."
- Never describe the gesture method ("tap", "double tap", "swipe") — VoiceOver already tells users how to interact. Describe what happens, not how to do it.
- Never repeat the label text or include the control type ("button", "link").
- Begin capitalized, end with a period.

```swift
// Good
Button(action: { toggleFavorite() }) {
    Image(systemName: "heart")
}
.accessibilityLabel("Favorite")
.accessibilityHint("Adds this item to your favorites.")

// Bad — describes the gesture method
.accessibilityHint("Double tap to add this item to your favorites.")

// Bad — repeats label
.accessibilityHint("Favorite")

// Good — hint required for long press
Text(item.name)
    .onLongPressGesture { showActions() }
    .accessibilityHint("Shows available actions.")

// Good — hint required for context menu
Button(item.name) { selectItem() }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cvs-health/ios-swiftui-accessibility-techniques](https://github.com/cvs-health/ios-swiftui-accessibility-techniques) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
