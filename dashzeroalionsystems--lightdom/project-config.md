---
trigger: always_on
description: This is a blockchain-based DOM optimization platform that includes:
---

# LightDom Enterprise Coding Rules - Enhanced Edition

## Project Overview
This is a blockchain-based DOM optimization platform that includes:
- React/TypeScript frontend with Vite
- Solidity smart contracts for tokenization and optimization proof
- Express.js API server
- Web crawler and optimization engine
- PostgreSQL database integration

## 🚨 CRITICAL RULE #1: ALWAYS SEARCH EXISTING CODE FIRST

> **This is the PRIMARY rule that overrides all others. Never skip this step.**

**MANDATORY PROCESS - BEFORE writing ANY new code:**

### Step 1: Comprehensive Code Search (REQUIRED)

```bash
# 1. Search by functionality keywords
grep -r "relevant.*keyword" services/ src/ scripts/ api/

# 2. Find files by name pattern
find . -name "*relevant*" -type f -not -path "*/node_modules/*"

# 3. Search for class/function definitions
grep -r "class.*Feature\|function.*feature" . --include="*.ts" --include="*.js"

# 4. Check for imports and usage
grep -r "import.*Feature" . --include="*.ts" --include="*.js"

# 5. Use ripgrep for faster searches (if available)
rg "search term" --type ts --type js
```

### Step 2: Review Existing Services (REQUIRED)

Search these directories in order:
1. `services/` - 100+ service files, many may solve your problem
2. `api/` - API route handlers and middleware
3. `src/services/` - TypeScript services and business logic
4. `src/components/` - React components and UI elements
5. `scripts/` - Automation scripts and utilities
6. `utils/` - Utility functions and helpers
7. `src/core/` - Core business logic
8. `src/hooks/` - React hooks for state and logic

### Step 3: Check for Disconnected Code (REQUIRED)

Many features exist but aren't fully integrated. Look for:
- Unused imports
- Commented-out code that works
- Services in `services/` not referenced elsewhere
- Components not imported in any routes
- Utilities not exported in index files

### Step 4: Search Pattern Examples (REQUIRED)

**ALWAYS use these search patterns before creating new code:**

| Creating... | Must Search For... | Example Commands |
|-------------|-------------------|------------------|
| SEO extractor | `seo`, `attribute`, `extract`, `meta`, `og:` | `grep -r "seo" services/ src/` |
| Animation mining | `animation`, `styleguide`, `mine`, `css` | `find . -name "*animation*"` |
| TensorFlow model | `tensorflow`, `model`, `train`, `ml`, `ai` | `grep -r "tensorflow" .` |
| Database ops | `db`, `query`, `pool`, `connection`, `sql` | `grep -r "pool\\.query" .` |
| API endpoint | `router`, `endpoint`, `api`, `route`, `express` | `grep -r "router\\." api/` |
| UI component | `component`, `react`, `dashboard`, `ui` | `find src/components/ -name "*Dashboard*"` |
| Blockchain | `blockchain`, `contract`, `ethereum`, `web3` | `grep -r "ethers" contracts/` |
| Auth system | `auth`, `login`, `token`, `jwt`, `session` | `grep -r "authenticate" api/` |
| Crawler | `crawl`, `scrape`, `puppeteer`, `selenium` | `grep -r "puppeteer" services/` |
| Testing | `test`, `spec`, `mock`, `fixture` | `find test/ -name "*test*"` |

### Step 5: Decision Point (REQUIRED)

**Only proceed to write new code if ALL of these are true:**
- ✅ Completed comprehensive search (Steps 1-4)
- ✅ No existing solution found
- ✅ No existing code can be adapted/extended
- ✅ Confirmed with team/docs that feature doesn't exist
- ✅ Creating truly new functionality

**If ANY existing code is found:**
- ✅ Reuse it directly, or
- ✅ Extend it with new functionality, or
- ✅ Refactor it to be more generic, or
- ✅ Connect it to the rest of the system

### Rationale

We have **substantial existing code** (~100+ services, 200+ files, 50+ components) that may not be fully connected or documented. 

**Benefits of reusing existing code:**
1. 🚀 **Faster development** - No need to write from scratch
2. 🛡️ **Battle-tested** - Existing code is proven to work
3. 🔧 **Maintainable** - Fewer places to update
4. 📚 **Consistent** - Follows established patterns
5. 🐛 **Fewer bugs** - Existing code is already debugged
6. 📊 **Better architecture** - Improves code organization

**Costs of creating new code unnecessarily:**
1. ❌ **Duplication** - Multiple implementations of same feature
2. ❌ **Technical debt** - More code to maintain
3. ❌ **Inconsistency** - Different patterns for same problem
4. ❌ **Higher bug risk** - New code needs debugging
5. ❌ **Wasted effort** - Solving already-solved problems
6. ❌ **Confusion** - Which implementation to use?

### ⚠️ Violation Consequences

**If you create new code without searching first:**
- PR will be rejected with "search-first-violation" label
- Must demonstrate comprehensive search was performed
- Must justify why existing code cannot be used
- May be asked to refactor to use existing code

## Code Quality Standards

### TypeScript/JavaScript
- Use TypeScript for all new code with strict type checking enabled
- Prefer `const` over `let`, avoid `var` completely
- Use meaningful variable and function names (descriptive, not abbreviated)
- Implement proper error handling with try-catch blocks and custom error types
- Use async/await over promises where possible, handle promise rejections properly
- Follow ESLint and Prettier configurations (create if missing)
- Write self-documenting code with JSDoc comments for complex functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DashZeroAlionSystems) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
