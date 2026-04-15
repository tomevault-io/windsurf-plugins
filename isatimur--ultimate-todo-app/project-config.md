---
trigger: always_on
description: You are an expert in building modern web applications with Next.js, Supabase, TypeScript, and React. Follow these guidelines:
---

You are an expert in building modern web applications with Next.js, Supabase, TypeScript, and React. Follow these guidelines:

**Architecture & Patterns**
1. Follow Next.js 14 App Router with hybrid rendering (SSR/CSR)
2. Implement 3-layer architecture:
   - Presentation (components/ui)
   - Application (lib/hooks, services)
   - Data (supabase, server actions)
3. Use feature-based organization:
   /app
    ├─ (dashboard)
    │  └─ layout.tsx
    ├─ (auth)
    │  └─ layout.tsx 
    ├─ tasks/
    ├─ projects/
    ├─ settings/
    └─ api/ (trpc or route handlers)

**Component Guidelines**
1. Follow atomic design with Shadcn UI:
   - Atoms: buttons, inputs
   - Molecules: form fields, cards
   - Organisms: sidebars, task lists
   - Templates: dashboard layout
2. Component structure:
   - TypeScript interfaces first
   - Hook calls at top
   - Derived state with useMemo
   - Event handlers with useCallback
   - JSX with Tailwind classes
3. Add JSDoc for complex components:
   ```tsx
   /**
    * Interactive task card with drag-n-drop
    * @param task - Task object with subtasks
    * @param onStatusChange - Callback for status updates
    * @param variant - Visual style variant
    */
   ```

**State Management**
1. Hierarchy:
   - Local: useState/useReducer
   - Shared: Context API
   - Server: SWR/React Query
   - Global: Zustand (sparingly)
2. Supabase realtime subscriptions:
   ```typescript:app/components/Todo.tsx
   startLine: 154
   endLine: 177
   ```

**Data & API**
1. Supabase patterns:
   - Use RLS for all tables
   - Implement soft deletes
   - Type-safe queries with zod
   - Server actions for mutations
2. Optimistic updates pattern:
   ```typescript:app/components/Todo.tsx 
   startLine: 164
   endLine: 170
   ```

**UI/UX Standards**
1. Responsive rules:
   - Mobile-first breakpoints
   - Fluid typography with clamp()
   - Grid-based layouts
   - Consistent spacing scale
2. Animation principles:
   - Motion for state changes
   - AnimatePresence for mounts
   - Layout animations for reordering
   ```typescript:app/components/Todo.tsx
   startLine: 1276
   endLine: 1303
   ```

**Performance**
1. Implement:
   - Component memoization
   - Code splitting for routes
   - Supabase query caching
   - Debounced search inputs
   - Virtualized lists
2. Avoid:
   - Prop drilling
   - Large bundle sizes
   - Frequent full re-renders
   - Unoptimized images

**Testing**
1. Test pyramid:
   - Unit: Jest/Vitest (utils/hooks)
   - Integration: Cypress (user flows) 
   - E2E: Playwright (critical paths)
   - Visual: Storybook + Chromatic
2. Key test cases:
   - Auth state transitions
   - Real-time sync
   - Offline behavior
   - Error boundaries

**Security**
1. Must implement:
   - Supabase RLS policies
   - Input sanitization
   - Rate limiting
   - Session validation
   - CSP headers
2. Audit monthly:
   - User permissions
   - Third-party deps
   - Security rules
   - Backup integrity

**Monitoring**
1. Track:
   - Real-time users
   - Task completion rate
   - API response times
   - Error budgets
   - Memory leaks
2. Tools:
   - Supabase Logs
   - Vercel Analytics
   - Sentry/Rollbar
   - Prometheus

**Documentation**
1. Maintain:
   - Component stories
   - API references
   - Data flow diagrams
   - Deployment playbook
   - Incident response
2. Use: 
   - TSDoc for core utils
   - Mermaid for diagrams
   - Swagger for endpoints
   - ADRs for decisions

**Deployment**
- Use Vercel for hosting
- Implement proper CI/CD
- Use proper environment variables
- Monitor performance and errors
- Implement proper backup strategies

### Core Architecture (v2.1)
1. Layered Architecture:
   - Presentation: `/components/*` (TSX)
   - Application: `/lib/*` (Hooks, Services)
   - Data: `/supabase/*` (RLS Policies)
   - AI: `/ai/*` (LLM Integrations) 💎

2. Real-Time Foundation:
   ```typescript:components/ultima-todo-app-component.tsx
   startLine: 50
   endLine: 53  // Supabase client usage
   ```
   - WebSockets for collaborative editing
   - Presence tracking for team members
   - Conflict-free data types (CRDTs)

### Component Standards (v3.0)
1. Compound Components Pattern:
   ```typescript:components/ui/sidebar.tsx
   startLine: 38
   endLine: 136  // Sidebar structure
   ```
   - Root
   - Header
   - Body
   - Footer
   - Item/Trigger/Content

2. Performance Rules:
   - Lazy load non-critical AI features
   - Virtualize lists >100 items
   - Canvas for complex visualizations

### AI Integration Layer 💎
1. Ethical AI Practices:
   - User opt-in for data training
   - Local LLM fallback
   - Audit trails for AI decisions

2. Architecture:
   - Edge-friendly model serving
   - Prompt version control
   - Cost-aware model routing

### Theme Engine Requirements
1. Dynamic Themes:
   - CSS4 color-mix() primitives
   - Semantic token hierarchy
   - Studio-grade contrast ratios
   ```typescript:components/ultima-todo-app-component.tsx
   startLine: 1268
   endLine: 1300  // Themed components
   ```

### Real-Time Sync Protocol
1. Conflict Resolution:
   - Last-write-wins for metadata
   - Operational transforms for text
   - Version vectors for docs

2. Presence Service:
   - Ephemeral user states
   - Location tracking (cursor/scroll)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/isatimur) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
