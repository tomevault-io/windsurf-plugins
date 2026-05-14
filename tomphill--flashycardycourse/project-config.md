---
trigger: always_on
description: This project MUST use ONLY shadcn/ui components for ALL UI elements. ABSOLUTELY NO custom UI components are allowed.
---

# SHADCN/UI STRICT USAGE RULES

## 🚨 CRITICAL MANDATE
This project MUST use ONLY shadcn/ui components for ALL UI elements. ABSOLUTELY NO custom UI components are allowed.

**MANDATORY Rules:**
- ❌ NEVER create custom buttons, inputs, cards, modals, or any UI elements
- ❌ NEVER use raw HTML elements (div, button, input) for UI
- ❌ NEVER write custom CSS for components beyond Tailwind utilities
- ✅ ALWAYS use shadcn/ui components for every UI element
- ✅ ALWAYS compose complex UI by combining shadcn/ui components
- ✅ This project is in dark mode so make sure there are no dark colored text on dark backgrounds (and vice versa)

## 🛡️ ENFORCEMENT POLICY
**NO EXCEPTIONS** - Every UI element must use shadcn/ui components. If a component doesn't exist in shadcn/ui, you must:
1. First check if it can be composed from existing components
2. If not, add the closest shadcn/ui component and adapt
3. NEVER create custom components

## 📋 BEFORE CODING CHECKLIST
**Before creating any UI:** Run `npx shadcn@latest add [component-name]`

Required components for this project:
```bash
npx shadcn@latest add button
npx shadcn@latest add card  
npx shadcn@latest add input
npx shadcn@latest add dialog
npx shadcn@latest add form
npx shadcn@latest add badge
npx shadcn@latest add separator
npx shadcn@latest add avatar
npx shadcn@latest add progress
npx shadcn@latest add toast
npx shadcn@latest add alert
npx shadcn@latest add tabs
npx shadcn@latest add select
npx shadcn@latest add textarea
npx shadcn@latest add label
```

## 🔐 CLERK INTEGRATION REQUIREMENTS

### Authentication Buttons
All Clerk sign-in/sign-up buttons MUST use shadcn/ui Button components:

```tsx
import { SignInButton, SignUpButton } from "@clerk/nextjs";
import { Button } from "@/components/ui/button";

// ✅ CORRECT - Using shadcn/ui Button
<SignInButton mode="modal">
  <Button variant="default">Sign In</Button>
</SignInButton>

<SignUpButton mode="modal">
  <Button variant="outline">Sign Up</Button>
</SignUpButton>

// ❌ WRONG - Custom button or raw HTML
<SignInButton>
  <button className="custom-btn">Sign In</button>
</SignInButton>
```

### Modal Requirements
Clerk MUST use modal mode with shadcn/ui Dialog components:

```tsx
import { SignInButton } from "@clerk/nextjs";
import { Dialog, DialogTrigger } from "@/components/ui/dialog";
import { Button } from "@/components/ui/button";

// ✅ CORRECT - Modal mode with shadcn/ui components
<SignInButton mode="modal">
  <Button>Access Dashboard</Button>
</SignInButton>
```

### User Profile Integration
```tsx
import { UserButton } from "@clerk/nextjs";
import { Avatar, AvatarFallback, AvatarImage } from "@/components/ui/avatar";

// ✅ CORRECT - If customizing UserButton appearance
<UserButton 
  appearance={{
    elements: {
      avatarBox: "w-10 h-10", // Use Tailwind classes only
    }
  }}
/>
```

## 🚫 ABSOLUTELY FORBIDDEN

### Never Create These:
- Custom Button components
- Custom Input/Form components  
- Custom Card components
- Custom Modal/Dialog components
- Custom Navigation components
- Custom Loading spinners
- Custom Icons (use Lucide React with shadcn/ui)
- Custom Tooltips or Popovers

### Never Use These:
```tsx
// ❌ WRONG - Raw HTML elements
<button>Click me</button>
<input type="text" />
<div className="card">Content</div>
<form>...</form>

// ❌ WRONG - Custom CSS classes for components
<div className="custom-button">Button</div>
<div className="my-modal">Modal content</div>

// ❌ WRONG - Third-party UI libraries
import { Button } from 'some-other-ui-lib';
```

## ✅ REQUIRED PATTERNS

### Button Usage
```tsx
import { Button } from "@/components/ui/button";

// All button variants available
<Button variant="default">Primary</Button>
<Button variant="destructive">Delete</Button>
<Button variant="outline">Secondary</Button>
<Button variant="secondary">Alt</Button>
<Button variant="ghost">Subtle</Button>
<Button variant="link">Link Style</Button>
```

### Form Components
```tsx
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";

<Card>
  <CardHeader>
    <CardTitle>Form Title</CardTitle>
  </CardHeader>
  <CardContent>
    <form className="space-y-4">
      <div>
        <Label htmlFor="email">Email</Label>
        <Input id="email" type="email" />
      </div>
      <Button type="submit">Submit</Button>
    </form>
  </CardContent>
</Card>
```

### Layout Components
```tsx
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Separator } from "@/components/ui/separator";
import { Badge } from "@/components/ui/badge";

<Card>
  <CardHeader>
    <CardTitle>Flashcard Deck</CardTitle>
    <Badge variant="secondary">25 cards</Badge>
  </CardHeader>
  <Separator />
  <CardContent>
    Content here
  </CardContent>
</Card>
```

## 🔧 COMPONENT INSTALLATION

### Immediate Installation Required
Run these commands now if components are missing:

```bash
# Core components for flashcard app
npx shadcn@latest add button
npx shadcn@latest add card
npx shadcn@latest add input
npx shadcn@latest add label
npx shadcn@latest add dialog

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomphill/flashycardycourse](https://github.com/tomphill/flashycardycourse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
