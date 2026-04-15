---
trigger: always_on
description: 1. NEVER use StyleSheet.create(). ALL styling via NativeWind className.
---

# AniUI — Cursor Rules
# shadcn/ui for React Native. Mobile-only. NativeWind v4 + Tailwind v3.

## Hard Rules
1. NEVER use StyleSheet.create(). ALL styling via NativeWind className.
2. NEVER use default exports. Named exports only.
3. NEVER use `any` type. Strict TypeScript.
4. NEVER install components as npm deps. Components are source files in components/ui/.
5. Always support className prop. Always spread ...props last.
6. Always set accessibilityRole on interactive elements.
7. Always wrap text inside Pressable in <Text>.
8. Min touch target: min-h-12 min-w-12 (48dp) on Pressable.
9. Import cn from "@/lib/utils" for className merging.
10. Mobile only — iOS + Android. No web.

## cn() Pattern
```tsx
import { cn } from "@/lib/utils";
// Usage: merge base styles with user overrides
<View className={cn("rounded-md bg-card p-4", className)} />
```

## Theme Tokens (Tailwind classes)
Colors from CSS variables in global.css. Use these class prefixes:
- bg-background, text-foreground (page background/text)
- bg-primary, text-primary-foreground (buttons, CTA)
- bg-secondary, text-secondary-foreground (secondary actions)
- bg-muted, text-muted-foreground (disabled, subtle)
- bg-accent, text-accent-foreground (highlights)
- bg-destructive, text-destructive-foreground (danger/error)
- bg-card, text-card-foreground (card surfaces)
- border-border (borders)
- bg-input (input backgrounds)
- ring (focus ring)

Dark mode: automatic via .dark class in global.css.

## Component Catalog (81 components)

Source of truth: `cli/src/registry.ts` (tiers 1–3). After `npx @aniui/cli add <name>`, install any extra npm packages the CLI prints.

Compound components export multiple named pieces (for example `Card` + `CardHeader`, `ToastProvider` + `useToast`, `Table` + `TableRow`). Import what you need from the same file as in the component source.

### Common variants (examples — see each file for full API)

- **Button** — default, secondary, outline, ghost, destructive, link × sm, md, lg
- **Text** — h1, h2, h3, h4, p, lead, large, small, muted
- **Input** — default, ghost × sm, md, lg
- **Textarea** — default, ghost
- **Badge** — default, secondary, outline, destructive
- **Alert** — default, destructive, success, warning
- **Toast** — default, destructive, success (via `useToast`)

### Registry tier 1 — 57 components

**Alert** — `import { Alert } from "@/components/ui/alert"`

**Avatar** — `import { Avatar } from "@/components/ui/avatar"`

**Badge** — `import { Badge } from "@/components/ui/badge"`

**Banner** — `import { Banner } from "@/components/ui/banner"`

**Button** — `import { Button } from "@/components/ui/button"`

**Calendar** — `import { Calendar } from "@/components/ui/calendar"`

**Card** — `import { Card } from "@/components/ui/card"`

**Carousel** — `import { Carousel } from "@/components/ui/carousel"`

**ChartTooltip** — `import { ChartTooltip } from "@/components/ui/chart-tooltip"`

**ChatBubble** — `import { ChatBubble } from "@/components/ui/chat-bubble"`

**Chip** — `import { Chip } from "@/components/ui/chip"`

**Combobox** — `import { Combobox } from "@/components/ui/combobox"`

**DatePicker** — `import { DatePicker } from "@/components/ui/date-picker"`

**Dialog** — `import { Dialog } from "@/components/ui/dialog"`

**EmptyState** — `import { EmptyState } from "@/components/ui/empty-state"`

**FAB** — `import { FAB } from "@/components/ui/fab"`

**FilePicker** — `import { FilePicker } from "@/components/ui/file-picker"`

**Form** — `import { Form } from "@/components/ui/form"`

**Grid** — `import { Grid } from "@/components/ui/grid"`

**Header** — `import { Header } from "@/components/ui/header"`

**Image** — `import { Image } from "@/components/ui/image"`

**ImageGallery** — `import { ImageGallery } from "@/components/ui/image-gallery"`

**InfiniteList** — `import { InfiniteList } from "@/components/ui/infinite-list"`

**Input** — `import { Input } from "@/components/ui/input"`

**InputOTP** — `import { InputOTP } from "@/components/ui/input-otp"`

**Label** — `import { Label } from "@/components/ui/label"`

**LabeledSeparator** — `import { LabeledSeparator } from "@/components/ui/labeled-separator"`

**List** — `import { List } from "@/components/ui/list"`

**MaskedInput** — `import { MaskedInput } from "@/components/ui/masked-input"`

**NumberInput** — `import { NumberInput } from "@/components/ui/number-input"`

**Pagination** — `import { Pagination } from "@/components/ui/pagination"`

**PasswordInput** — `import { PasswordInput } from "@/components/ui/password-input"`

**PhoneInput** — `import { PhoneInput } from "@/components/ui/phone-input"`

**Price** — `import { Price } from "@/components/ui/price"`

**ProgressSteps** — `import { ProgressSteps } from "@/components/ui/progress-steps"`

**Rating** — `import { Rating } from "@/components/ui/rating"`

**RefreshControl** — `import { RefreshControl } from "@/components/ui/refresh-control"`

**SafeArea** — `import { SafeArea } from "@/components/ui/safe-area"`

**SearchBar** — `import { SearchBar } from "@/components/ui/search-bar"`

**SegmentedControl** — `import { SegmentedControl } from "@/components/ui/segmented-control"`

**Select** — `import { Select } from "@/components/ui/select"`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anishlp7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
