---
trigger: always_on
description: Guidelines for frontend development in Langflow, focusing on React/TypeScript UI components, build processes, and frontend testing.
---


## 1. Frontend Environment Setup

### Prerequisites
- **Node.js:** v22.12 LTS for JavaScript runtime
- **Package Manager:** npm (v10.9) for dependency management
- **Development Tools:** Vite for build tooling

### Frontend Service
```bash
make frontend  # Start Vite dev server on port 3000
```
- Hot-reload enabled for UI changes
- Access at: http://localhost:3000/
- Frontend source: `src/frontend/`

---

## 2. Frontend Structure

### Directory Layout
```
src/frontend/src/
├── components/          # Reusable UI components
├── pages/              # Page-level components
├── icons/              # Component icons and lazy loading
├── stores/             # State management (Zustand)
├── types/              # TypeScript type definitions
├── utils/              # Utility functions
├── hooks/              # Custom React hooks
├── services/           # API service functions
└── assets/             # Static assets
```

### Key Technologies
- **React 18** with TypeScript
- **Vite** for build tooling and dev server
- **Tailwind CSS** for styling
- **Zustand** for state management
- **React Flow** for flow graph visualization
- **Lucide React** for icons

---

## 3. Frontend Code Quality

### Formatting
```bash
make format_frontend  # Format TypeScript/JavaScript code
```

### Linting
```bash
make lint  # Run ESLint and TypeScript checks
```

### Testing
```bash
make tests_frontend  # Run frontend tests (requires additional setup)
```

### Pre-commit Workflow
1. Run `make format_frontend`
2. Run `make lint`
3. Test changes in browser
4. Commit changes

---

## 4. State Management

### Zustand Stores
```typescript
// stores/myStore.ts
import { create } from 'zustand';

interface MyState {
  value: string;
  setValue: (value: string) => void;
}

export const useMyStore = create<MyState>((set) => ({
  value: '',
  setValue: (value) => set({ value }),
}));
```

### Using Stores in Components
```typescript
// components/MyComponent.tsx
import { useMyStore } from '@/stores/myStore';

export function MyComponent() {
  const { value, setValue } = useMyStore();

  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

---

## 5. API Integration

### Service Functions
```typescript
// services/api.ts
import { api } from '@/controllers/API';

export async function createFlow(flowData: FlowData) {
  const response = await api.post('/flows/', flowData);
  return response.data;
}

export async function getFlows() {
  const response = await api.get('/flows/');
  return response.data;
}
```

### Error Handling
```typescript
// hooks/useApi.ts
import { useState, useCallback } from 'react';

export function useApi<T>(apiFunction: (...args: any[]) => Promise<T>) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const execute = useCallback(async (...args: any[]) => {
    try {
      setLoading(true);
      setError(null);
      const result = await apiFunction(...args);
      return result;
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
      throw err;
    } finally {
      setLoading(false);
    }
  }, [apiFunction]);

  return { execute, loading, error };
}
```

---

## 6. React Flow Integration

### Flow Graph Components
```typescript
// components/FlowGraph.tsx
import ReactFlow, {
  Node,
  Edge,
  Controls,
  Background
} from 'reactflow';

interface FlowGraphProps {
  nodes: Node[];
  edges: Edge[];
  onNodesChange: (changes: NodeChange[]) => void;
  onEdgesChange: (changes: EdgeChange[]) => void;
}

export function FlowGraph({
  nodes,
  edges,
  onNodesChange,
  onEdgesChange
}: FlowGraphProps) {
  return (
    <div className="w-full h-full">
      <ReactFlow
        nodes={nodes}
        edges={edges}
        onNodesChange={onNodesChange}
        onEdgesChange={onEdgesChange}
        fitView
      >
        <Controls />
        <Background />
      </ReactFlow>
    </div>
  );
}
```

### Custom Node Types
```typescript
// components/nodes/ComponentNode.tsx
import { memo } from 'react';
import { Handle, Position } from 'reactflow';

interface ComponentNodeProps {
  data: {
    label: string;
    icon?: string;
  };
}

export const ComponentNode = memo(({ data }: ComponentNodeProps) => {
  return (
    <div className="px-4 py-2 shadow-md rounded-md bg-white border">
      <Handle type="target" position={Position.Top} />

      <div className="flex items-center">
        {data.icon && (
          <img src={data.icon} alt="" className="w-4 h-4 mr-2" />
        )}
        <span className="text-sm">{data.label}</span>
      </div>

      <Handle type="source" position={Position.Bottom} />
    </div>
  );
});
```

---

## 7. Styling with Tailwind

### Component Styling
```typescript
// components/Button.tsx
import { cn } from '@/utils/cn';

interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
  onClick?: () => void;
}

export function Button({
  variant = 'primary',
  size = 'md',
  children,
  onClick
}: ButtonProps) {
  return (
    <button
      className={cn(
        'rounded-md font-medium transition-colors',
        {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UhtredRegress/LangFlowClone](https://github.com/UhtredRegress/LangFlowClone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
