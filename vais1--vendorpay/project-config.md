---
trigger: always_on
description: shadcn/ui Component Rules
---


# shadcn/ui Components Reference

This project includes the following components in `@/components/ui` (shadcn/ui + custom extensions), all compatible with Vite + React:

## Core Components

- **Button** — Button with variants (default, outline, destructive, ghost, link, secondary, icon, size).
- **Card, CardHeader, CardTitle, CardDescription, CardContent, CardFooter** — Card and its sections for content wrapping.
- **Badge** — Small label for statuses, tags, etc.
- **Banner** — Banner for notifications, errors, info.
- **Avatar, AvatarImage, AvatarFallback** — User avatar with fallback.
- **Skeleton** — Skeleton loader.
- **Separator** — Horizontal or vertical divider.
- **Label** — Form label.
- **Input** — Text input field.
- **Textarea** — Multiline text input.
- **Switch** — Toggle switch.
- **Checkbox** — Checkbox input.
- **RadioGroup, RadioGroupItem** — Radio button group.
- **Select, SelectItem, ...** — Dropdown select.
- **DropdownMenu, ...** — Context menu.
- **ContextMenu, ...** — Right-click context menu.
- **Popover, PopoverContent, ...** — Popover window.
- **Tooltip, TooltipContent, ...** — Tooltip (hover hint).
- **Tabs, TabsList, TabsTrigger, TabsContent** — Tab navigation.
- **Accordion, AccordionItem, AccordionTrigger, AccordionContent** — Accordion sections.
- **Dialog, DialogContent, ...** — Modal dialog.
- **AlertDialog, ...** — Confirmation modal dialog.
- **Sheet, SheetContent, ...** — Sliding side panel.
- **Drawer, DrawerContent, ...** — Drawer for mobile/desktop.
- **Toast, ToastProvider, ...** — Toast notifications.
- **Sonner, Toaster** — Alternative toast (sonner).
- **Progress** — Progress bar.
- **Table, TableHeader, TableBody, ...** — Table.
- **Pagination, PaginationItem, ...** — Pagination controls.
- **Timeline, TimelineItem, ...** — Timeline/steps.
- **HeroPill** — Decorative element for hero sections.
- **Mockup, MockupFrame** — Wrapper for mockups/demos.
- **CircleProgress** — Circular progress indicator.
- **Chart, ChartContainer, ...** — Charts (based on recharts).
- **Multiselect** — Multi-select with search (cmdk).
- **Command, CommandDialog, ...** — Command palette (cmdk).
- **ResizablePanel, ResizablePanelGroup, ResizableHandle** — Resizable panels.
- **Sidebar, SidebarProvider, ...** — Sidebar with groups, menus, triggers.
- **ThemeToggle** — Light/dark theme toggle.
- **Tag** — Tag (uses plain `<a>` for links).

## Descriptions and Usage Examples

### Button

A button with variants and sizes.

```tsx
import { Button } from "@/components/ui/button"
<Button>Click me</Button>
<Button variant="outline">Outline</Button>
<Button size="icon"><Rocket /></Button>
```

### Card

A card for wrapping content.

```tsx
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card";
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent>Content</CardContent>
</Card>;
```

### Badge

Small label for statuses, tags, etc.

```tsx
import { Badge } from "@/components/ui/badge";
<Badge>New</Badge>;
```

### Banner

Banner for notifications, errors, info.

```tsx
import { Banner } from "@/components/ui/banner";
<Banner variant="muted">Info message</Banner>;
```

### Avatar

User avatar with fallback.

```tsx
import { Avatar, AvatarImage, AvatarFallback } from "@/components/ui/avatar";
<Avatar>
  <AvatarImage src="/user.png" />
  <AvatarFallback>AB</AvatarFallback>
</Avatar>;
```

### Skeleton

Skeleton loader.

```tsx
import { Skeleton } from "@/components/ui/skeleton";
<Skeleton className="h-8 w-32" />;
```

### Separator

Horizontal or vertical divider.

```tsx
import { Separator } from "@/components/ui/separator";
<Separator />;
```

### Label

Form label.

```tsx
import { Label } from "@/components/ui/label";
<Label htmlFor="email">Email</Label>;
```

### Input

Text input field.

```tsx
import { Input } from "@/components/ui/input";
<Input placeholder="Type here..." />;
```

### Textarea

Multiline text input.

```tsx
import { Textarea } from "@/components/ui/textarea";
<Textarea placeholder="Type here..." />;
```

### Switch

Toggle switch.

```tsx
import { Switch } from "@/components/ui/switch";
<Switch />;
```

### Checkbox

Checkbox input.

```tsx
import { Checkbox } from "@/components/ui/checkbox";
<Checkbox />;
```

### RadioGroup

Radio button group.

```tsx
import { RadioGroup, RadioGroupItem } from "@/components/ui/radio-group";
<RadioGroup defaultValue="1">
  <RadioGroupItem value="1" />
  <RadioGroupItem value="2" />
</RadioGroup>;
```

### Select

Dropdown select.

```tsx
import {
  Select,
  SelectItem,
  SelectTrigger,
  SelectContent,
} from "@/components/ui/select";
<Select>
  <SelectTrigger />
  <SelectContent>
    <SelectItem value="1">One</SelectItem>
    <SelectItem value="2">Two</SelectItem>
  </SelectContent>
</Select>;
```

### DropdownMenu

Context menu.

```tsx
import {
  DropdownMenu,
  DropdownMenuTrigger,
  DropdownMenuContent,
  DropdownMenuItem,
} from "@/components/ui/dropdown-menu";
<DropdownMenu>
  <DropdownMenuTrigger>Open</DropdownMenuTrigger>
  <DropdownMenuContent>
    <DropdownMenuItem>Item 1</DropdownMenuItem>
  </DropdownMenuContent>
</DropdownMenu>;
```

### Popover

Popover window.

```tsx
import {
  Popover,
  PopoverTrigger,
  PopoverContent,
} from "@/components/ui/popover";
<Popover>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Vais1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
