---
trigger: always_on
description: End-to-end guidance for AppRun apps in TypeScript using MVU including component patterns, event handling, state management (including async generators), routing/navigation with params and guards, and testing with vitest. Use when designing or reviewing AppRun components, wiring routes, managing state flows, or writing AppRun tests.
---


# AppRun Skills

## Overview

- Build AppRun apps with MVU (Model-View-Update) in TypeScript.
- Prefer pure update functions for testability.
- Use `mounted()` for components embedded in JSX.
- Use `state = async` only for top-level routed pages that must load async data.

## Project Setup

### Recommended Project Structure

```
web/                        # Frontend application root
├── index.html              # Entry HTML file
├── package.json            # Dependencies and scripts
├── vite.config.js          # Vite configuration
├── src/
│   ├── main.tsx            # Application entry point (routes registration)
│   ├── api.ts              # REST API client (optional)
│   ├── styles.css          # Application styles
│   ├── tsconfig.json       # TypeScript configuration
│   ├── components/         # Reusable UI components
│   │   ├── Layout.tsx      # Root layout container
│   │   └── ...             # Other reusable components
│   ├── domain/             # Business logic modules (optional)
│   │   └── ...             # Pure functions and business logic
│   ├── pages/              # Top-level page components
│   │   ├── Home.tsx        # Example: Home page
│   │   └── ...             # Other route pages
│   ├── types/              # TypeScript type definitions
│   │   ├── index.ts        # Shared types
│   │   └── jsx.d.ts        # JSX type declarations
│   └── utils/              # Utility functions
└── public/                 # Static assets (optional)
```

### Vite Configuration

```javascript
import { defineConfig } from 'vite'

export default defineConfig({
  build: {
    outDir: 'dist',
    emptyOutDir: true,
  },
  server: {
    port: 8080,
    open: true,
    historyApiFallback: true,  // SPA mode
    proxy: {
      // Proxy API requests to backend
      '/api': {
        target: 'http://127.0.0.1:3000',
        changeOrigin: true,
        secure: false
      }
    }
  }
})
```

### Package.json

```json
{
  "name": "my-apprun-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "check": "tsc --noEmit"
  },
  "devDependencies": {
    "apprun": "^3.38.0",
    "typescript": "^5.0.0",
    "vite": "^5.0.0"
  }
}
```

### TypeScript Configuration

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "react",
    "jsxFactory": "app.createElement",
    "jsxFragmentFactory": "app.Fragment",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

**Critical Settings for AppRun:**
- `jsx: "react"` - Enables JSX syntax
- `jsxFactory: "app.createElement"` - Uses AppRun's JSX factory
- `jsxFragmentFactory: "app.Fragment"` - Uses AppRun's Fragment support
- `moduleResolution: "bundler"` - Optimized for Vite

### Entry Points

**HTML Entry (`index.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My AppRun App</title>
</head>
<body>
  <div id="root"></div>
  <script type="module" src="src/main.tsx"></script>
</body>
</html>
```

**Application Entry (`src/main.tsx`):**

```tsx
import app from 'apprun';
import Layout from './components/Layout';
import Home from './pages/Home';
import About from './pages/About';
import './styles.css';

app.render('#root', <Layout />);

app.addComponents('#pages', {
  '/': Home,
  '/about': About,
});
```

**Layout Component (`src/components/Layout.tsx`):**

```tsx
import app from 'apprun';

export default () => (
  <div id="app">
    <div id="pages"></div>
  </div>
);
```

### Styling Options

**Option 1: Vanilla CSS**

```css
/* src/styles.css */
:root {
  --color-primary: #007bff;
  --color-text: #333;
  --spacing-unit: 8px;
}

body {
  font-family: system-ui, -apple-system, sans-serif;
  color: var(--color-text);
  margin: 0;
  padding: 0;
}
```

**Option 2: Tailwind CSS v4**

Install Tailwind v4:
```bash
npm install -D tailwindcss@next @tailwindcss/vite@next
```

Update `vite.config.js`:
```javascript
import { defineConfig } from 'vite'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [tailwindcss()],
  // ... other config
})
```

Import in `src/styles.css`:
```css
@import "tailwindcss";
```

Use in components:
```tsx
<div className="flex items-center gap-4 p-4 bg-white rounded-lg shadow">
  <h1 className="text-2xl font-bold">Hello World</h1>
</div>
```

**Option 3: CSS Modules**

```tsx
import styles from './MyComponent.module.css';

export default () => (
  <div className={styles.container}>
    <h1 className={styles.title}>Hello</h1>
  </div>
);
```

### API Client Pattern

```typescript
// src/api.ts
const API_BASE_URL = '/api';


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yysun/apprun](https://github.com/yysun/apprun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
