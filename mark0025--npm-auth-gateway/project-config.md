---
trigger: always_on
description: Defines how to use shadcn/ui components - never modify originals, always wrap and extend. Clear boundaries between Tailwind utilities and shadcn/ui components.
---


# shadcn/ui Component Strategy

This rule teaches **when to use shadcn/ui** vs **when to use Tailwind utilities**, and **how to extend components** without breaking updates.

**Core Principle**: shadcn/ui components are **source code you own**, not a package. Treat them as a **starting point**, not a final destination.

---

## 🎯 The Golden Rule

### ✅ ALWAYS: Import → Wrap → Extend

```tsx
// ✅ CORRECT: Import and wrap
import { Button } from "@/components/ui/button"

export function AwesomeButton({ children, ...props }: AwesomeButtonProps) {
  return (
    <Button
      className="bg-gradient-to-r from-purple-500 to-pink-500 hover:from-purple-600 hover:to-pink-600"
      {...props}
    >
      {children}
    </Button>
  )
}

// Use your wrapped version
<AwesomeButton>Click me</AwesomeButton>
```

### ❌ NEVER: Modify shadcn/ui Source Files

```tsx
// ❌ WRONG: Modifying ui/button.tsx directly
// File: components/ui/button.tsx
const buttonVariants = cva(
  "inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        // ❌ DON'T ADD YOUR CUSTOM VARIANTS HERE
        awesome: "bg-gradient-to-r from-purple-500 to-pink-500", // ❌ WRONG
      },
    },
  }
)
```

**Why this is wrong**:
- Breaks when you update shadcn/ui components
- Makes it hard to track custom vs. library code
- Difficult to share across projects
- Can't easily revert to defaults

---

## 🚦 When to Use What

### Use shadcn/ui Components When You Need:

✅ **Complex Interactive Components**
- Forms (Input, Select, Checkbox, Radio, Switch)
- Overlays (Dialog, Popover, Tooltip, Sheet)
- Navigation (Tabs, Accordion, Dropdown Menu)
- Data Display (Table, Card with multiple parts)
- Feedback (Alert, Toast, Progress)

✅ **Accessibility Out of the Box**
- Keyboard navigation
- Screen reader support
- ARIA attributes
- Focus management

✅ **Compound Components**
- Components with multiple sub-components
- Complex state management
- Coordinated behavior

### Use Tailwind Utilities When You Need:

✅ **Layout and Spacing**
- Grid, Flexbox, positioning
- Margins, padding, gaps
- Responsive layouts

✅ **Simple Styling**
- Colors, backgrounds
- Borders, shadows
- Typography (size, weight, color)

✅ **One-Off Components**
- Simple divs, sections
- Static content containers
- Basic wrappers

---

## 📦 Component Extension Patterns

### Pattern 1: Simple Wrapper (Most Common)

Use when you want to add default styles or behaviors.

```tsx
// components/awesome-button.tsx
import { Button, ButtonProps } from "@/components/ui/button"

export function AwesomeButton({ className, ...props }: ButtonProps) {
  return (
    <Button
      className={cn(
        "bg-gradient-to-r from-purple-500 to-pink-500",
        "hover:from-purple-600 hover:to-pink-600",
        "shadow-lg shadow-purple-500/50",
        className // Allow overrides
      )}
      {...props}
    />
  )
}
```

### Pattern 2: Extended Props

Use when you need custom behavior or props.

```tsx
// components/icon-button.tsx
import { Button, ButtonProps } from "@/components/ui/button"
import { type LucideIcon } from "lucide-react"

interface IconButtonProps extends ButtonProps {
  icon: LucideIcon
  iconPosition?: "left" | "right"
}

export function IconButton({
  icon: Icon,
  iconPosition = "left",
  children,
  className,
  ...props
}: IconButtonProps) {
  return (
    <Button className={cn("gap-2", className)} {...props}>
      {iconPosition === "left" && <Icon className="size-4" />}
      {children}
      {iconPosition === "right" && <Icon className="size-4" />}
    </Button>
  )
}
```

### Pattern 3: Variant-Specific Wrapper

Use when you always want a specific variant with custom styles.

```tsx
// components/danger-button.tsx
import { Button, ButtonProps } from "@/components/ui/button"

export function DangerButton({ className, ...props }: ButtonProps) {
  return (
    <Button
      variant="destructive"
      className={cn(
        "hover:scale-105 transition-transform",
        "shadow-md shadow-red-500/30",
        className
      )}
      {...props}
    />
  )
}
```

### Pattern 4: Compound Component Wrapper

Use when wrapping complex components like Dialog or Card.

```tsx
// components/confirmation-dialog.tsx
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog"
import { Button } from "@/components/ui/button"

interface ConfirmationDialogProps {
  open: boolean
  onOpenChange: (open: boolean) => void
  title: string
  description: string
  onConfirm: () => void
  onCancel: () => void
  confirmText?: string
  cancelText?: string
}

export function ConfirmationDialog({
  open,
  onOpenChange,
  title,
  description,
  onConfirm,
  onCancel,
  confirmText = "Confirm",
  cancelText = "Cancel",
}: ConfirmationDialogProps) {
  return (

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mark0025/npm-auth-gateway](https://github.com/Mark0025/npm-auth-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
