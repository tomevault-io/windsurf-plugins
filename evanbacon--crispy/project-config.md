---
trigger: always_on
description: Personal component library for building iOS-style universal apps with Expo Router. Extends the beautiful components that are built-in to Expo Router for iOS and adds customizable fallbacks for web and Android.
---

# Bacon UI

Personal component library for building iOS-style universal apps with Expo Router. Extends the beautiful components that are built-in to Expo Router for iOS and adds customizable fallbacks for web and Android.

## TODO

- [ ] Convert into more of a docs / gallery for different components.
- [ ] Add web components that mostly noop on native (add desktop later): tooltip, hover-card
- [ ] Fork stack on web to add support for new toolbar API in Expo Router v7. Stack should coordinate with custom tabs automatically to work like the ipad.
- [ ] Add ability for tab bar to group extra links together.
- [ ] Build out gallery + easy copy paste code snippets pages for things like segments.
- [x] Dialog component that is a component API around `alert()`.
- [ ] `popover` which uses a form-sheet on native.
- [ ] Components that are mostly just context menus on native: context-menu, dropdown-menu
- [ ] Echo components: anchor avatar badge button-group button card collapsible drawer label select separator skeleton
- [ ] Input-based components: form input-group input-otp input textarea
- [ ] Expo Router fallbacks: context-menu, sheet, toolbar
- [ ] Chat components: conversation, message, prompt-input, preview, speech to text, image-picker.
- [ ] Add prebuilt UIs as examples: AI chat page, dashboard, commerce page.
- [ ] Create complex `command` palette component.
- [ ] Make something like navigation-menu for web and toolbar + menu items for native.
- [ ] `sonner` component.
- [ ] `map` component.
- [ ] merge themes stuff into expo-router (?)

## Stack

- **Framework**: Expo SDK 55 beta with Expo Router 55 beta
- **Styling**: Tailwind CSS v4 + react-native-css + NativeWind v5 preview
- **Icons**: expo-symbols (SF Symbols) with fallback to @expo/vector-icons

## Project Structure

```
src/
├── app/              # Expo Router file-based routes
├── components/
│   ├── ui/           # Core UI components (form, switch, segments, etc.)
│   ├── layout/       # Layout components (tabs, stack, modal)
│   ├── runtime/      # Platform utilities (clipboard, local-storage)
│   └── example/      # Example/demo components
├── tw/               # CSS-wrapped primitives (View, Text, Image, etc.)
├── hooks/            # Custom hooks
├── css/              # CSS files (sf.css for Apple system colors)
├── lib/              # Utilities (cn helper)
└── constants/        # Font definitions per platform
```

## Key Patterns

### CSS-Wrapped Components (`src/tw/`)

All base RN components are wrapped with `useCssElement` from react-native-css to enable className support:

```tsx
import { View, Text, TextInput } from "@/tw";
```

### Form Components (`src/components/ui/form.tsx`)

SwiftUI-style List/Section/Item components:

```tsx
<List>
  <Section title="Settings">
    <Text>Label</Text>
    <Toggle value={on} onValueChange={setOn}>
      Toggle
    </Toggle>
    <Link href="/page">Navigate</Link>
  </Section>
</List>
```

### Apple System Colors

Use `sf-` prefixed Tailwind classes that map to iOS system colors:

- `text-sf-text`, `text-sf-text-2`, `text-sf-text-3` - Label hierarchy
- `bg-sf-bg`, `bg-sf-grouped-bg` - Backgrounds
- `text-sf-blue`, `text-sf-red`, etc. - Accent colors
- `border-sf-border` - Separators

Colors use `light-dark()` CSS function and `platformColor()` on iOS for native dynamic colors.

### SF Symbols

```tsx
import { SFIcon } from "@/components/ui/sf-icon";

<SFIcon name="gear" className="text-sf-blue" />;
```

### Platform-Specific Files

Use `.ios.tsx`, `.web.tsx`, `.android.ts` extensions for platform code. Examples:

- `switch.tsx` / `switch.web.tsx`
- `skeleton.tsx` / `skeleton.web.tsx`
- `fonts.ts` / `fonts.web.ts` / `fonts.android.ts`

### Theme Handling

Theme controlled via `color-scheme` CSS property. Add `.light` or `.dark` class to force theme, otherwise follows system preference.

### Path Aliases

```tsx
import { something } from "@/components/ui/something"; // src/components/ui/something
```

## Adding New Components

Follow these guidelines when creating new UI components to ensure consistency across the library.

### File Structure

Each component should have:

1. **Native implementation**: `src/components/ui/{component}.tsx` - Default/native behavior
2. **Web implementation**: `src/components/ui/{component}.web.tsx` - Web-specific with Radix UI primitives
3. **Example page**: `src/app/(index,info)/{component}.tsx` - Demo page with usage examples
4. **Documentation**: `docs/ui/{component}.md` - API reference and usage guide

### Platform Implementation Strategy

| Pattern | Native | Web |
|---------|--------|-----|
| Alerts/Dialogs | `Alert.alert()`, `Alert.prompt()` | Radix UI primitives |
| Tabs/Segments | `@react-native-segmented-control` | Radix Tabs |
| Context Menus | Native context menu APIs | Radix Context Menu |
| Switches | React Native `Switch` | Custom styled component |
| Accordion/Collapsible | react-native-reanimated (layout + fade) | Radix Accordion + CSS keyframes |
| Simple components | Direct implementation | Same or enhanced with hover states |

### Compound Component Pattern


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EvanBacon/crispy](https://github.com/EvanBacon/crispy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
