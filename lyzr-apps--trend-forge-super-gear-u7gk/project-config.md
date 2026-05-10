---
trigger: always_on
description: For FRESH BUILDS: Do NOT list directories or browse the file structure. Read ONLY `response_schemas/*.json` and `workflow.json` — then immediately start writing `app/page.tsx`. No exploring, no reading other files.
---

# Next.js React Frontend

## DO NOT EXPLORE — START BUILDING IMMEDIATELY

For FRESH BUILDS: Do NOT list directories or browse the file structure. Read ONLY `response_schemas/*.json` and `workflow.json` — then immediately start writing `app/page.tsx`. No exploring, no reading other files.

For ITERATIONS: Read `app/page.tsx` first to understand current state, then `response_schemas/*.json` for any updated response shapes.

---

## Import Rules

**Icons:** `lucide-react` ONLY (never react-icons)
```tsx
import { Loader2, Send, X } from 'lucide-react'
```

**Components:** `@/components/ui/*` (shadcn only)
```tsx
import { Button } from '@/components/ui/button'
import { Card, CardHeader, CardContent } from '@/components/ui/card'
import { Input } from '@/components/ui/input'
```

**Agent Calls:** `@/lib/aiAgent` (client-side, calls pre-built `/api/agent` route — NEVER create new routes)
```tsx
import { callAIAgent } from '@/lib/aiAgent'
// callAIAgent(message, agent_id) — ONLY way to call agents. NEVER custom fetch.
```

---

## COMPONENT WHITELIST (CLOSED SET — nothing else exists)

**ONLY these components may be used in `app/page.tsx`.** If a name is not listed here, it does NOT exist. Define custom components inline as functions in page.tsx.

### shadcn/ui Components (import from `@/components/ui/<file>`)

| File | Exports |
|------|---------|
| accordion | `Accordion`, `AccordionItem`, `AccordionTrigger`, `AccordionContent` |
| alert | `Alert`, `AlertTitle`, `AlertDescription` |
| alert-dialog | `AlertDialog`, `AlertDialogTrigger`, `AlertDialogContent`, `AlertDialogHeader`, `AlertDialogFooter`, `AlertDialogTitle`, `AlertDialogDescription`, `AlertDialogAction`, `AlertDialogCancel` |
| aspect-ratio | `AspectRatio` |
| avatar | `Avatar`, `AvatarImage`, `AvatarFallback` |
| badge | `Badge` |
| breadcrumb | `Breadcrumb`, `BreadcrumbList`, `BreadcrumbItem`, `BreadcrumbLink`, `BreadcrumbPage`, `BreadcrumbSeparator` |
| button | `Button` |
| calendar | `Calendar` |
| card | `Card`, `CardHeader`, `CardFooter`, `CardTitle`, `CardDescription`, `CardContent` |
| carousel | `Carousel`, `CarouselContent`, `CarouselItem`, `CarouselPrevious`, `CarouselNext` |
| chart | `ChartContainer`, `ChartTooltip`, `ChartTooltipContent`, `ChartLegend`, `ChartLegendContent` |
| checkbox | `Checkbox` |
| collapsible | `Collapsible`, `CollapsibleTrigger`, `CollapsibleContent` |
| command | `Command`, `CommandDialog`, `CommandInput`, `CommandList`, `CommandEmpty`, `CommandGroup`, `CommandItem`, `CommandShortcut`, `CommandSeparator` |
| context-menu | `ContextMenu`, `ContextMenuTrigger`, `ContextMenuContent`, `ContextMenuItem`, `ContextMenuLabel`, `ContextMenuSeparator` |
| dialog | `Dialog`, `DialogTrigger`, `DialogContent`, `DialogHeader`, `DialogFooter`, `DialogTitle`, `DialogDescription`, `DialogClose` |
| drawer | `Drawer`, `DrawerTrigger`, `DrawerContent`, `DrawerHeader`, `DrawerFooter`, `DrawerTitle`, `DrawerDescription`, `DrawerClose` |
| dropdown-menu | `DropdownMenu`, `DropdownMenuTrigger`, `DropdownMenuContent`, `DropdownMenuItem`, `DropdownMenuLabel`, `DropdownMenuSeparator`, `DropdownMenuGroup` |
| empty | `Empty`, `EmptyHeader`, `EmptyTitle`, `EmptyDescription`, `EmptyContent`, `EmptyMedia` |
| form | `Form`, `FormItem`, `FormLabel`, `FormControl`, `FormDescription`, `FormMessage`, `FormField` |
| hover-card | `HoverCard`, `HoverCardTrigger`, `HoverCardContent` |
| input | `Input` |
| input-group | `InputGroup`, `InputGroupAddon`, `InputGroupButton`, `InputGroupText` |
| input-otp | `InputOTP`, `InputOTPGroup`, `InputOTPSlot`, `InputOTPSeparator` |
| label | `Label` |
| menubar | `Menubar`, `MenubarMenu`, `MenubarTrigger`, `MenubarContent`, `MenubarItem`, `MenubarSeparator` |
| navigation-menu | `NavigationMenu`, `NavigationMenuList`, `NavigationMenuItem`, `NavigationMenuContent`, `NavigationMenuTrigger`, `NavigationMenuLink` |
| pagination | `Pagination`, `PaginationContent`, `PaginationItem`, `PaginationLink`, `PaginationNext`, `PaginationPrevious` |
| popover | `Popover`, `PopoverTrigger`, `PopoverContent` |
| progress | `Progress` |
| radio-group | `RadioGroup`, `RadioGroupItem` |
| resizable | `ResizablePanelGroup`, `ResizablePanel`, `ResizableHandle` |
| scroll-area | `ScrollArea`, `ScrollBar` |
| select | `Select`, `SelectTrigger`, `SelectContent`, `SelectItem`, `SelectValue`, `SelectGroup`, `SelectLabel` |
| separator | `Separator` |
| sheet | `Sheet`, `SheetTrigger`, `SheetContent`, `SheetHeader`, `SheetFooter`, `SheetTitle`, `SheetDescription`, `SheetClose` |
| sidebar | `Sidebar`, `SidebarContent`, `SidebarFooter`, `SidebarGroup`, `SidebarGroupContent`, `SidebarGroupLabel`, `SidebarHeader`, `SidebarMenu`, `SidebarMenuButton`, `SidebarMenuItem`, `SidebarProvider`, `SidebarTrigger` |
| skeleton | `Skeleton` |
| slider | `Slider` |
| sonner | `Toaster` |
| spinner | `Spinner` |
| switch | `Switch` |
| table | `Table`, `TableHeader`, `TableBody`, `TableFooter`, `TableHead`, `TableRow`, `TableCell`, `TableCaption` |
| tabs | `Tabs`, `TabsList`, `TabsTrigger`, `TabsContent` |
| textarea | `Textarea` |
| toggle | `Toggle` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lyzr-Apps/trend-forge-super-gear-u7gk](https://github.com/Lyzr-Apps/trend-forge-super-gear-u7gk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
