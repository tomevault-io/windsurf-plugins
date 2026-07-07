---
trigger: always_on
description: 1. **Test Coverage is Critical**: Higher coverage creates more confidence and helps identify bugs effectively.
---

# MetaMask Mobile E2E Testing Guidelines

## Core Principles

1. **Test Coverage is Critical**: Higher coverage creates more confidence and helps identify bugs effectively.
2. **Tests Should Be Reliable**: Tests should consistently produce the same results and be resilient to minor system changes.
3. **Tests Should Provide Fast Feedback**: Optimize for quick execution and clear failure messages.
4. **Tests Should Be Easy to Debug**: When a test fails, it should be clear what functionality is broken.
5. **Tests Should Be Maintainable**: Structure tests for easy maintenance as the application evolves.

## Test Naming Conventions

### DO:
- Use clear, descriptive names that communicate the purpose of the test
- Name tests based on what they verify (e.g., `adds Bob to the address book`)
- Keep names concise but informative

### DON'T:
- Use the prefix 'should' (e.g., `should add Bob to the address book`)
- Include multiple behaviors with 'and' in a single test name
- Use vague or generic names

## Test Organization - MANDATORY

- Organize tests into folders based on features and scenarios
- Each feature team should own one or more folders of tests
- Follow the same organization pattern as the extension team for consistency
- Place tests in logical feature directories:
  ```
  e2e/specs/<feature-name>/<e2e-test-name.spec.ts>
  e2e/specs/tokens/import/import-erc1155.spec.ts
  e2e/specs/settings/clear-activity.spec.ts
  e2e/specs/ppom/ppom-blockaid-alert-erc20-approval.spec.ts
  ```

## Framework Architecture

### Core Classes:

- **`Assertions`** - Enhanced assertions with auto-retry and detailed error messages
- **`Gestures`** - Robust user interactions with configurable element state checking
- **`Matchers`** - Type-safe element selectors with flexible options
- **`Utilities`** - Core utilities with specialized element state checking

### Key Features:

- ✅ **Auto-retry** - Handles flaky network/UI conditions
- ✅ **Configurable element state checking** - Control visibility, enabled, and stability checks per interaction
- ✅ **Performance optimization** - Stability checking disabled by default for better performance
- ✅ **Better error messages** - Descriptive errors with retry context and timing
- ✅ **Type safety** - Full TypeScript support with IntelliSense

## Test Atomicity and Coupling

### When to Isolate Tests:
- Testing specific functionality of a single component or feature
- When you need to pinpoint exact failure causes
- For basic unit-level behaviors

### When to Combine Tests:
- For multi-step user flows that represent real user behavior
- When testing how different parts of the application work together
- When the setup for multiple tests is time-consuming and identical

### Guidelines:
- Each test should run with a dedicated browser and mock services
- Use the `withFixtures` function to create test prerequisites and clean up afterward
- Avoid shared mocks and services between tests when possible
- Consider the "fail-fast" philosophy - if an initial step fails, subsequent steps may not need to run

## Controlling State

### Best Practices:
- Control application state programmatically rather than through UI interactions
- Use fixtures to set up test prerequisites instead of UI steps
- Minimize UI interactions to reduce potential breaking points
- Improve test stability by reducing timing and synchronization issues

### Example:
```javascript
// GOOD: Use fixture to set up prerequisites
new FixtureBuilder()
  .withAddressBookControllerContactBob()
  .withTokensControllerERC20()
  .build();

// Then test only the essential steps:
// Login
// Send TST
// Assertion

// BAD: Building all state through UI
new FixtureBuilder().build();
// Login
// Add Contact
// Open test dapp
// Connect to test dapp
// Deploy TST
// Add TST to wallet
// Send TST
// Assertion
```

## Framework Best Practices

### Page Object Model (POM) Pattern
- ALWAYS use the Page Object Model pattern for organizing test code
- Move all element selectors to Page Objects or dedicated selector files
- Access UI elements through Page Object methods, not directly in test specs

#### Page Object Structure Example:
```typescript
import { LoginPageSelectors } from './LoginPage.selectors';

class LoginPage {
  // Getter pattern for elements
  get emailInput() {
    return Matchers.getElementByID(LoginPageSelectors.EMAIL_INPUT);
  }
  get passwordInput() {
    return Matchers.getElementByID(LoginPageSelectors.PASSWORD_INPUT);
  }
  get loginButton() {
    return Matchers.getElementByID(LoginPageSelectors.LOGIN_BUTTON);
  }

  // Public methods for actions
  async login(email: string, password: string): Promise<void> {
    await Gestures.typeText(this.emailInput, email, {
      description: 'enter email',
    });
    await Gestures.typeText(this.passwordInput, password, {
      description: 'enter password',
    });
    await Gestures.tap(this.loginButton, { description: 'tap login button' });
  }

  // Public methods for verifications
  async verifyLoginError(expectedError: string): Promise<void> {
    await Assertions.expectTextDisplayed(expectedError, {
      description: 'login error should be displayed',
    });
  }
}

export default new LoginPage();
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [granjacours410-source/supreme-disco](https://github.com/granjacours410-source/supreme-disco) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
