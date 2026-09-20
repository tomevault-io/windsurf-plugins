---
trigger: always_on
description: Incremental development approach with always-working builds and controlled changes
---


# Incremental Development Strategy

## Core Development Principles

### 1. **Always-Working Builds**

- `pnpm run dev` must always work
- Never break the development server
- Incremental changes only
- Test each change before proceeding

### 2. **Controlled Development**

- One feature at a time
- Small, focused commits
- Easy rollback capability
- Clear progress tracking

### 3. **Domain Strategy**

- **Primary Domain**: everprompt.ai
- **Development**: localhost:3000
- **Staging**: staging.everprompt.ai (future)
- **Production**: everprompt.ai

## Development Workflow

### **Git Strategy for Solo Developer**

#### **Primary: Main Branch Development**

```bash
# Work directly on main branch
git checkout main
# Make incremental changes
git add .
git commit -m "feat: add dark/light mode toggle"
git push origin main
```

#### **Rollback Points Strategy**

```bash
# Create rollback points after each working feature
git tag v0.1.0-working-basic-layout
git tag v0.2.0-working-theme-toggle
git tag v0.3.0-working-prompt-editor

# Rollback if needed
git checkout v0.2.0-working-theme-toggle
```

#### **Feature Branches (Only for Major Features)**

```bash
# Only for complex features or experiments
git checkout -b feature/prompt-editor
# Make changes
git add .
git commit -m "feat: implement prompt editor"
git push origin feature/prompt-editor
# Merge when ready
git checkout main
git merge feature/prompt-editor
git push origin main
```

#### **When to Use Feature Branches:**

- **Major experiments** (e.g., trying a new UI approach)
- **Complex features** that might break the app
- **Integration work** (e.g., adding authentication)
- **Refactoring** that affects multiple files
- **When you're unsure** if the approach will work

#### **When to Use Main Branch:**

- **Small, incremental changes**
- **UI tweaks and improvements**
- **Bug fixes**
- **Documentation updates**
- **Configuration changes**
- **Most development work**

### **Phase 1: Foundation (Always Working)**

```bash
# Start with basic Next.js app
pnpm run dev  # Must work immediately

# Incremental changes:
1. Basic layout ✅
2. Dark/light mode toggle ✅
3. Simple prompt editor ✅
4. Basic label system ✅
5. n8n JSON parser ✅
```

### **Phase 2: Core Features (One at a Time)**

```bash
# Each step must work before next:
1. Authentication (Clerk) ✅
2. Database setup (Neon + Prisma) ✅
3. Prompt CRUD operations ✅
4. Label management ✅
5. Workflow collection system ✅
```

### **Phase 3: Advanced Features (Controlled)**

```bash
# Build on working foundation:
1. Search and filtering ✅
2. Public sharing ✅
3. Payment integration (Stripe) ✅
4. API endpoints ✅
5. Mobile optimization ✅
```

## Commit Strategy

### **Commit After Each Working Feature**

```bash
# Example commit pattern:
git add .
git commit -m "feat: add dark/light mode toggle

- Implement theme switcher component
- Add CSS variables for theme colors
- Update layout to support theme switching
- Test: pnpm run dev works ✅"
```

### **Major Milestone Commits**

```bash
# After completing major features:
git commit -m "feat: complete MVP prompt editor

- Working prompt editor with autosave
- Dark/light mode toggle
- Basic label system
- n8n JSON parser integration
- Ready for authentication phase

Test: pnpm run dev works ✅
Next: Add Clerk authentication"
```

## Development Checklist

### **Before Each Change**

- [ ] `pnpm run dev` works
- [ ] No TypeScript errors
- [ ] No console errors
- [ ] Current feature is complete

### **After Each Change**

- [ ] `pnpm run dev` still works
- [ ] Feature works as expected
- [ ] No breaking changes
- [ ] Commit the change

### **Before Major Commits**

- [ ] All features working
- [ ] No linting errors
- [ ] TypeScript compilation successful
- [ ] Test in browser
- [ ] Write descriptive commit message

## File Organization

### **Incremental File Structure**

```
everprompt-n8n/
├── app/
│   ├── page.tsx              # Home page (start here)
│   ├── layout.tsx            # Root layout
│   └── globals.css           # Global styles
├── components/
│   ├── ui/                   # Basic UI components
│   ├── features/             # Feature components
│   └── layout/               # Layout components
├── lib/
│   ├── utils.ts              # Utility functions
│   ├── n8n-parser.ts         # n8n JSON parser
│   └── db.ts                 # Database utilities
└── .cursor/rules/            # Development rules
```

### **Component Development Order (shadcn/ui)**

1. **Basic UI Components** (shadcn/ui Button, Input, Card, etc.)
2. **Layout Components** (Header, Sidebar, etc.)
3. **Feature Components** (PromptEditor with shadcn/ui Card, ArcLabels, etc.)
4. **Page Components** (Home, Dashboard, etc.)
5. **Integration Components** (Auth, Database, etc.)

## Testing Strategy

### **Development Testing**

```bash
# Always test these before committing:
pnpm run dev          # Must work
pnpm run build        # Must build successfully
pnpm run lint         # Must pass linting
pnpm run type-check   # Must pass TypeScript checks
```

### **Feature Testing**

- **Manual testing** in browser
- **Console error checking**
- **Responsive design testing**
- **Cross-browser compatibility**

## Rollback Strategy

### **Git Rollback Points**

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitsue-eth/everprompt-n8n-shadcn](https://github.com/mitsue-eth/everprompt-n8n-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
