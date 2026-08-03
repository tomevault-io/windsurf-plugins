---
trigger: always_on
description: > **Purpose**: This document serves as a comprehensive guide for AI coding agents and developers working with the B2B Buyer Portal codebase. It explains architectural patterns, testing practices, and critical workflows.
---

# AI Agent Guide: B2B Buyer Portal

> **Purpose**: This document serves as a comprehensive guide for AI coding agents and developers working with the B2B Buyer Portal codebase. It explains architectural patterns, testing practices, and critical workflows.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Critical: Working Directory Rules](#️-critical-working-directory-rules)
- [Architecture Philosophy & Patterns](#-architecture-philosophy--patterns)
- [Development Commands](#-development-commands)
- [File Structure Patterns](#-file-structure-patterns)
- [Testing Guidelines](#-testing-guidelines)
- [Import Rules & Path Aliases](#-import-rules--path-aliases)
- [Code Generation](#-code-generation)
- [Common Pitfalls & Anti-Patterns](#-common-pitfalls--anti-patterns)
- [Redux State (Legacy - Use Sparingly)](#-redux-state-legacy---use-sparingly)
- [Key Configuration Files](#-key-configuration-files)
- [Pull Request Guidelines](#-pull-request-guidelines)
- [Component Structure Examples](#-component-structure-examples)
- [Testing Environment Setup](#-testing-environment-setup)
- [Quick Reference: Testing Checklist](#-quick-reference-testing-checklist)

---

## 🤖 AI Coding Agent Setup (BigCommerce Team)

Install the internal `b2b-buyer-portal` plugin from the internal plugin repository to get the full context.

---

## 🎯 Project Overview

**B2B Buyer Portal** is a React-based frontend application for BigCommerce's B2B Edition buyer experience.

### Tech Stack

- **Framework**: React 18
- **Language**: TypeScript
- **Build Tool**: Vite
- **Testing**: Vitest + Testing Library
- **State Management**: Redux Toolkit (legacy, minimizing use)
- **Styling**: Material-UI (MUI) + Emotion
- **Monorepo**: Turborepo (single package structure)

### Project Structure

```
b2b-buyer-portal-app/
├── apps/
│   └── storefront/              ← Main application (work here!)
│       ├── src/                 ← Source code
│       ├── tests/               ← Test utilities & builders
│       ├── vite.config.ts       ← Build & test config
│       └── package.json         ← Dependencies & scripts
├── config/                      ← Deployment configs
├── docs/                        ← Documentation
└── rfc/                         ← RFCs & GraphQL schemas
```

### Current State

⚠️ **The codebase is undergoing architectural improvements**. Legacy patterns exist but **should not be replicated** in new code. Always follow the [Target Architecture](#target-architecture-use-this) patterns outlined below.

---

## ⚠️ Critical: Working Directory Rules

### **ALL commands MUST be run from `apps/storefront/`**

```bash
# Navigate to the correct directory first
cd apps/storefront

# Then run commands
yarn dev
yarn test
yarn build
```

### Why This Matters

This project uses Turborepo as a monorepo structure, even though it currently contains only one package (`storefront`). All package dependencies, scripts, and configurations are defined at the `apps/storefront/` level, not at the root.

**Running commands from the wrong directory will cause:**
- ❌ Commands not found
- ❌ Wrong dependencies loaded
- ❌ Incorrect build outputs
- ❌ Test failures

---

## 🏛 Architecture Philosophy & Patterns

### Target Architecture (Use This)

When writing new code, follow these principles:

#### 1. **Matroska-Style Structure**
Each component/page owns its domain-specific dependencies. Group related files together.

```
src/pages/Invoice/
├── index.tsx                    # Main component
├── index.test.tsx               # Tests
├── InvoiceHeader.tsx            # Page-specific component
├── InvoicePayments/             # More complex sub-component
│   ├── index.tsx
│   └── usePaymentHistory.ts     # Component-specific hook
└── hooks/
    └── useInvoiceData.ts        # Page-specific hook
```

#### 2. **Prefer Props Over Context/Redux**
Pass data through component props whenever possible. This makes data flow explicit and components more testable.

```typescript
// ✅ GOOD: Explicit props
interface Props {
  orderId: string;
  customerId: string;
}

function OrderDetails({ orderId, customerId }: Props) {
  // ...
}

// ❌ BAD: Hidden dependencies
function OrderDetails() {
  const orderId = useAppSelector(state => state.order.id);
  const customerId = useContext(CustomerContext);
  // ...
}
```

#### 3. **Co-located Files**
Keep related files together. Tests live next to the code they test.

```
InvoiceList/
├── index.tsx              # Component
├── index.test.tsx         # Desktop tests
├── index.mobile.test.tsx  # Mobile-specific tests
└── useInvoiceFilters.ts   # Component-specific logic
```

#### 4. **URL-Driven State**
Use route parameters and query strings for state that should persist across page loads.

```typescript
// ✅ GOOD: State in URL
function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const page = Number(searchParams.get('page')) || 1;
  const category = searchParams.get('category') || 'all';
  
  return (
    <button onClick={() => setSearchParams({ page: page + 1, category })}>
      Next Page
    </button>
  );
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bigcommerce/b2b-buyer-portal](https://github.com/bigcommerce/b2b-buyer-portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
