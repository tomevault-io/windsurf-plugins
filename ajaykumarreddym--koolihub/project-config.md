---
trigger: always_on
description: Figma Design Integration - Design-to-Code Guidelines
---


# Figma Design Integration Rules

## Overview
Guidelines for integrating Figma designs into KooliHub codebase using the Figma MCP tools and maintaining design-code consistency.

---

## 1. Figma MCP Tool Usage

### 1.1 Available Tools
```typescript
// Primary tools for design extraction
mcp_Figma_get_design_context   // Generate UI code from Figma node
mcp_Figma_get_screenshot       // Visual reference screenshot
mcp_Figma_get_metadata         // Structure overview (XML format)
mcp_Figma_get_variable_defs    // Design tokens extraction
mcp_Figma_get_code_connect_map // Component mapping
```

### 1.2 Extracting Design Context
```typescript
// When given a Figma URL like:
// https://figma.com/design/ABC123/MyProject?node-id=1-2

// Extract:
fileKey: 'ABC123'
nodeId: '1:2' (or '1-2')

// Call with:
get_design_context({
  fileKey: 'ABC123',
  nodeId: '1:2',
  clientFrameworks: 'react',
  clientLanguages: 'typescript,html,css'
})
```

### 1.3 Design Token Extraction
```typescript
// Get variable definitions
get_variable_defs({
  fileKey: 'ABC123',
  nodeId: '1:2'
})

// Response format:
{
  'color/primary': '#FFD700',
  'color/secondary': '#6B7280',
  'spacing/sm': '8px',
  'radius/md': '12px'
}
```

---

## 2. Design-to-Code Workflow

### 2.1 Standard Workflow
```
1. Get Figma URL from designer
2. Extract nodeId and fileKey from URL
3. Call get_design_context for code generation
4. Call get_variable_defs for design tokens
5. Map Figma tokens to existing design system
6. Implement using existing UI components
7. Validate against screenshot
```

### 2.2 Component Mapping Priority
```
Figma Component → KooliHub Equivalent
─────────────────────────────────────
Button          → @/components/ui/button
Card            → @/components/ui/card
Input           → @/components/ui/input
Select          → @/components/ui/select
Modal           → @/components/ui/dialog
Dropdown        → @/components/ui/dropdown-menu
Toast           → @/components/ui/toast
Badge           → @/components/ui/badge
Avatar          → @/components/ui/avatar
Tabs            → @/components/ui/tabs
```

---

## 3. Design Token Mapping

### 3.1 Color Mapping
```typescript
// Figma Variable → CSS Variable → Tailwind Class

// Map Figma colors to existing tokens
const figmaToTokenMap = {
  // Figma naming → KooliHub token
  'primary': '--primary',
  'primary-foreground': '--primary-foreground',
  'secondary': '--secondary',
  'background': '--background',
  'foreground': '--foreground',
  'muted': '--muted',
  'accent': '--accent',
  'destructive': '--destructive',
  'border': '--border',
  
  // Service-specific
  'grocery': 'grocery-500',
  'trips': 'trips-500',
  'handyman': 'handyman-500',
};
```

### 3.2 Spacing Mapping
```typescript
const spacingMap = {
  // Figma → Tailwind
  '4': 'p-1',    // 4px
  '8': 'p-2',    // 8px
  '12': 'p-3',   // 12px
  '16': 'p-4',   // 16px
  '20': 'p-5',   // 20px
  '24': 'p-6',   // 24px
  '32': 'p-8',   // 32px
  '40': 'p-10',  // 40px
  '48': 'p-12',  // 48px
  '64': 'p-16',  // 64px
};
```

### 3.3 Typography Mapping
```typescript
const typographyMap = {
  // Figma text style → Tailwind classes
  'Heading/H1': 'text-4xl font-bold',
  'Heading/H2': 'text-3xl font-bold',
  'Heading/H3': 'text-2xl font-semibold',
  'Heading/H4': 'text-xl font-semibold',
  'Body/Large': 'text-lg',
  'Body/Regular': 'text-base',
  'Body/Small': 'text-sm',
  'Caption': 'text-xs text-muted-foreground',
  'Label': 'text-sm font-medium',
};
```

---

## 4. Implementation Rules

### 4.1 Never Hardcode Design Values
```typescript
// ❌ BAD - Hardcoded values from Figma
<div style={{ padding: '16px', backgroundColor: '#FFD700' }}>

// ✅ GOOD - Use design tokens
<div className="p-4 bg-primary">

// ❌ BAD - Inline pixel values
<div className="text-[14px] leading-[20px]">

// ✅ GOOD - Use semantic classes
<div className="text-sm leading-normal">
```

### 4.2 Color Extraction Rules
```typescript
// When Figma provides hex colors, map to tokens:

// Primary brand colors → use CSS variables
'#FFD700' → 'bg-primary' (update --primary if needed)
'#10B981' → 'bg-grocery-500'
'#3B82F6' → 'bg-trips-500'
'#F97316' → 'bg-handyman-500'

// Grayscale → use standard palette
'#F9FAFB' → 'bg-gray-50'
'#F3F4F6' → 'bg-gray-100'
'#E5E7EB' → 'bg-gray-200'
'#6B7280' → 'text-gray-500'
'#374151' → 'text-gray-700'
'#111827' → 'text-gray-900'

// Add new tokens to global.css if not exists
```

### 4.3 Spacing Extraction Rules
```typescript
// Convert Figma auto-layout to Tailwind

// Gap → gap-{n}
gap: 8   → gap-2
gap: 16  → gap-4
gap: 24  → gap-6

// Padding → p-{n}, px-{n}, py-{n}
padding: 16 → p-4
paddingX: 24, paddingY: 16 → px-6 py-4

// Margin → m-{n}, mx-{n}, my-{n}
marginTop: 32 → mt-8
```

---

## 5. Component Generation Rules

### 5.1 From Figma Auto-Layout
```typescript
// Figma Auto Layout → Tailwind Flex/Grid

// Horizontal layout
direction: horizontal → flex flex-row
spacing: 16 → gap-4

// Vertical layout
direction: vertical → flex flex-col
spacing: 8 → gap-2

// Wrap
wrap: true → flex-wrap

// Alignment
primaryAxisAlignItems: center → justify-center
counterAxisAlignItems: center → items-center
primaryAxisAlignItems: spaceBetween → justify-between
```

### 5.2 Component Structure Template
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ajaykumarreddym) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
