---
trigger: always_on
description: This is a **Playwright E2E testing framework** with **Cucumber BDD** integration, written in **TypeScript**. The framework provides a robust, maintainable, and scalable solution for end-to-end browser automation testing with comprehensive Allure reporting capabilities.
---

# GitHub Copilot Instructions for Playwright E2E Template

## Project Overview

This is a **Playwright E2E testing framework** with **Cucumber BDD** integration, written in **TypeScript**. The framework provides a robust, maintainable, and scalable solution for end-to-end browser automation testing with comprehensive Allure reporting capabilities.

## Tech Stack

- **Browser Automation**: Playwright v1.45.0 (Chromium, Firefox, WebKit support)
- **BDD Framework**: Cucumber v10.3.1 (Gherkin syntax)
- **Language**: TypeScript v5.3.3 (strict mode enabled)
- **Test Runner**: cucumber-js
- **Reporting**: Allure Reports v2.34.1 + Cucumber HTML/JSON/XML reports
- **Node Runtime**: ts-node v10.9.2

## Architecture & Design Patterns

### Page Object Model (POM) Pattern ⭐ PRIMARY PATTERN

- **BasePage** (`src/pages/base.page.ts`): Abstract base class with common page operations
- **Page Classes** (`src/pages/*.page.ts`): Concrete page classes extending BasePage
  - Elements declared as `protected` properties
  - Locators initialized in constructor
  - Public methods for business actions
  - Encapsulate all page-specific logic
- **Step Definitions** (`src/step-definitions/*.steps.ts`):
  - ONLY call methods from page objects
  - NO direct interaction with browser or elements
  - NO dependency on PageHelper for page interactions
  - Focus on orchestrating test flow

### Singleton Pattern

- **BrowserManager** (`src/config/browser.config.ts`) uses the Singleton pattern to ensure one browser instance per test scenario
- Access browser via `browserManager.getPage()` ONLY when initializing page objects

### Helper Pattern

- **AssertionHelper**: Custom assertion utilities for validation patterns
- **AllureHelper**: Allure report attachment and metadata management
- **PageHelper**: DEPRECATED - Use Page Object classes instead

### BDD Pattern

- Feature files in Gherkin syntax define test scenarios in human-readable format
- Step definitions implement test orchestration by calling page object methods
- Hooks manage test lifecycle (BeforeAll, Before, After, AfterAll)

## Project Structure

```
src/
├── config/                    # Configuration files
│   ├── browser.config.ts      # Browser manager (singleton)
│   └── test.config.ts         # Test environment config
├── pages/                     # Page Object Model classes ⭐ NEW
│   ├── base.page.ts           # Abstract base page class
│   ├── *.page.ts              # Concrete page classes
│   └── index.ts               # Page exports
├── features/                  # Gherkin feature files
│   └── *.feature              # BDD test scenarios
├── step-definitions/          # Step implementations
│   └── *.steps.ts             # Step definition files (call page methods)
├── hooks/                     # Test lifecycle hooks
│   └── hooks.ts               # BeforeAll, Before, After, AfterAll
└── support/                   # Helper utilities
    ├── assertion-helper.ts    # Custom assertions
    └── allure-helper.ts       # Allure reporting helpers
```

## Coding Guidelines

### When Writing Feature Files (.feature)

1. **Use Gherkin Syntax**: Feature, Scenario, Given, When, Then, And, But
2. **Add Tags**: Use tags for organization (@smoke, @regression, @critical, etc.)
3. **Be Descriptive**: Write clear, business-readable scenarios
4. **Use Background**: For common setup steps shared across scenarios
5. **Parameterize**: Use placeholders like `{string}` for dynamic values

**Example:**

```gherkin
@smoke @regression
Feature: User Authentication
  As a user
  I want to log in to the application
  So that I can access protected features

  Background:
    Given I navigate to the login page

  @critical
  Scenario: Successful login with valid credentials
    When I enter username "user@example.com"
    And I enter password "SecurePass123"
    And I click the login button
    Then I should see the dashboard
    And I should see welcome message "Welcome back!"
```

### When Writing Step Definitions (\*.steps.ts)

1. **Import Required Decorators**: Use `Given`, `When`, `Then` from `@cucumber/cucumber`
2. **Import Page Objects**: Import page classes from `../pages`
3. **Initialize Page Objects**: Create page instance with `browserManager.getPage()`
4. **Call Page Methods**: Use page object methods instead of direct browser interaction
5. **Type Parameters**: Use TypeScript types for step parameters (string, number, etc.)
6. **Add Logging**: Include console.log for debugging and visibility
7. **Async/Await**: All step functions must be async
8. **Attach to Allure**: Use AllureHelper for screenshots, HTML snapshots, and metadata

**Example:**

```typescript
import { Given, When, Then } from "@cucumber/cucumber";
import { browserManager } from "../config/browser.config";
import { PlaywrightHomePage } from "../pages";
import { AssertionHelper } from "../support/assertion-helper";
import { AllureHelper } from "../support/allure-helper";

let playwrightHomePage: PlaywrightHomePage;

Given("I navigate to {string}", async function (url: string) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/karamata) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
