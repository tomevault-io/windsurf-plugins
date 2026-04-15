---
trigger: always_on
description: - **TypeScript:** Strict typing, no `any` types without justification
---


## 📋 **Development Standards**

### **Code Quality Requirements**

- **TypeScript:** Strict typing, no `any` types without justification
  - **Ref Types:** Always use `RefObject<HTMLElement | null>` in interfaces (see TypeScript Ref Types section in `core.mdc`)
  - **useRef Pattern:** `useRef<HTMLElement>(null)` returns `RefObject<HTMLElement | null>`, always type accordingly
- **React Patterns:** Functional components with hooks, proper error boundaries
- **Performance:** Lazy loading, image optimization, Core Web Vitals optimization
- **Accessibility:** ARIA labels, semantic HTML, keyboard navigation support
- **SEO:** Proper meta tags, structured data, semantic markup
- **Universal Design:** Components that work on web and mobile

### **Naming Conventions**

- **Files:** kebab-case (e.g., `exit-intent-popup.tsx`)
- **Components:** PascalCase (e.g., `ExitIntentPopup`)
- **Functions:** camelCase with descriptive verbs (e.g., `trackUserEngagement`)
- **Constants:** UPPER_SNAKE_CASE (e.g., `GTM_EVENTS`)
- **CSS Classes:** Tailwind utility classes with custom CSS variables

### **Testing Requirements**

- **Unit Tests:** All utility functions and components
- **Integration Tests:** Analytics integration and user flows
- **E2E Tests:** Critical user journeys (lead capture, purchase)
- **Performance Tests:** Core Web Vitals and loading times

**Note:** For comprehensive testing standards, patterns, and best practices, see `testing.mdc`.

## 🔧 **Code Formatting & Quality Tools**

**Prettier:** `npm run format` | `npm run format:check` | `npm run format:staged`. Config: `.prettierrc` (single quotes, semi, 100 width, trailing commas, `prettier-plugin-tailwindcss`). Runs automatically via `lint-staged` on commit.

**ESLint:** `npm run lint`. Config: `eslint.config.mjs` extends `next/core-web-vitals`. Unescaped entities enforced — use `&apos;`, `&quot;` in JSX.

## 🚀 **CI/CD & Automation**

**Note:** For detailed deployment process, monitoring, and operations standards, see `operations.mdc`.

### **Pre-Deployment Checks**

**MANDATORY:** Before pushing to `main`, run `npm run pre-deploy` to verify your code will deploy successfully on Vercel.

**What it checks:**

- Node version (>=22.0.0)
- Dependencies installation (`npm ci`)
- Lint check (`npm run lint`)
- Type check (`npm run type-check`)
- Format check (`npm run format:check`)
- Cleanup check (`npm run cleanup:check`)
- Build check (`npm run build`) - **Most important, this is what Vercel runs**

**Usage:**

```bash
# Run all pre-deployment checks
npm run pre-deploy
```

**See Also:** `operations.mdc` (Deployment Process) for complete pre-deployment checklist and `docs/SCRIPTS.md` (Pre-Deployment Check) for detailed documentation.

### **GitHub Actions CI Workflow**

**File:** `.github/workflows/ci-cd.yml`

**Triggers:**

- Pull requests to `main`
- Pushes to `main`

**Jobs:**

1. **Lint** - Runs `npm run lint`
2. **Type Check** - Runs `npm run type-check`
3. **Format Check** - Runs `npm run format:check`
4. **Build** - Runs `npm run build`

**Requirements:**

- All jobs must pass for PR merge
- Node.js 22 LTS
- Automatic npm cache

**Status:** ✅ Configured and active

### **PR Auto-Labeling**

**File:** `.github/workflows/pr-labels.yml`

**Configuration:** `.github/labeler.yml`

**Labels Applied Automatically:**

- `refactor` - Code refactoring changes
- `bugfix` - Bug fixes
- `ui` - UI/component changes
- `breakpoints` - Responsive/breakpoint changes
- `documentation` - Documentation updates
- `ci` - CI/CD changes
- `codemod` - Codemod transformations
- `config` - Configuration changes
- `api` - API route changes
- `hooks` - React hooks changes
- `types` - TypeScript type changes

**How It Works:**

- Analyzes changed files in PR
- Matches patterns in `.github/labeler.yml`
- Applies appropriate labels automatically
- Runs on PR open, sync, and reopen

**Status:** ✅ Configured and active

### **Automatic CHANGELOG Generation**

**Script:** `scripts/generate-changelog.js`

**Command:** `npm run changelog`

**How It Works:**

- Analyzes git commits since last tag (or all commits)
- Parses Conventional Commits format: `type(scope): subject`
- Groups commits by type (feat, fix, docs, etc.)
- Generates formatted CHANGELOG.md entry

**Commit Types:**

- 🚀 `feat:` - New features
- 🐛 `fix:` - Bug fixes
- 📚 `docs:` - Documentation
- 💎 `style:` - Code style changes
- ♻️ `refactor:` - Code refactoring
- ⚡ `perf:` - Performance improvements
- 🧪 `test:` - Test additions/changes
- 🔧 `chore:` - Maintenance tasks
- ⚙️ `ci:` - CI/CD changes
- 📦 `build:` - Build system changes
- ⏪ `revert:` - Reverted commits

**Output Format:**

```markdown
## [0.1.1] - 2025-01-XX

### 🚀 Features

- New feature description (abc1234)

### 🐛 Fixes

- Bug fix description (def5678)
```

**Usage:**

```bash
# Generate changelog from recent commits
npm run changelog
```

**Status:** ✅ Script created and ready

## 📝 **JSDoc Documentation Standards**

### **JSDoc Requirements**

**MANDATORY:** All public functions, components, and utilities must have JSDoc documentation.

### **JSDoc Template**

````typescript
/**
 * Brief description of what the function/component does.
 *
 * @param {Type} paramName - Description of parameter

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KuschiKuschbert) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
