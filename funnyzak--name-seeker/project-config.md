---
trigger: always_on
description: UI/UX Design and User Experience Rules
---


# UI/UX Design Rules

## 🎨 Design System

### Color Theme
- Use CSS variables to define theme colors
- Support both dark and light themes
- Ensure color contrast meets accessibility standards

```css
/* ✅ Correct color theme */
:root {
  --primary-color: #007bff;
  --primary-hover: #0056b3;
  --secondary-color: #6c757d;
  --success-color: #28a745;
  --warning-color: #ffc107;
  --danger-color: #dc3545;
  --info-color: #17a2b8;
  
  /* Background colors */
  --bg-primary: #ffffff;
  --bg-secondary: #f8f9fa;
  --bg-dark: #343a40;
  
  /* Text colors */
  --text-primary: #212529;
  --text-secondary: #6c757d;
  --text-muted: #adb5bd;
}
```

### Spacing System
- Use consistent spacing units
- Based on 8px grid system
- Use CSS variables to define spacing

```css
/* ✅ Correct spacing system */
:root {
  --spacing-xs: 0.25rem;   /* 4px */
  --spacing-sm: 0.5rem;    /* 8px */
  --spacing-md: 1rem;      /* 16px */
  --spacing-lg: 1.5rem;     /* 24px */
  --spacing-xl: 2rem;      /* 32px */
  --spacing-xxl: 3rem;     /* 48px */
}
```

## 🎯 Component Design

### Button Components
- Use consistent button styles
- Support different states (hover, active, disabled)
- Provide clear visual feedback

```css
/* ✅ Correct button styles */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: var(--spacing-sm) var(--spacing-md);
  border: 1px solid transparent;
  border-radius: 0.375rem;
  font-size: 0.875rem;
  font-weight: 500;
  line-height: 1.5;
  text-align: center;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.15s ease-in-out;
}

.btn-primary {
  background-color: var(--primary-color);
  border-color: var(--primary-color);
  color: white;
}

.btn-primary:hover {
  background-color: var(--primary-hover);
  border-color: var(--primary-hover);
}

.btn:disabled {
  opacity: 0.65;
  cursor: not-allowed;
}
```

### Form Components
- Use consistent input styles
- Provide clear labels and placeholders
- Support validation state display

```css
/* ✅ Correct form styles */
.form-group {
  margin-bottom: var(--spacing-md);
}

.form-label {
  display: block;
  margin-bottom: var(--spacing-xs);
  font-weight: 500;
  color: var(--text-primary);
}

.form-control {
  display: block;
  width: 100%;
  padding: var(--spacing-sm) var(--spacing-md);
  font-size: 0.875rem;
  line-height: 1.5;
  color: var(--text-primary);
  background-color: var(--bg-primary);
  border: 1px solid #ced4da;
  border-radius: 0.375rem;
  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

.form-control:focus {
  border-color: var(--primary-color);
  outline: 0;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.form-control.is-invalid {
  border-color: var(--danger-color);
}
```

## 📱 Responsive Design

### Breakpoint System
- Use mobile-first design approach
- Define clear breakpoints
- Ensure good experience on all devices

```css
/* ✅ Correct responsive design */
/* Mobile first */
.container {
  width: 100%;
  padding: 0 var(--spacing-md);
  margin: 0 auto;
}

/* Tablet */
@media (min-width: 768px) {
  .container {
    max-width: 750px;
  }
}

/* Desktop */
@media (min-width: 992px) {
  .container {
    max-width: 970px;
  }
}

/* Large screens */
@media (min-width: 1200px) {
  .container {
    max-width: 1170px;
  }
}
```

### Grid System
- Use CSS Grid or Flexbox
- Support different screen sizes
- Maintain layout consistency

```css
/* ✅ Correct grid system */
.grid {
  display: grid;
  gap: var(--spacing-md);
}

.grid-cols-1 {
  grid-template-columns: 1fr;
}

.grid-cols-2 {
  grid-template-columns: repeat(2, 1fr);
}

.grid-cols-3 {
  grid-template-columns: repeat(3, 1fr);
}

@media (max-width: 768px) {
  .grid-cols-2,
  .grid-cols-3 {
    grid-template-columns: 1fr;
  }
}
```

## 🎭 Animations and Transitions

### Transition Effects
- Use consistent transition timing
- Provide smooth user experience
- Avoid excessive animations

```css
/* ✅ Correct transition effects */
.transition {
  transition: all 0.15s ease-in-out;
}

.transition-fast {
  transition: all 0.1s ease-in-out;
}

.transition-slow {
  transition: all 0.3s ease-in-out;
}

/* Hover effects */
.hover-lift:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

/* Loading animation */
@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.spinner {
  animation: spin 1s linear infinite;
}
```

### Micro-interactions
- Provide immediate visual feedback
- Use appropriate animation duration
- Maintain animation consistency

```css
/* ✅ Correct micro-interactions */
.btn {
  transition: all 0.15s ease-in-out;
}

.btn:active {
  transform: translateY(1px);
}

/* Loading state */
.btn-loading {
  position: relative;
  color: transparent;
}

.btn-loading::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 1rem;
  height: 1rem;
  margin: -0.5rem 0 0 -0.5rem;
  border: 2px solid transparent;
  border-top-color: currentColor;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}
```

## ♿ Accessibility

### Keyboard Navigation
- Support Tab key navigation
- Provide clear focus indicators
- Support keyboard shortcuts

```css
/* ✅ Correct keyboard navigation */
.focusable:focus {
  outline: 2px solid var(--primary-color);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [funnyzak/name-seeker](https://github.com/funnyzak/name-seeker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
