---
trigger: always_on
description: Testing guidelines for NOWCRM
---


# Testing Guidelines for NOWCRM

## Overview

NOWCRM uses **Playwright** for end-to-end (E2E) testing. All tests follow the **Page Object Model (POM)** pattern for maintainability and reusability.

## Test Structure

### Directory Organization

```
apps/nowcrm/tests/
├── *.spec.ts              # Test specification files (numbered for execution order)
├── pages/                  # Page Object Models (POMs)
│   ├── CommonPage.ts
│   ├── ContactsListPage.ts
│   └── ...
├── utils/                  # Test utilities and helpers
│   ├── authHelper.ts
│   └── data.ts
├── setup/                  # Setup and teardown scripts
│   ├── global-setup.ts
│   ├── create-users.ts
│   └── delete-users.ts
└── files/                  # Test fixtures and data files
```

### File Naming Conventions

- **Test files**: Use numbered prefixes for execution order (e.g., `01Authentication.spec.ts`, `02Contacts.spec.ts`)
- **Page Objects**: Use descriptive names ending with `Page` or `Modal` (e.g., `ContactsListPage.ts`, `ContactCreateModal.ts`)
- **Utilities**: Use descriptive names (e.g., `authHelper.ts`, `data.ts`)

## Page Object Model (POM) Pattern

### Structure

Every Page Object should follow this structure:

```typescript
import { type Locator, type Page, expect } from '@playwright/test';

export class PageName {
    readonly page: Page;
    
    // Locators - declare as readonly
    readonly elementName: Locator;
    
    constructor(page: Page) {
        this.page = page;
        // Initialize locators
        this.elementName = page.getByRole('button', { name: 'Button Name' });
    }
    
    // Actions - async methods that perform interactions
    async performAction() {
        await expect(this.elementName).toBeVisible();
        await this.elementName.click();
    }
    
    // Assertions - async methods that verify state
    async expectSomethingVisible(timeout: number = 5000) {
        await expect(this.elementName, 'Descriptive message').toBeVisible({ timeout });
    }
}
```

### Locator Best Practices

1. **Prefer role-based selectors**:
```typescript
// ✅ Good - accessible and stable
this.createButton = page.getByRole('button', { name: 'Create' });
this.emailInput = page.getByRole('textbox', { name: 'Email' });

// ❌ Avoid - fragile CSS selectors
this.createButton = page.locator('.btn-primary');
```

2. **Scope locators within dialogs/modals**:
```typescript
constructor(page: Page) {
    this.dialog = page.getByRole('dialog', { name: /Create Contact/i });
    // Scope inputs within dialog
    this.firstNameInput = this.dialog.getByRole('textbox', { name: 'First name' });
}
```

3. **Use descriptive locator names**:
```typescript
// ✅ Good
readonly userMenuTrigger: Locator;
readonly deleteMassActionMenuItem: Locator;

// ❌ Avoid
readonly btn1: Locator;
readonly menuItem: Locator;
```

### Action Methods

- **Naming**: Use verb phrases (e.g., `clickCreateButton`, `fillAndSubmit`, `openUserMenu`)
- **Wait for visibility**: Always wait for elements before interacting
- **Return values**: Return relevant data when needed (e.g., created entity ID)

```typescript
async clickCreateButton() {
    await expect(this.createButton, 'Create button should be visible').toBeVisible({ timeout: 20000 });
    await this.createButton.click();
}

async fillAndSubmit(data: ContactData) {
    await this.firstNameInput.fill(data.firstName);
    await this.lastNameInput.fill(data.lastName);
    await this.emailInput.fill(data.email);
    await this.createButton.click();
}
```

### Assertion Methods

- **Naming**: Prefix with `expect` (e.g., `expectDashboardVisible`, `expectStatusMessage`)
- **Descriptive messages**: Always include meaningful error messages
- **Configurable timeouts**: Accept timeout parameters with sensible defaults

```typescript
async expectStatusMessage(message: string, timeout: number = 20000) {
    const messageLocator = this.page.getByText(message, { exact: true });
    await expect(messageLocator, `Status message "${message}" should be visible`)
        .toBeVisible({ timeout });
}

async expectDashboardVisible(timeout: number = 10000) {
    await expect(this.page, 'URL should indicate CRM dashboard')
        .toHaveURL(/\/crm$/, { timeout });
}
```

## Test File Structure

### Basic Template

```typescript
import { test, expect } from '@playwright/test';
import { faker } from '@faker-js/faker';

// Import Page Object Models
import { ContactsListPage } from './pages/ContactsListPage';
import { ContactCreateModal } from './pages/ContactCreateModal';

// Import utilities
import { loginUser } from './utils/authHelper';

test.describe('Feature Name', () => {
    let pageObject1: ContactsListPage;
    let pageObject2: ContactCreateModal;

    test.beforeEach(async ({ page }) => {
        // Initialize POMs
        pageObject1 = new ContactsListPage(page);
        pageObject2 = new ContactCreateModal(page);
        
        // Common setup (e.g., login)
        await loginUser(page);
        await pageObject1.goto();
    });

    test('User can perform action', async () => {
        // Arrange - set up test data
        const testData = { 
            firstName: faker.person.firstName(), 
            email: faker.internet.email() 
        };
        
        // Act - perform actions
        await pageObject1.clickCreateButton();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nowtec/nowCRM](https://github.com/nowtec/nowCRM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
