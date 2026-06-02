---
trigger: always_on
description: Guidelines for creating and updating components following components.build standards
---


## Methodology

Follow the [components.build](https://www.components.build/) specification for building modern, composable, and accessible UI components. Use the [Composition Pattern](https://www.radix-ui.com/primitives/docs/guides/composition).

## Artifact Taxonomy

Understand the hierarchy of UI artifacts we work with:

1. **Primitive** - Lowest-level building block providing behavior and accessibility without styling (e.g., Radix UI Primitives)
2. **Component** - Styled, reusable UI unit that adds visual design to primitives (e.g., shadcn/ui components)
3. **Pattern** - Specific composition solving a UI/UX problem (e.g., form validation with inline errors)

## Core Principles

### Composability and Reusability

- Favor composition over inheritance
- Build components that can be combined and nested
- Expose clear API via props/slots for customization
- Make components reusable in different contexts

### Accessible by Default

- Accessibility is not optional - it's a baseline feature
- See the detailed [Accessibility](#accessibility) section below for implementation guidelines

### Customizability and Theming

- Avoid hard-coding visual styles that cannot be overridden
- Provide mechanisms for theming (CSS variables, className, style props)
- Come with sensible defaults but allow easy customization
- Use design tokens for visual values (see [Styling](#styling) section)

### Lightweight and Performant

- Minimize dependencies
- Avoid bloating with unnecessary logic
- Strive for good rendering and interaction performance
- Minimize unnecessary re-renders

## Existing Components & Library Selection

**Decision order when building features:**

1. **Check existing UI components** (in order):
   - [React Native Reusables](https://reactnativereusables.com/docs) - UI components
   - [React Native Primitives](https://rn-primitives.vercel.app/) - Radix primitives
   - [RNR Community Resources](https://github.com/founded-labs/react-native-reusables/blob/main/COMMUNITY_RESOURCES.md) - Community examples

2. **Check Expo SDK** - [Expo SDK docs](https://docs.expo.dev/versions/latest/) for native APIs (camera, file system, haptics, etc.)

3. **Only then** consider third-party libraries or custom components.

## Composition Patterns

### Children / Slots

- **Children** (implicit slot): JSX between opening/closing tags
- **Named slots**: Props like `icon`, `footer`, or `<Component.Slot>` subcomponents
- **Slot forwarding**: Pass DOM attributes/className/refs through to underlying element

### Render Props (Function-as-Child)

Use when parent must own data/behavior but consumer must fully control markup:

```tsx
<ParentComponent data={data}>
  {(item) => <ChildComponent key={item.id} {...item} />}
</ParentComponent>
```

### Compound Components

Use separate component imports to compose complex UI (shadcn-style):

```tsx
import {
  Card,
  CardHeader,
  CardContent,
  CardFooter
} from '@/components/ui/card';

<Card>
  <CardHeader>Title</CardHeader>
  <CardContent>Body</CardContent>
  <CardFooter>Actions</CardFooter>
</Card>;
```

### Controlled vs. Uncontrolled

- **Controlled**: Value driven by props, emits `onChange` (source of truth is parent)
- **Uncontrolled**: Holds internal state, may expose `defaultValue` and imperative reset
- Many inputs should support both patterns

### Polymorphism / asChild

Use `asChild` prop to allow component to render as a different element:

```tsx
<Button asChild>
  <Link href="/">Click me</Link>
</Button>
```

This renders as `<Link>` instead of `<button>`, preserving all Button behavior.

## Accessibility

### Keyboard Navigation

- Document and implement keyboard map for every interactive component
- Support standard patterns: `Tab`, `Arrow keys`, `Home/End`, `Escape`
- Ensure all interactive elements are keyboard accessible

### Focus Management

- Rules for initial focus, roving focus, focus trapping
- Focus return on teardown (e.g., modals)
- Focus indicators visible and clear

### ARIA Attributes

- Use semantic HTML elements (`<button>`, `<ul>/<li>`, etc.)
- Augment with ARIA when necessary:
  - `role` - Communicate semantics (`role="menu"`)
  - `aria-*` states - State (`aria-checked`, `aria-expanded`)
  - `aria-*` properties - Relationships (`aria-controls`, `aria-labelledby`)

### Color Contrast

- Ensure sufficient color contrast for text and interactive elements
- Don't rely solely on color to convey information

## Implementation Notes

### Styling

- Don't use margin (`m-*` properties in tailwind), use `View`s with flex (`flex`), flex gap (`gap-*`), and flex direction (`flex-[row|col]`).
- Don't concatenate tailwind classnames with template strings, always use the `cn` utility in `utils/styleUtils.ts`.
- Don't use Tailwind leading-none or anything with a line-height less than 1.3, as it causes text clipping.
- Don't use `SafeAreaView` around `ScrollView` or `FlatList`. Instead, always use `contentInsetAdjustmentBehavior="automatic"` prop on the scrollable component itself.
- Use **variants** for discrete style/behavior permutations (e.g., `size="sm|md|lg"`, `tone="neutral|destructive"`). Variants are not separate components.
- Use **design tokens** (CSS variables) for theming: `--color-bg`, `--radius-md`, `--space-2`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [genesis-ai-dev/langquest](https://github.com/genesis-ai-dev/langquest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
