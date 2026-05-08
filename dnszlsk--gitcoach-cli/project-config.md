---
trigger: always_on
description: **GitCoach** is an AI-powered Git coach CLI that prevents mistakes before they happen. Built for the GitHub Copilot CLI Challenge (deadline: February 15, 2026, 23:59 PST).
---

# CLAUDE.md - GitCoach Project

## Project Overview

**GitCoach** is an AI-powered Git coach CLI that prevents mistakes before they happen. Built for the GitHub Copilot CLI Challenge (deadline: February 15, 2026, 23:59 PST).

**Core Problem:** Beginners lose work from Git mistakes; developers waste time searching for solutions; commit messages are generic.

**Solution:** Interactive multilingual CLI with guided menus for beginners, quick shortcuts for experts, contextual analysis via Copilot CLI, real-time error prevention, and intelligent commit generation.

---

## Tech Stack

| Category | Technology |
|----------|------------|
| Language | TypeScript (Node.js) |
| CLI Framework | Oclif |
| Prompts/Menus | Inquirer.js |
| Git Operations | simple-git |
| Display | Chalk, Boxen, cli-table3 |
| i18n | i18next |
| AI Integration | GitHub Copilot CLI |
| Config Storage | Conf |
| Analytics | Local SQLite |
| Testing | Jest + mock-git |

---

## Project Structure

```
gitcoach/
├── bin/
│   └── run.js                    # Entry point
├── src/
│   ├── commands/
│   │   ├── index.ts              # Main menu
│   │   ├── init.ts               # First-time setup
│   │   ├── config.ts             # Configuration menu
│   │   ├── quick.ts              # Expert mode (hotkey)
│   │   └── stats.ts              # Analytics dashboard
│   ├── services/
│   │   ├── git-service.ts        # Git operations wrapper
│   │   ├── copilot-service.ts    # Copilot CLI integration
│   │   ├── analysis-service.ts   # Context analysis
│   │   └── prevention-service.ts # Error detection
│   ├── ui/
│   │   ├── menus/
│   │   │   ├── main-menu.ts
│   │   │   ├── add-menu.ts
│   │   │   ├── commit-menu.ts
│   │   │   ├── branch-menu.ts
│   │   │   └── config-menu.ts
│   │   ├── themes/
│   │   │   ├── colored.ts
│   │   │   └── monochrome.ts
│   │   └── components/
│   │       ├── box.ts
│   │       ├── table.ts
│   │       └── prompt.ts
│   ├── i18n/
│   │   ├── locales/
│   │   │   ├── en.json
│   │   │   ├── fr.json
│   │   │   └── es.json
│   │   └── index.ts
│   ├── config/
│   │   ├── user-config.ts        # User preferences
│   │   └── defaults.ts           # Default settings
│   ├── analytics/
│   │   ├── tracker.ts            # Usage tracking
│   │   └── stats-calculator.ts   # Metrics calculation
│   └── utils/
│       ├── logger.ts
│       ├── validators.ts
│       └── helpers.ts
├── test/
│   ├── unit/
│   └── integration/
├── docs/
│   ├── README.md
│   ├── INSTALLATION.md
│   ├── USAGE.md
│   └── DEMO.md
├── package.json
├── tsconfig.json
└── .eslintrc.js
```

---

## Development Commands

```bash
# Install dependencies
npm install

# Development mode with watch
npm run dev

# Build for production
npm run build

# Run tests
npm test

# Run tests with coverage
npm run test:coverage

# Lint code
npm run lint

# Format code
npm run format

# Link CLI globally for testing
npm link

# Run CLI locally
./bin/run.js
```

---

## Key Features to Implement

### MVP (Must Have)

1. **Interactive Main Menu** - Spring Boot CLI inspired design
2. **Basic Git Operations** - status, add, commit, push with explanations
3. **Copilot CLI Integration** - commit message generation, context analysis
4. **Multilingual Support** - EN, FR, ES via i18next
5. **Adaptive Modes** - Beginner (verbose), Intermediate (tips), Expert (alerts only)
6. **Theme Toggle** - Colored/Monochrome
7. **Critical Error Prevention**:
   - Uncommitted changes warnings before checkout
   - Force push protection
   - Wrong branch alerts
   - Detached HEAD detection
8. **Expert Quick Mode** - Ctrl+Shift+G hotkey for rapid commit+push
9. **Basic Analytics** - Errors prevented, commits generated, time saved
10. **Persistent Configuration** - User preferences saved locally

### Nice to Have (If Time Permits)

- Interactive git log history
- Branch management wizards (merge, rebase)
- Stash helper
- Conflict resolution assistant
- Custom workflows
- Export reports

---

## Copilot CLI Integration Points

### 1. Commit Message Generation
```typescript
// Analyze diff and generate conventional commit message
const prompt = `Analyze git changes and generate conventional commit: ${diff}`;
await exec(`gh copilot suggest "${prompt}"`);
```

### 2. Context Analysis
```typescript
// Analyze current state and suggest next action
const prompt = `Current branch: ${branch}, files: ${files}. What should user do next?`;
await exec(`gh copilot suggest "${prompt}"`);
```

### 3. Error Prediction
```typescript
// Predict if action will cause problems
const prompt = `User wants to: ${action}. State: ${state}. Will this cause problems?`;
await exec(`gh copilot suggest "${prompt}"`);
```

### 4. Educational Explanations
```typescript
// Explain Git concepts for beginners
const prompt = `Explain to a beginner: ${concept}`;
await exec(`gh copilot suggest "${prompt}"`);
```

---

## i18n Keys Structure

All user-facing strings must use i18next keys:

```typescript
// Usage
import { t } from '../i18n';
console.log(t('menu.title'));
console.log(t('warnings.uncommitted'));
console.log(t('warnings.wrongBranch', { branch: 'main' }));
```

Key namespaces:
- `menu.*` - Menu items and titles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DNSZLSK/gitcoach-cli](https://github.com/DNSZLSK/gitcoach-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
