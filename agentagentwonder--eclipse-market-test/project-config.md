---
trigger: always_on
description: **Build/Dev:** `npm run tauri dev` (dev), `npm run tauri build` (production)
---

# Eclipse Market Pro - Agent Guide

## Commands

**Build/Dev:** `npm run tauri dev` (dev), `npm run tauri build` (production)
**Testing:** `npm run test` (unit), `npm run test:e2e` (Playwright), `npm run test:e2e:ui` (Playwright UI)
**Test single file:** `npm run test -- <file-path>` (e.g., `npm run test -- tests/stocks.test.ts`)
**Lint:** `npm run lint` (check), `npm run lint:fix` (auto-fix)
**Format:** `npm run format` (auto-format), `npm run format:check` (check only)

## Architecture

- **Stack:** Vite + React 18 + TypeScript frontend, Rust/Tauri backend
- **State:** Zustand stores in `src/store/`
- **Backend:** Tauri commands in `src-tauri/src/` (modular: `wallet/`, `market/`, `trading/`, `ai/`, etc.)
- **Database:** SQLite via sqlx, managed in `src-tauri/src/db.rs`
- **Key Modules:** Phantom wallet integration, Solana blockchain, real-time market data, AI sentiment analysis
- **Tests:** Vitest (unit) in `tests/`, Playwright (e2e) in `e2e/`, mobile tests in `mobile-tests/`

## Project Structure

src/
├── components/ # React components
├── store/ # Zustand state stores
├── hooks/ # Custom React hooks
├── utils/ # Utility functions
├── types/ # TypeScript definitions
├── services/ # API services
├── App.tsx # Main app
└── main.tsx # Entry point

src-tauri/src/
├── wallet/ # Wallet operations
├── market/ # Market data
├── trading/ # Trade execution
├── ai/ # AI/sentiment
├── db.rs # SQLite database
└── main.rs # Entry point

tests/ # Vitest unit tests
e2e/ # Playwright e2e tests
mobile-tests/ # Mobile tests


## Code Style

- **Format:** Prettier (semi, singleQuote, printWidth: 100, tabWidth: 2, arrowParens: avoid)
- **TypeScript:** Strict mode, ESNext target, react-jsx (no React import needed)
- **React:** Functional components, hooks recommended
- **Naming:** `camelCase` (JS/TS), `snake_case` (Rust), `PascalCase` (components/types), `UPPER_SNAKE_CASE` (constants)
- **No `any`:** Use proper types; `@typescript-eslint/no-explicit-any` is warn level
- **Unused vars:** Prefix with `_` to ignore (e.g., `_unusedVar`)
- **Error Handling:** Use `anyhow`/`thiserror` in Rust, proper `try/catch` in TypeScript

## Error Analysis Framework

### 1. Syntax & Compilation Errors
- Missing semicolons, brackets, parentheses (Prettier enforces)
- Malformed expressions, invalid operators, illegal tokens
- TypeScript strict mode violations
- Rust compilation errors
- **Check:** Code compiles/builds without errors

### 2. Type System & Compatibility
- `any` types (should be minimal, warn-only)
- Type mismatches between Tauri (Rust) and frontend (TypeScript)
- Missing type definitions, incorrect generics
- Implicit type coercions causing runtime errors
- **Check:** All types explicit, no `any` except necessary

### 3. Tauri Bridge Integrity
- **Critical:** Command name mismatches: `#[tauri::command]` vs `invoke('...')`
- Parameter type mismatches (Rust ↔ TypeScript)
- Async handling (all commands return Promises)
- Serialization errors (serde)
- **Example:**
  ```rust
  #[tauri::command]
  async fn get_balance(address: String) -> Result<f64, String>
const balance = await invoke<number>('get_balance', { address: addr });
4. Import & Dependency Integrity
All imports resolve correctly (check tsconfig paths)
No circular dependencies
Missing deps in package.json or Cargo.toml
Unused imports (ESLint flags)
Check: No import 404s, paths match tsconfig aliases
5. Naming & Reference Errors
Undefined variables, functions, classes, modules
Typos in identifiers
Convention violations (camelCase vs snake_case)
Unused vars not prefixed with _
Check: Naming consistency across files
6. Solana Integration Errors
Wallet adapter config (Phantom)
Transaction signing flow broken
Invalid public key formats
RPC endpoint connectivity issues
Network mismatch (devnet vs mainnet)
Check: Wallet connection flow, transaction handling
7. State Management (Zustand)
Store selectors not typed
Direct state mutation (use set functions)
Subscriptions not cleaned up
Pattern:
interface Store {
  data: Data;
  setData: (data: Data) => void;
}
const useStore = create<Store>(set => ({
  data: {},
  setData: data => set({ data }),
}));
8. Database (SQLite) Errors
SQL syntax in db.rs
sqlx query macro compile-time errors
Connection pool leaks
Schema migration issues
Check: Parameterized queries, no SQL injection
9. React Hooks Rules
Hooks called conditionally or in loops
Hooks not at top level
Dependency arrays incomplete
Fix: Move conditions inside hooks, not around them
10. Logic & Runtime Errors
Null/undefined reference errors
Off-by-one errors, incorrect array indexing
Infinite loops, unreachable code
Unhandled promise rejections
Check: All async properly awaited or .catch()ed
11. Performance Issues
Unnecessary re-renders (use React DevTools Profiler)
Heavy computations in render
Missing memoization (useMemo, useCallback)
WebSocket connection leaks
Check: Bundle size, render performance
12. Security Concerns
Critical: Private keys, API keys in source
Sensitive data in localStorage (use Tauri secure storage)
SQL injection vulnerabilities
XSS in dynamic content
Check: All secrets in environment variables only
13. Documentation & Comments
Typos in comments/
docs

Outdated comments not matching code
Misleading documentation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentAgentwonder/Eclipse-Market-test](https://github.com/AgentAgentwonder/Eclipse-Market-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
