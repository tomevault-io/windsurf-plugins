---
trigger: always_on
description: Media gallery grid styling pattern with medium dark gradient backgrounds for admin pages and conservative gradients for public pages
---


# Media Gallery Grid Styling Pattern

## **Overview**
This rule defines the standard pattern for displaying media/image grids with gradient backgrounds, matching the design aesthetic of both admin pages (medium dark gradients) and public-facing gallery pages (conservative warm tones). The pattern provides consistent styling, subtle visual depth, and proper icon presentation.

## **Problem Solved**
- **Consistent Grid Styling**: Ensures all media grids use the same visual pattern across admin and public pages
- **Medium Dark Gradient Backgrounds**: Provides professional medium dark gradient backgrounds for admin pages that enhance without overwhelming
- **Conservative Gradient Backgrounds**: Provides subtle, professional gradient backgrounds for public pages
- **Icon Standardization**: Provides consistent icon container and sizing for action buttons (references Icon Standards and Admin Action Buttons rules)
- **Visual Hierarchy**: Creates clear separation between grid container and individual media items
- **Responsive Design**: Works consistently across different screen sizes

## **Core Pattern**

### **Grid Container Structure**
```tsx
// ✅ DO: Use the standard media grid container pattern
// Admin Pages (Medium Dark Gradient)
<div className="relative overflow-hidden rounded-3xl bg-gradient-to-br from-gray-700 via-gray-800 to-gray-700 border border-gray-600/30 shadow-2xl mb-8">
  {/* Medium Dark Radial Gradient Overlay */}
  <div className="absolute inset-0 pointer-events-none opacity-60" style={{ backgroundImage: 'radial-gradient(circle at top left, rgba(255, 255, 255, 0.12), transparent 55%)' }} />

  {/* Grid Content */}
  <div className="relative px-6 py-10 sm:px-10 lg:px-14">
    <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
      {/* Media items */}
    </div>
  </div>
</div>
```

## **Key CSS Properties**

### **Container Requirements**
- **`relative`**: Enables absolute positioning of overlay
- **`overflow-hidden`**: Clips content to rounded corners
- **`rounded-3xl`**: Large border radius (24px) for modern appearance
- **`bg-gradient-to-br`**: Diagonal gradient from top-left to bottom-right
- **Admin Pages**: `from-gray-700 via-gray-800 to-gray-700` (medium dark gradient)
- **Public Pages**: `from-background via-muted to-background` (conservative warm tones)
- **Event Pages**: `from-gray-900 via-purple-900 to-indigo-900` (bold dark gradient)
- **Border**:
  - Admin: `border-gray-600/30` (medium dark border)
  - Public: `border-border/30` (subtle border)
  - Event: `border-white/10` (light border on dark)
- **Shadow**:
  - Admin: `shadow-2xl` (large shadow for depth)
  - Public: `sacred-shadow-lg` (MOSC shadow)
  - Event: `shadow-2xl` (large shadow)
- **`mb-8`**: Bottom margin (32px) for spacing

### **Radial Gradient Overlay Requirements**
- **`absolute inset-0`**: Covers entire container
- **`pointer-events-none`**: Doesn't interfere with interactions
- **Opacity**:
  - **Admin Pages (Medium Dark)**: `opacity-60` (60% for medium dark backgrounds)
  - **Public Pages (Conservative)**: `opacity-30` (30% for subtlety)
  - **Event Pages (Bold Dark)**: `opacity-70` (70% for bold dark backgrounds)
- **Radial Gradient**:
  - **Admin Pages**: `radial-gradient(circle at top left, rgba(255, 255, 255, 0.12), transparent 55%)` (white at 12% opacity)
  - **Public Pages**: `radial-gradient(circle at top left, rgba(139, 125, 107, 0.08), transparent 55%)` (primary color at 8% opacity)
  - **Event Pages**: `radial-gradient(circle at top left, rgba(255,255,255,0.18), transparent 55%)` (white at 18% opacity)
  - Starts at top-left corner
  - Fades to transparent at 55% radius

### **Grid Content Requirements**
- **`relative`**: Positions content above overlay
- **`px-6 py-10 sm:px-10 lg:px-14`**: Responsive padding
  - Mobile: 24px horizontal, 40px vertical
  - Small screens: 40px horizontal
  - Large screens: 56px horizontal

### **Grid Requirements**
- **`grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4`**: Responsive grid
  - Mobile: 1 column
  - Small screens (640px+): 2 columns
  - Medium screens (768px+): 3 columns
  - Large screens (1024px+): 4 columns
- **`gap-6`**: Consistent gap between items (24px)

## **Color Coding System**

### **Admin Pages** (Medium Dark Gradient)
- **Background**: `from-gray-700 via-gray-800 to-gray-700`
- **Border**: `border-gray-600/30`
- **Radial Gradient**: `rgba(255, 255, 255, 0.12)` (white at 12% opacity)
- **Overlay Opacity**: `opacity-60` (60% for medium dark backgrounds)

### **Public Pages** (MOSC Warm Earth Tones)
- **Background**: `from-background via-muted to-background`
- **Border**: `border-border/30`
- **Radial Gradient**: `rgba(139, 125, 107, 0.08)` (primary color at 8% opacity)

### **Event Pages** (Bold Dark Gradient)
- **Background**: `from-gray-900 via-purple-900 to-indigo-900`
- **Border**: `border-white/10` (light border on dark background)
- **Radial Gradient**: `rgba(255,255,255,0.18)` (white at 18% opacity)
- **Overlay Opacity**: `opacity-70` (70% for bold dark backgrounds)
- **Inner Thumbnail Container**: `bg-white/10 backdrop-blur-sm rounded-2xl border border-white/20 p-6 shadow-inner` (glassmorphism effect)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
