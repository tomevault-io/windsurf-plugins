---
trigger: always_on
description: Create an automated setup process that handles all configuration:
---

# Setup & Migration Guide for Zen Flow UI

## One-Command Setup

### Quick Start Script
Create an automated setup process that handles all configuration:

```bash
# Single command to set up zen-flow-ui in any React project
npx @gurusharan3107/zen-flow-ui init
```

This command should:
1. Install all required dependencies
2. Configure Tailwind CSS with zen-flow design tokens
3. Set up the utility functions
4. Create example components
5. Add TypeScript configuration if needed

### CLI Tool Enhancement
Enhance the existing [cli/index.cjs](mdc:cli/index.cjs) to support full project initialization:

```javascript
#!/usr/bin/env node

const commands = {
  init: initProject,
  add: addComponent,
  update: updateComponents,
  migrate: runMigration,
  doctor: runDiagnostics
};

async function initProject(options = {}) {
  // Detect project type (Vite, Next.js, CRA, etc.)
  // Install dependencies
  // Configure files
  // Setup examples
}
```

## Installation Methods

### Method 1: Automated Setup (Recommended)
```bash
# For new projects
npx create-react-app my-app --template typescript
cd my-app
npx @gurusharan3107/zen-flow-ui init

# For existing projects
npx @gurusharan3107/zen-flow-ui init
```

### Method 2: Manual Installation
```bash
# Install core package
npm install @gurusharan3107/zen-flow-ui

# Install peer dependencies
npm install react react-dom class-variance-authority clsx tailwind-merge gsap

# Install dev dependencies for TypeScript projects
npm install -D @types/react @types/react-dom @types/gsap
```

### Method 3: Yarn/PNPM Support
```bash
# Yarn
yarn add @gurusharan3107/zen-flow-ui
yarn add react react-dom class-variance-authority clsx tailwind-merge gsap

# PNPM
pnpm add @gurusharan3107/zen-flow-ui
pnpm add react react-dom class-variance-authority clsx tailwind-merge gsap
```

## Automated Configuration

### Package.json Scripts
The CLI tool should add helpful scripts to package.json:
```json
{
  "scripts": {
    "zen:add": "npx @gurusharan3107/zen-flow-ui add",
    "zen:update": "npx @gurusharan3107/zen-flow-ui update",
    "zen:doctor": "npx @gurusharan3107/zen-flow-ui doctor"
  }
}
```

### Tailwind CSS Auto-Configuration
Create [scripts/setup-tailwind.js](mdc:scripts/setup-tailwind.js) that automatically configures Tailwind:

```javascript
export const setupTailwind = async (projectRoot) => {
  const tailwindConfig = `
module.exports = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx}",
    "./node_modules/@gurusharan3107/zen-flow-ui/dist/**/*.js"
  ],
  theme: {
    extend: {
      colors: {
        'zen-void': '#0a0a0a',
        'zen-ink': '#1a1a1a',
        'zen-shadow': '#2a2a2a',
        'zen-stone': '#4a4a4a',
        'zen-mist': '#8a8a8a',
        'zen-cloud': '#dadada',
        'zen-paper': '#fafafa',
        'zen-light': '#ffffff',
        'zen-accent': '#ff4757',
        'zen-water': '#3742fa',
        'zen-leaf': '#26de81',
        'zen-sun': '#fed330',
      },
      spacing: {
        'zen-xs': '4px',
        'zen-sm': '8px',
        'zen-md': '16px',
        'zen-lg': '24px',
        'zen-xl': '32px',
        'zen-2xl': '48px',
        'zen-3xl': '64px',
      },
      borderRadius: {
        'zen-sm': '4px',
        'zen-md': '8px',
        'zen-lg': '12px',
        'zen-xl': '16px',
      },
      animation: {
        'zen-shimmer': 'shimmer 2s linear infinite',
        'zen-accordion-down': 'accordion-down 0.2s ease-out',
        'zen-accordion-up': 'accordion-up 0.2s ease-out',
        'zen-fade-in': 'fade-in 0.3s ease-out',
        'zen-slide-up': 'slide-up 0.3s ease-out',
      },
      keyframes: {
        'shimmer': {
          '0%': { transform: 'translateX(-100%)' },
          '100%': { transform: 'translateX(100%)' },
        },
        'accordion-down': {
          from: { height: 0 },
          to: { height: 'var(--radix-accordion-content-height)' },
        },
        'accordion-up': {
          from: { height: 'var(--radix-accordion-content-height)' },
          to: { height: 0 },
        },
        'fade-in': {
          from: { opacity: 0 },
          to: { opacity: 1 },
        },
        'slide-up': {
          from: { opacity: 0, transform: 'translateY(10px)' },
          to: { opacity: 1, transform: 'translateY(0)' },
        },
      },
    },
  },
  plugins: [],
};`;

  await writeFile(path.join(projectRoot, 'tailwind.config.js'), tailwindConfig);
};
```

### CSS Setup Automation
Auto-create [src/lib/utils.ts](mdc:src/lib/utils.ts):
```typescript
import { type ClassValue, clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// Utility for reduced motion
export function useReducedMotion() {
  if (typeof window === 'undefined') return false;
  return window.matchMedia('(prefers-reduced-motion: reduce)').matches;
}

// Theme utilities
export const zenColors = {
  void: '#0a0a0a',
  ink: '#1a1a1a',
  shadow: '#2a2a2a',
  stone: '#4a4a4a',
  mist: '#8a8a8a',
  cloud: '#dadada',
  paper: '#fafafa',
  light: '#ffffff',
  accent: '#ff4757',
  water: '#3742fa',
  leaf: '#26de81',
  sun: '#fed330',
};
```

## Project Type Detection

### Framework-Specific Setup
Detect and configure for different React frameworks:

```javascript
// scripts/detect-framework.js

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ingpoc/zen-flow-ui](https://github.com/ingpoc/zen-flow-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
