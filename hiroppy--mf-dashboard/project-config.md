---
trigger: always_on
description: The following rules must always be followed. Code violating these rules will be rejected in review.
---

# Claude Code Development Guidelines

## Mandatory Rules (MUST)

The following rules must always be followed. Code violating these rules will be rejected in review.

### DB Schema

- [ ] All tables MUST have `createdAt: text("created_at").notNull()`
- [ ] All tables MUST have `updatedAt: text("updated_at").notNull()`
- [ ] Foreign keys MUST specify `onDelete` (cascade/set null)
- [ ] When adding/modifying DB schema, MUST update `docs/architecture/database-schema.md` to reflect current schema structure

### Component Creation

- [ ] All components under `components/` MUST have `*.stories.tsx`
- [ ] Chart-related: separate into `charts/` (pure UI) and `info/` (data fetching)
- [ ] `info/` components: Server Component (data fetching) + `.client.tsx` ONLY if interactivity is required

### Testing

- [ ] New logic MUST have unit tests
- [ ] After writing Storybook, verify with `test:storybook`

### Code Structure

- [ ] Do NOT create barrel files (`index.ts` for re-exports)
- [ ] Import directly from the source file

### Logging (Crawler)

For the crawler (`apps/crawler`), use the following log functions appropriately:

- [ ] Use `info()` for important information that should be visible in CI environments (GitHub Actions)
- [ ] Use `log()` for detailed logs during local development (hidden in CI environments)
- [ ] Use `debug()` for debug information (visible when `DEBUG=true`)

**Log Level Guidelines:**

| Function  | Local | CI Env | Purpose                                        |
| --------- | ----- | ------ | ---------------------------------------------- |
| `info()`  | ✅    | ✅     | Important progress info, must be tracked in CI |
| `log()`   | ✅    | ❌     | Detailed debug info, local development only    |
| `debug()` | ⚙️    | ❌     | Debug mode only (enabled with `DEBUG=true`)    |
| `warn()`  | ✅    | ✅     | Warnings                                       |
| `error()` | ✅    | ✅     | Errors                                         |

(⚙️ = Enabled with `DEBUG=true`)

**Example:**

```typescript
import { info, log, warn } from "./logger.js";

// Important information that should be visible in CI
info("Refreshing accounts...");
info(`[${elapsed}s] Remaining: ${updatingCount}`);

// Detailed logs for local development (hidden in CI)
log("Navigating to accounts page...");
log(`Found ${count} rows in the table`);

// Warnings (always visible)
warn("Max wait time exceeded");
```

### Personal Information (Strictly Prohibited)

**NEVER include personally identifiable information (PII) in test data, Storybook, comments, or documentation.**

- [ ] Do NOT use real names or nicknames → Use "User A", "User B", "Group A", etc.
- [ ] Do NOT use real addresses, phone numbers, or email addresses
- [ ] Do NOT use real account numbers or card numbers
- [ ] Do NOT include personal names in code comments

**Use anonymous placeholders:**

| NG                     | OK                 |
| ---------------------- | ------------------ |
| Real person's account  | User B's account   |
| Named group            | Group A            |
| Real name              | Test User          |
| real-email@example.com | user-a@example.com |

### Semantic Colors (Strictly Enforced)

Always use these classes for monetary values. Custom color definitions are prohibited.
See `apps/web/src/app/globals.css` for CSS variable definitions.

| Purpose          | Class                   |
| ---------------- | ----------------------- |
| Income amount    | `text-income`           |
| Expense amount   | `text-expense`          |
| Positive balance | `text-balance-positive` |
| Negative balance | `text-balance-negative` |

**Usage Guidelines:**

- `text-income` / `text-expense` — Actual amounts (income, expenses)
- `text-balance-positive` / `text-balance-negative` — Differences or evaluations (balance, unrealized gains/losses)

**Decision Tree:**

"Did money actually move?"

- **Yes** → `text-income` / `text-expense`
- **No (comparison/evaluation/change)** → `text-balance-positive` / `text-balance-negative`

**Specific Use Cases:**

| Use Case                     | Correct Class           | Reason                          |
| ---------------------------- | ----------------------- | ------------------------------- |
| Food expense ¥5,000          | `text-expense`          | Actual expense                  |
| Salary ¥300,000              | `text-income`           | Actual income                   |
| Day-over-day +¥10,000        | `text-balance-positive` | Comparison difference           |
| Day-over-day -¥5,000         | `text-balance-negative` | Comparison difference           |
| Unrealized gain +¥50,000     | `text-balance-positive` | Valuation gain/loss             |
| Unrealized loss -¥30,000     | `text-balance-negative` | Valuation gain/loss             |
| Monthly balance +¥20,000     | `text-balance-positive` | Income minus expense difference |
| Monthly balance -¥10,000     | `text-balance-negative` | Income minus expense difference |
| Liability balance ¥1,000,000 | `text-balance-negative` | Negative item on balance sheet  |
| Asset change +¥100,000       | `text-balance-positive` | Period-over-period change       |

**Common Mistakes:**

```tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hiroppy/mf-dashboard](https://github.com/hiroppy/mf-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
