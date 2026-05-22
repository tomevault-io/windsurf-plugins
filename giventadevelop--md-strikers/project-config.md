---
trigger: always_on
description: Pattern for creating separate layouts for different sections/routes in Next.js App Router
---


# Conditional Layout Separation Pattern

## **Overview**
This pattern enables creating completely separate layouts for different sections of a Next.js application while maintaining a single root layout. Useful for creating public sections (like marketing sites, documentation, or church websites) that need different styling and navigation from the main authenticated application.

## **Problem Solved**
- **Layout Duplication**: Prevents main app header/footer from showing alongside section-specific headers/footers
- **Authentication Separation**: Allows public sections to work without authentication requirements
- **Styling Isolation**: Enables completely different design systems for different sections
- **Hydration Issues**: Avoids Next.js App Router hydration errors from nested HTML tags

## **Implementation Pattern**

### **1. Create ConditionalLayout Component**

```typescript
// src/components/ConditionalLayout.tsx
'use client';

import React from 'react';
import { usePathname } from 'next/navigation';

interface ConditionalLayoutProps {
  children: React.ReactNode;
  header: React.ReactNode;
  footer: React.ReactNode;
}

export default function ConditionalLayout({ children, header, footer }: ConditionalLayoutProps) {
  const pathname = usePathname();

  // Define which routes should use separate layout
  const isSeparateLayoutRoute = pathname?.startsWith("/section-prefix") ?? false;

  // For separate layout routes, just render children (section handles its own header/footer)
  if (isSeparateLayoutRoute) {
    return <>{children}</>;
  }

  // For main app routes, render the full layout with header and footer
  return (
    <>
      {header}
      <div className="flex-1 flex flex-col">
        {children}
      </div>
      {footer}
    </>
  );
}
```

### **2. Update Root Layout**

```typescript
// src/app/layout.tsx
import ConditionalLayout from "../components/ConditionalLayout";
import Header from "../components/Header";
import Footer from "../components/Footer";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <ClerkProvider>
          <TrpcProvider>
            <ConditionalLayout
              header={<Header hideMenuItems={false} />}
              footer={<Footer />}
            >
              {children}
            </ConditionalLayout>
          </TrpcProvider>
        </ClerkProvider>
      </body>
    </html>
  );
}
```

### **3. Create Section-Specific Layout**

```typescript
// src/app/section-prefix/layout.tsx
import React from 'react';
import { Metadata } from 'next';
import SectionHeader from './components/SectionHeader';
import SectionFooter from './components/SectionFooter';
import './section-globals.css'; // Section-specific styles

export const metadata: Metadata = {
  title: {
    template: '%s | Section Name',
    default: 'Section Name',
  },
  description: 'Section-specific description',
};

interface SectionLayoutProps {
  children: React.ReactNode;
}

export default function SectionLayout({ children }: SectionLayoutProps) {
  return (
    <div className="section-layout min-h-screen bg-section-background flex flex-col">
      <SectionHeader />

      <main className="section-main flex-1">
        {children}
      </main>

      <SectionFooter />
    </div>
  );
}
```

### **4. Create Section-Specific Styling**

```css
/* src/app/section-prefix/section-globals.css */
@import '../globals.css';

/* Override main app styles for section */
body {
  background-color: var(--section-background) !important;
  color: var(--section-foreground) !important;
  font-family: 'SectionFont', sans-serif !important;
}

/* Section-specific CSS variables */
:root {
  --section-background: #F5F1E8;
  --section-foreground: #2D2A26;
  --section-primary: #8B7D6B;
  /* ... other section-specific variables */
}

/* Section layout isolation */
.section-layout {
  background-color: var(--section-background) !important;
  color: var(--section-foreground) !important;
}

/* Override any conflicting main app styles */
.section-layout * {
  box-sizing: border-box;
}
```

### **5. Update Middleware for Public Routes**

```typescript
// src/middleware.ts
export default authMiddleware({
  publicRoutes: [
    "/",
    "/section-prefix",
    "/section-prefix/(.*)",
    // ... other public routes
  ],
  // ... rest of middleware config
});
```

## **Key Implementation Details**

### **Route Detection Logic**
```typescript
// Single section
const isSeparateLayoutRoute = pathname?.startsWith("/mosc") ?? false;

// Multiple sections
const separateLayoutSections = ["/mosc", "/docs", "/marketing"];
const isSeparateLayoutRoute = separateLayoutSections.some(section =>
  pathname?.startsWith(section)
) ?? false;

// Regex pattern matching
const isSeparateLayoutRoute = pathname?.match(/^\/(mosc|docs|marketing)/) !== null;
```

### **Null Safety**
```typescript
// Always handle potential null pathname
const pathname = usePathname();
const isSeparateLayoutRoute = pathname?.startsWith("/section") ?? false;
```

### **Conditional Provider Wrapping**
```typescript
// For sections that don't need authentication
export default function ConditionalLayout({ children, header, footer }: ConditionalLayoutProps) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
