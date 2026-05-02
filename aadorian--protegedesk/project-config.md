---
trigger: always_on
description: **ProtegeDesk** is a modern, web-based ontology editor built with Next.js 16, React 19, and TypeScript 5. It provides a browser-based alternative to Protégé Desktop with features for editing OWL 2 ontologies, reasoning, and visual graph exploration.
---

# GitHub Copilot Instructions for ProtegeDesk

## Project Overview

**ProtegeDesk** is a modern, web-based ontology editor built with Next.js 16, React 19, and TypeScript 5. It provides a browser-based alternative to Protégé Desktop with features for editing OWL 2 ontologies, reasoning, and visual graph exploration.

**Key Technologies**: React 19, Next.js 16, TypeScript 5, Zustand (via React Context), Tailwind CSS 4, Jest, React Flow, Monaco Editor, ELK.js for graph layout

---

## Architecture Overview

### Core Application Structure

```
app/                 # Next.js app directory (server/client boundary)
├── page.tsx         # Main ontology editor page
├── layout.tsx       # Root layout with providers
└── globals.css      # Global styling

components/
├── ui/              # Shadcn/ui base components (Card, Button, Dialog, etc.)
├── ontology/        # Domain components
│   ├── header.tsx              # Top toolbar (import/export, new entity, reasoner)
│   ├── tabs-navigation.tsx      # Left sidebar tabs (Classes, Properties, Individuals)
│   ├── class-tree.tsx           # Hierarchical class browser
│   ├── details-panel.tsx        # Right panel showing selected entity details
│   ├── graph-view.tsx           # React Flow visualization
│   ├── import-export-dialog.tsx # Format handling (Turtle, JSON-LD, OWL/XML)
│   ├── reasoner-dialog.tsx      # Consistency checking UI
│   └── ...                      # Property, Individual, and other entity editors

lib/
├── utils.ts                      # Utility functions (class name merging, etc.)
├── ontology/
│   ├── types.ts                 # Core domain types (Ontology, OntologyClass, etc.)
│   ├── context.tsx              # React Context + useOntology() hook
│   ├── reasoner.ts              # Consistency checking & inference algorithms
│   ├── serializers.tsx          # Format conversion (Turtle, JSON-LD, OWL/XML)
│   └── sample-data.ts           # Mock ontology for development
└── __tests__/                   # Comprehensive unit tests (130+ tests)

hooks/
├── use-toast.ts                 # Toast notification state management
└── __tests__/

public/              # Static assets
styles/              # Global CSS
```

### Data Flow Architecture

1. **State Root**: `useOntology()` context hook provides centralized ontology state
2. **Component Access**: Any component can `useOntology()` to read/dispatch CRUD actions
3. **Immutability Pattern**: All state updates create new Map instances (no mutations)
4. **Selection State**: Currently selected class/property/individual stored separately from ontology
5. **UI Sync**: Components automatically re-render when selections or ontology changes

**Key Pattern Example**:

```typescript
// In any component:
const { ontology, selectedClass, addClass } = useOntology()
```

### Critical Data Structures

The `Ontology` type in [lib/ontology/types.ts](lib/ontology/types.ts) defines:

- `classes: Map<string, OntologyClass>` - Class hierarchy with IRI keys
- `properties: Map<string, OntologyProperty>` - Object/Data properties
- `individuals: Map<string, Individual>` - Instances and named entities

Each entity has: `id`, `iri`, `label`, `comment`, relationships to other entities.

---

## Developer Workflows

### Build & Dev Commands

```bash
npm run dev              # Start dev server (localhost:3000)
npm run build           # Production build
npm start               # Run production server
npm test                # Run Jest tests
npm run test:watch      # Watch mode for tests
npm run test:coverage   # Generate coverage report
npm run lint            # Run ESLint
npm run lint:fix        # Auto-fix linting issues
npm run type-check      # TypeScript type checking
npm run validate        # Full validation: types + lint + format + tests
```

### Key Development Patterns

**Run validation before committing:**

```bash
npm run validate  # This is the pre-commit workflow
```

**Working with ontology logic:**

1. Test files live in `__tests__/` alongside implementation ([lib/ontology/**tests**/](lib/ontology/__tests__/))
2. Reasoning tests: [lib/ontology/**tests**/reasoner.test.ts](lib/ontology/__tests__/reasoner.test.ts) (31 tests)
3. State management tests: [lib/ontology/**tests**/context.test.tsx](lib/ontology/__tests__/context.test.tsx) (23 tests)
4. Serializer tests: [lib/ontology/**tests**/serializers.test.ts](lib/ontology/__tests__/serializers.test.ts) (43 tests)

---

## TypeScript & Code Style

### Type Safety Requirements

- **No `any` types** - Use specific types or `unknown` with proper guards
- **Type imports**: Use `import type { ... }` for types only
- **Interfaces before implementation** - Define shape first, then implement
- **Strict mode enabled** in tsconfig.json

**Component Props Example**:

```typescript
interface ClassNodeProps {
  id: string
  label: string
  onSelect: (id: string) => void
}

export const ClassNode: React.FC<ClassNodeProps> = ({ id, label, onSelect }) => {
  // implementation
}
```

### React Component Conventions

- **Functional components only** - No class components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aadorian/ProtegeDesk](https://github.com/aadorian/ProtegeDesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
