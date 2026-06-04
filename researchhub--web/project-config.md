---
trigger: always_on
description: When applying buttons or styles that depend on screen size, or mobile, or having issues with staging compared to local
---

# Mobile Optimization Strategy

This document outlines the mobile optimization strategy developed for ResearchHub landing page components to address environment-specific responsive layout issues.

## Problem Context

### Root Issue Identified
- **Problem**: Responsive Tailwind classes (`sm:`, `md:`, etc.) weren't compiling properly on staging despite JavaScript correctly detecting screen sizes
- **Symptoms**: 
  - Buttons stacked vertically and went full-width on staging
  - Same code displayed correctly on localhost
  - Debug tools showed correct screen size detection but wrong visual rendering
- **Root Cause**: Environment-specific CSS compilation/loading issue affecting responsive class generation

## Core Solution Strategy

Replace responsive breakpoints with environment-agnostic approaches that work consistently across all deployment environments.

### 1. Button Standardization
```tsx
// ❌ Avoid: Custom responsive button implementations
<button className="px-6 py-3 sm:px-8 sm:py-4 md:w-auto w-full">

// ✅ Preferred: Use existing Button component with cva variants
<Button size="lg" className="bg-gradient-to-r from-primary-600 to-primary-400">
```

**Benefits:**
- Leverages class-variance-authority (cva) for consistent styling
- Eliminates dependency on responsive class compilation
- Ensures uniform behavior across environments

### 2. Natural Flex-Wrapping Over Breakpoints
```tsx
// ❌ Avoid: Responsive grid systems
<div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3">

// ✅ Preferred: Natural flex-wrapping with constraints
<div className="flex flex-wrap gap-4 justify-center">
  <div className="flex-1 min-w-64 max-w-80">
```

**Benefits:**
- Uses intrinsic sizing for natural responsive behavior
- Eliminates breakpoint dependencies
- Provides predictable layout across screen sizes

### 3. Fixed Typography Over Responsive Text
```tsx
// ❌ Avoid: Responsive text sizing
<h2 className="text-3xl sm:text-4xl md:text-5xl">

// ✅ Preferred: Fixed sizes appropriate for content hierarchy
<h2 className="text-4xl font-bold">
```

**Benefits:**
- Ensures consistent typography rendering
- Removes compilation dependencies
- Maintains readability without environment risks

## Implementation Patterns

### Dual Button Layout (Fund Feature Pattern)
```tsx
// For features with primary + secondary actions
<div className="flex flex-wrap gap-4 justify-center items-start max-w-2xl mx-auto">
  <div className="flex-1 min-w-64 max-w-80 text-center">
    <Button size="lg" className="w-full bg-gradient-to-r from-primary-600 to-primary-400">
      Primary Action
    </Button>
    <p className="text-sm text-gray-500 mt-2">Description</p>
  </div>
  <div className="flex-1 min-w-64 max-w-80 text-center">
    <Button variant="outlined" size="lg" className="w-full">
      Secondary Action
    </Button>
    <p className="text-sm text-gray-500 mt-2">Description</p>
  </div>
</div>
```

### Single Button Layout
```tsx
// For features with single action
<div className="text-center">
  <Button size="lg" className="bg-gradient-to-r from-primary-600 to-primary-400">
    Single Action
  </Button>
  <p className="text-sm text-gray-500 mt-2">Description</p>
</div>
```

### Benefits Grid Layout
```tsx
// Replace responsive grids with flex-wrap
<div className="flex flex-wrap gap-4 max-w-xl mx-auto px-4 justify-center">
  {benefits.map((benefit, index) => (
    <div key={index} className="flex items-center space-x-3 text-left min-w-60">
      <div className="w-2 h-2 rounded-full bg-gradient-to-r from-primary-600 to-primary-400 flex-shrink-0" />
      <span className="text-gray-700 text-base">{benefit}</span>
    </div>
  ))}
</div>
```

### Navigation Tabs
```tsx
// Environment-agnostic tab navigation
<div className="inline-flex p-1 bg-gray-100 rounded-full overflow-x-auto">
  {features.map((feature, index) => (
    <button
      key={feature.id}
      className={`px-6 py-3 rounded-full font-medium transition-all duration-300 text-base whitespace-nowrap flex-shrink-0 ${
        activeFeature === index
          ? 'bg-gradient-to-r from-primary-600 to-primary-400 text-white shadow-lg'
          : 'text-gray-600 hover:text-gray-900 hover:bg-white/50'
      }`}
    >
      {feature.title}
    </button>
  ))}
</div>
```

## Mobile-Specific Optimizations

### Icon Layout Prevention
```tsx
// ❌ Avoid: Horizontal layouts that can distort icons on mobile
<div className="flex items-center space-x-4">
  <Icon name="icon" size={64} />
  <div>Content</div>
</div>

// ✅ Preferred: Vertical layouts for mobile-friendly icons
<div className="flex flex-col items-center space-y-4">
  <div className="w-16 h-16 flex-shrink-0">
    <Icon name="icon" size={32} />
  </div>
  <div>Content</div>
</div>
```

### Compact Data Display
```tsx
// For price/data displays that need to fit on mobile
<div className="p-4 flex flex-wrap items-start justify-center gap-6">
  <div className="text-center">
    <div className="text-sm text-gray-500 mb-2 font-medium h-5 flex items-center justify-center">
      Label
    </div>
    <div className="text-2xl font-bold">$0.33</div>
  </div>
  
  <div className="h-12 w-px bg-gray-300 self-center"></div>
  
  <div className="text-center">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ResearchHub/web](https://github.com/ResearchHub/web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
