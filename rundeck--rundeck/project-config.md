---
trigger: always_on
description: These best practices should be applied:
---

These best practices should be applied:

# Product Naming Guidelines

Always use the correct product names in descriptions, documentation, and API specifications:

- **Open Source Product**: "Rundeck"
- **Commercial Product**: "Runbook Automation"
- **Combined Reference**: "Rundeck / Runbook Automation" (preferred for API titles, descriptions, and user-facing text)

## Examples:
- ✅ **Correct**: "Rundeck / Runbook Automation API", "Rundeck / Runbook Automation Server"
- ❌ **Incorrect**: "Rundeck Pro API", "Rundeck Pro Server"

## Technical Elements (keep as-is):
- API headers: `X-Rundeck-Auth-Token` (maintain for compatibility)
- Internal identifiers: `rundeckApiToken`, `rundeckAuth` (technical references)
- Documentation URLs: `docs.rundeck.com` (legitimate technical URLs)

## API Documentation Specifics

### API Endpoint Descriptions:
- No need to include product names ("Rundeck" or "Runbook Automation") in API endpoint descriptions
- API consumers are already using the product API - product names are redundant and create unnecessary noise
- Focus descriptions on functionality, authorization requirements, and behavior
- Example: ✅ "Creates a native/local user with a password for use with the built-in authentication system"
- Example: ❌ "Creates a native/local Rundeck user..." or "Creates a Runbook Automation user..."

### Commercial-Only Features:
- Mark commercial-only endpoints with `[Enterprise]` tag in the summary
- Example: `summary = 'Create A Local User [Enterprise]'`

This ensures consistent branding across all user-facing elements while maintaining technical compatibility.

# All Java and Groovy Code (general instructions)

New or modified code should include appropriate Javadoc or Groovydoc comments. The comments should explain the purposes of the classes, methods, etc.

# Groovy code

Groovy code should use the `@CompileStatic` (or `@GrailsCompileStatic` for Grails aretefacts like Controllers, Services, etc.) on all classes.  
If some methods of a class require using Groovy dynamic typing, the `@CompileDynamic` annotation can be applied to those methods, but it is preferable if possible to convert them to use static typing.
If a class was already not using `@CompileStatic`, and the PR only changes/adds a few methods, then the `@CompileDynamic` should be added to those methods where possible.

Groovy classes do not need to explicitly implement getters and setters, like Java, so avoid that if possible.

# Testing

Changes to most groovy and Java code should be accompanied by updated or added Unit tests using the Spock testing framework.

If existing Junit tests need to be modified, it is preferred to convert them into Spock tests if possible.

Changes to front-end code (javascript, and typescript) should have accompanying Jest unit tests.

Changes that add or significantly change features of the application user interface, should have Selenium based tests added in the pro-functional-test module.

## Selenium Testing Guidelines

Selenium tests must follow the Page Object Model pattern and these strict guidelines:

### Wait Strategies
- **AVOID using `Thread.sleep()`** - Prefer explicit waits when possible. Only use `Thread.sleep()` with `WaitingTime` constants for special cases like external system initialization where explicit waits cannot be used.
- **Implicit waits are globally configured** - The framework sets a global implicit wait in the `BasePage` constructor and uses `implicitlyWait(2000)` in `go()` methods. This is an established pattern. However, for specific element interactions, **ALWAYS prefer explicit waits** to ensure reliability and avoid unpredictable behavior.
- **ALWAYS use explicit waits** provided by the Page Object base classes:
  - `waitForElementVisible()` - Wait for element to be present AND visible
  - `waitForElementToBeClickable()` - Wait for element to be visible AND enabled
  - `waitForElementAttributeToChange()` - Wait for specific attribute changes
  - `waitForNumberOfElementsToBeMoreThan()` - Wait for multiple elements to be present
  - `waitForNumberOfElementsToBe()` - Wait for the number of elements to match a specific count
  - `waitForTextToBePresentInElement()` - Wait for specific text to appear in an element
  - `waitForUrlToContain()` - Wait for the current URL to contain a specific substring
  - `waitForAttributeContains()` - Wait for an element's attribute to contain a specific value
  - `waitIgnoringForElementVisible()` - Wait for element to be visible, ignoring certain exceptions
  - `waitIgnoringForElementToBeClickable()` - Wait for element to be clickable, ignoring certain exceptions

### Page Object Model
- **NEVER put CSS/XPath selectors directly in test files** - All selectors belong in Page Object classes
- **Define selectors as `By` fields** at the top of Page Object classes:
  ```groovy
  By nodeDetailsTableBy = By.cssSelector(".popover-content .node-details-simple")
  By parameterKeyBy = By.cssSelector(".key")
  ```
- **Create getter methods** that include appropriate waits:
  ```groovy
  WebElement getNodeDetailsTable() {
      waitForPopoverToAppear()
      waitForElementVisible nodeDetailsTableBy
      el nodeDetailsTableBy
  }
  ```
- **Encapsulate interactions** - Complex operations should be methods in Page Objects, not inline in tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rundeck/rundeck](https://github.com/rundeck/rundeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
