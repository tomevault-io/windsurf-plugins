---
trigger: always_on
description: MOSC Redesign Project - Comprehensive Styling Standards and Guidelines
---


# MOSC Redesign Project - Styling Standards

## **Design System Overview**
This project uses a **sacred, reverent design system** inspired by Orthodox Christian traditions with warm earth tones, elegant typography, and thoughtful spacing. All new pages and components must follow these established patterns.

## **Color Palette & CSS Variables**

### **Core System Colors**
- **Background**: `#F5F1E8` (soft cream) - `bg-background`
- **Foreground**: `#2D2A26` (near-black with warm undertones) - `text-foreground`
- **Border**: `rgba(139, 125, 107, 0.2)` (warm earth tone with transparency) - `border-border`
- **Input**: `#FFFFFF` (pure white) - `bg-input`
- **Ring**: `#8B7D6B` (warm earth tone) - `ring-ring`

### **Semantic Colors**
- **Primary**: `#8B7D6B` (warm earth tone) - `bg-primary text-primary-foreground`
- **Secondary**: `#A0926B` (lighter complement) - `bg-secondary text-secondary-foreground`
- **Accent**: `#6B4E3D` (rich brown) - `bg-accent text-accent-foreground`
- **Success**: `#4A6741` (muted sage green) - `bg-success text-success-foreground`
- **Warning**: `#A67C52` (warm amber) - `bg-warning text-warning-foreground`
- **Error/Destructive**: `#8B4A42` (subdued terracotta) - `bg-destructive text-destructive-foreground`
- **Muted**: `#EDE7D3` (lighter complement) - `bg-muted text-muted-foreground`

### **Card & Surface Colors**
- **Card**: `#FFFFFF` (pure white) - `bg-card text-card-foreground`
- **Popover**: `#FFFFFF` (pure white) - `bg-popover text-popover-foreground`

## **Typography System**

### **Font Families**
- **Headings**: `font-heading` (Crimson Text, serif) - For titles, headings, and important text
- **Body**: `font-body` (Source Sans Pro, sans-serif) - For paragraphs and general content
- **Caption**: `font-caption` (Lato, sans-serif) - For small text, labels, and captions
- **Monospace**: `font-mono` (JetBrains Mono, monospace) - For code and technical content

### **Font Usage Patterns**
```jsx
// ✅ DO: Use appropriate font families
<h1 className="font-heading font-semibold text-3xl text-foreground">Main Title</h1>
<p className="font-body text-lg text-muted-foreground">Body text content</p>
<small className="font-caption text-xs text-muted-foreground">Caption text</small>

// ❌ DON'T: Mix font families inappropriately
<h1 className="font-body font-semibold text-3xl">Main Title</h1> // Wrong font for heading
```

## **Spacing & Layout Standards**

### **Container Patterns**
- **Max Width**: `max-w-7xl mx-auto` for main content containers
- **Padding**: `px-4 sm:px-6 lg:px-8` for responsive horizontal padding
- **Sacred Spacing**: `space-sacred` (2rem) for consistent vertical spacing

### **Grid Systems**
- **Cards Grid**: `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-8`
- **Feature Grid**: `grid grid-cols-1 md:grid-cols-3 gap-8 lg:gap-12`
- **Stats Grid**: `grid grid-cols-2 gap-4` for statistics

### **Section Spacing**
- **Large Sections**: `py-16` for major content sections
- **Medium Sections**: `py-12` for secondary sections
- **Small Sections**: `py-8` for compact sections

## **Component Styling Patterns**

### **Cards & Panels**
```jsx
// ✅ DO: Use consistent card styling
<div className="bg-card rounded-lg sacred-shadow p-6">
  <div className="flex items-center space-x-3 mb-6">
    <div className="w-8 h-8 bg-primary rounded-full flex items-center justify-center">
      <Icon name="IconName" size={16} color="white" />
    </div>
    <h3 className="text-lg font-heading font-semibold text-foreground">Card Title</h3>
  </div>
  {/* Card content */}
</div>
```

### **Buttons**
- Use the `Button` component with proper variants:
  - `default`, `destructive`, `outline`, `secondary`, `ghost`, `link`
  - `success`, `warning`, `danger` for semantic actions
- Sizes: `xs`, `sm`, `default`, `lg`, `xl`, `icon`

### **Form Elements**
- Use the `Input` component for all form fields
- Include proper labels, descriptions, and error states
- Use semantic colors for validation states

### **Icons**
- Always use the `AppIcon` component with consistent sizing
- Standard sizes: `16px` (small), `20px` (medium), `24px` (large), `32px` (extra large)
- Use semantic colors: `text-primary`, `text-muted-foreground`, etc.

## **Custom CSS Classes & Utilities**

### **Sacred Design Elements**
- **Shadows**: `sacred-shadow`, `sacred-shadow-sm`, `sacred-shadow-lg`
- **Gradients**: `sacred-gradient` (background gradient)
- **Border Radius**: `rounded-sacred` (8px)
- **Transitions**: `reverent-transition` (200ms ease-out)
- **Hover Effects**: `reverent-hover` (subtle scale transform)

### **Circular Elements**
- **Frames**: `circular-frame` for profile images and icons
- **Medallions**: Use `bg-primary rounded-full` with `sacred-shadow-lg`

## **Layout Patterns**

### **Hero Sections**
```jsx
<section className="relative bg-gradient-to-br from-background to-muted min-h-[600px] flex items-center">
  <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 w-full">
    {/* Hero content */}
  </div>
</section>
```

### **Content Sections**
```jsx
<section className="py-16 bg-card">
  <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div className="text-center mb-12">
      <h2 className="font-heading font-semibold text-3xl text-foreground mb-4">Section Title</h2>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
