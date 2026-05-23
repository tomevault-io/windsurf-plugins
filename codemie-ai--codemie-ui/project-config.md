---
trigger: always_on
description: **Purpose**: AI-optimized execution guide for Claude Code agents working with the CodeMie UI codebase
---

# CLAUDE.md

**Purpose**: AI-optimized execution guide for Claude Code agents working with the CodeMie UI codebase

---

## ⚡ INSTANT START (Read First)

### 1. Critical Rules (MANDATORY - Always Check)

| Rule | Trigger | Action Required |
|------|---------|-----------------|
| 🚨 **Tailwind Only** | ANY styling task | ONLY use Tailwind classes → [Styling Guide](#styling-guide) |
| 🚨 **Popup Component** | Modal/dialog task | NEVER use Dialog directly → [Modal Patterns](#modal-patterns) |
| 🚨 **API Client** | API call task | Use fetch wrapper (NOT Axios) → [API Integration](#api-integration) |
| 🚨 **State Management** | Global state task | Use Valtio stores → [State Management](#state-management) |
| 🚨 **Code Review** | "do code review" request | Invoke skill directly → `Skill("code-reviewer")` — skill collects context itself |
| 🚨 **Onboarding Guide** | ANY change to onboarding types, store, flows, or UI entry points | Update `.codemie/onboarding-flows/FLOW-CREATION-GUIDE.md` to reflect the change |

**Emergency Recovery**: If imports fail → Check [Troubleshooting](#-troubleshooting-quick-reference)

### 2. Task Classifier (What am I doing?)

**Understand user intent → Map to category → Load guides → Execute**

**Match user's intent/goal to category, then load corresponding guides:**

| Category | User Intent / Purpose | Example Requests | P0 Guide (Required) | P1 Guide (Optional) |
|----------|----------------------|------------------|---------------------|---------------------|
| **Architecture** | System structure, feature planning, design decisions | "How should I structure...?", "Where should this go?", "Plan new feature" | `architecture/project-architecture.md` | `architecture/routing-patterns.md` |
| **UI Components** | Creating/modifying components, UI elements | "Create component", "Add button", "Build card" | `components/component-patterns.md` | `components/reusable-components.md` |
| **Forms** | Forms with validation, input handling | "Create form", "Add validation", "Handle input" | `patterns/form-patterns.md` | `components/component-patterns.md` |
| **Modals/Dialogs** | Modal windows, popups, dialogs | "Show modal", "Create popup", "Add dialog" | `patterns/modal-patterns.md` | `components/component-patterns.md` |
| **State** | Global state, data management | "Manage state", "Store data", "Share data between components" | `patterns/state-management.md` | `development/api-integration.md` |
| **API/Backend** | API calls, backend integration | "Fetch data", "Call API", "Save to server" | `development/api-integration.md` | `patterns/state-management.md` |
| **Styling** | CSS, styling, theming | "Style component", "Change colors" | `styling/styling-guide.md` | `styling/theme-management.md` |
| **Routing** | Navigation, routes, links | "Add route", "Navigate to", "Create link" | `architecture/routing-patterns.md` | `patterns/state-management.md` |
| **Workflows** | Visual workflow editor, React Flow | "Edit workflow", "Add node", "Connect states" | `development/workflow-editor-patterns.md` | `patterns/state-management.md` |
| **Onboarding** | Onboarding flows, types, store, UI entry points | "Add onboarding flow", "Change triggers", "Add welcome step" | `.codemie/onboarding-flows/FLOW-CREATION-GUIDE.md` | `src/types/onboarding.ts` |
| **Errors** | Error handling, validation, error states | "Handle errors", "Show error", "Validate input" | `development/error-handling-patterns.md` | `patterns/form-patterns.md` |
| **Constants** | Extract magic values, configuration | "Remove magic numbers", "Extract constants" | `development/constants-usage.md` | `development/code-organization.md` |
| **Custom Hooks** | Extract logic, create reusable hooks | "Create hook", "Extract logic", "Reuse code" | `patterns/custom-hooks.md` | `components/component-patterns.md` |
| **Refactoring** | Split components, clean up code | "Split component", "Refactor", "Clean up" | `development/refactoring-patterns.md` | `components/component-organization.md` |
| **Accessibility** | ARIA, keyboard nav, screen readers | "Make accessible", "Add ARIA", "Keyboard support" | `patterns/accessibility-patterns.md` | `components/component-patterns.md` |
| **Performance** | Optimization, memoization, lazy loading | "Optimize", "Speed up", "Reduce re-renders" | `development/performance-patterns.md` | `patterns/custom-hooks.md` |
| **Testing** | Write tests, fix tests | "Test component", "Write test", "Fix failing test" | `testing/testing-patterns.md` | `components/component-patterns.md` |
| **Code Review** | AI code review with verifiable commit markers | "Do code review", "Review my changes", "Run code reviewer" | `.claude/agents/code-reviewer.md` | N/A |

**Guide Path**: All guides in `.codemie/guides/<category>/` (Code Review: `.claude/skills/code-reviewer/SKILL.md`)

**How to Use**:
1. Read user's request and understand **what they want to accomplish** (intent)
2. Map intent to category above
3. Load P0 guide (required for that intent)
4. If still unclear, load P1 guide (optional)
5. Apply patterns from guides

**Complexity Guide**:
- **Simple**: 1-2 files, < 5 min, obvious pattern → Use direct tools (Read/Edit)
- **Medium**: 2-5 files, 5-15 min, standard patterns → Use guides + direct tools

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codemie-ai/codemie-ui](https://github.com/codemie-ai/codemie-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
