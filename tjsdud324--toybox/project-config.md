---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with artifacts in this TOYBOX repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with artifacts in this TOYBOX repository.

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run linter
npm run lint

# Preview production build
npm run preview

# Deploy to GitHub Pages
npm run deploy
```

## Artifact Development Workflow

This is a React-based portfolio for showcasing Claude-generated artifacts. All artifacts share common patterns while maintaining creative freedom.

### Required Patterns

All artifacts must use:
- **React** with TypeScript
- **Tailwind CSS** for styling
- **shadcn/ui** component patterns with **Radix UI primitives**
- **ArtifactMetadata** type for metadata export

### Creating New Artifacts

Place artifacts in `src/artifacts/` as either:
- Direct files: `src/artifacts/my-component.tsx`
- Subdirectories: `src/artifacts/my-component/index.tsx` (for complex artifacts with assets)

**Required Structure:**

```typescript
const ComponentName: React.FC = () => {
  return (
    <div className="p-6">
      {/* Your component implementation */}
      {/* Note: Avoid max-width constraints if your artifact needs fullscreen capability */}
    </div>
  );
};

export default ComponentName;
```

### Metadata System

Metadata can be provided in three ways (in priority order):

1. **External TypeScript file** (recommended for complex artifacts):
   - Direct: `src/artifacts/my-artifact.metadata.ts`
   - Subdirectory: `src/artifacts/my-artifact/metadata.ts`

2. **External JSON file**:
   - Direct: `src/artifacts/my-artifact.metadata.json`
   - Subdirectory: `src/artifacts/my-artifact/metadata.json`

3. **Component export** (simplest approach):
   ```typescript
   export const metadata: ArtifactMetadata = { ... };
   ```

**Metadata Interface:**

```typescript
interface ArtifactMetadata {
  title: string;
  description?: string;
  type: 'react' | 'svg' | 'mermaid';
  tags: string[];
  folder?: string;           // Logical grouping without changing file structure
  createdAt: string;         // ISO date string (use `date -u +"%Y-%m-%dT%H:%M:%SZ"`)
  updatedAt: string;         // ISO date string
  hidden?: boolean;          // Hide from gallery (still accessible via direct URL)
  fullscreen?: boolean;      // Auto-fullscreen in standalone mode
  underMaintenance?: boolean; // Show maintenance banner
}
```

**Example with component export:**

```typescript
import { ArtifactMetadata } from '@/lib/artifactLoader';

export const metadata: ArtifactMetadata = {
  title: 'Component Name',
  description: 'Brief description of what this artifact does',
  type: 'react',
  tags: ['interactive', 'demo'],
  folder: 'Category Name',
  createdAt: '2024-01-15T10:00:00Z',
  updatedAt: '2024-01-15T10:00:00Z',
};

const ComponentName: React.FC = () => {
  return <div className="p-6">{/* Your implementation */}</div>;
};

export default ComponentName;
```

**Example with external TypeScript metadata** (`my-artifact.metadata.ts`):

```typescript
import { ArtifactMetadata } from '@/lib/artifactLoader';

export const metadata: ArtifactMetadata = {
  title: 'My Artifact',
  description: 'A description of my artifact',
  type: 'react',
  tags: ['demo', 'example'],
  createdAt: '2024-01-01',
  updatedAt: '2024-01-15',
};
```

**Example with external JSON metadata** (`my-artifact.metadata.json`):

```json
{
  "title": "My Artifact",
  "description": "A description of my artifact",
  "type": "react",
  "tags": ["demo", "example"],
  "createdAt": "2024-01-01",
  "updatedAt": "2024-01-15"
}
```

### Viewing Modes

- **Gallery view** (`/a/:artifactName`): Shows artifact with metadata, tags, and navigation
- **Standalone view** (`/standalone/:artifactName`): Clean presentation with optional fullscreen toggle

### Gallery Features

- **Type filtering**: Filter by React, SVG, or Mermaid
- **Tag filtering**: Filter by any tag present in artifacts
- **Text search**: Search across title, description, and tags
- **Sorting**: By updated date, created date, or alphabetical
- **Per-card error boundaries**: Individual card errors don't crash the gallery

### Styling Guidelines

- Use **Tailwind CSS** classes for all styling
- Follow **shadcn/ui** component patterns for UI elements
- Leverage **Radix UI primitives** for complex interactions
- Import shadcn/ui components from `@/components/ui/`
- Use responsive design patterns (`sm:`, `md:`, `lg:` prefixes)

**Example with shadcn/ui:**

```typescript
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';

const MyArtifact: React.FC = () => {
  return (
    <div className="p-6 space-y-6">
      {/* Example layout - adjust max-width/centering based on your artifact's needs */}
      <div className="max-w-4xl mx-auto">
        <Card>
          <CardHeader>
            <CardTitle>My Interactive Component</CardTitle>
          </CardHeader>
          <CardContent className="space-y-4">
            <Button variant="default" size="lg">
              Click me
            </Button>
            <Badge variant="secondary">Demo</Badge>
          </CardContent>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tjsdud324/toybox](https://github.com/tjsdud324/toybox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
