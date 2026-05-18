---
trigger: always_on
description: Testing
---

# Frontend Testing Best Practices

## Core Philosophy
- **Test behavior, not implementation**: Focus on what the user experiences, not how it's implemented
- **Refactor with confidence**: Tests should remain valid even when implementation details change
- **Test like a user**: Interact with components as a user would, not as a developer would

## React Testing Library Guidelines

### General Principles
- Prefer queries that reflect how users find elements (accessibility-focused)
- Avoid testing implementation details (state, props, component instances)
- Write tests that would pass regardless of the underlying implementation
- Focus on component behavior and user interactions
- Test what the component does, not what it is

### Query Priority (from most to least preferred)
1. **Accessible queries** (most preferred)
   - `getByRole` - Find by ARIA role (button, link, etc.) - **Use this most of the time!**
   - `getByLabelText` - Find form elements by their label
   - `getByPlaceholderText` - Find input by placeholder
   - `getByText` - Find elements by their text content
   - `getByDisplayValue` - Find form elements by their current value

2. **Test ID queries** (use only when necessary)
   - `getByTestId` - Find elements by data-testid attribute

3. **Avoid completely**
   - Direct DOM queries with `querySelector`
   - Querying by class or id attributes
   - Accessing component props or state directly

### Common Mistakes to Avoid
- **Using the wrong query** (high importance)
  - Prefer `getByRole` over other queries when possible
  - Query by actual text rather than test IDs when possible
  - Never use `container.querySelector` directly

- **Using query* variants incorrectly** (high importance)
  - Only use `queryBy*` for checking that elements don't exist
  - Use `getBy*` for elements that should be in the DOM
  - Use `findBy*` for elements that appear asynchronously

- **Not using screen** (medium importance)
  ```jsx
  // ❌
  const { getByRole } = render(<Example />)
  const button = getByRole('button')
  
  // ✅
  render(<Example />)
  const button = screen.getByRole('button')
  ```

- **Not using userEvent over fireEvent** (medium importance)
  ```jsx
  // ❌
  fireEvent.change(input, { target: { value: 'hello world' } })
  
  // ✅
  userEvent.type(input, 'hello world')
  ```

- **Using cleanup unnecessarily** (medium importance)
  - Cleanup happens automatically in most testing frameworks

- **Wrapping things in act unnecessarily** (medium importance)
  - `render` and `fireEvent` are already wrapped in `act`

### Assertion Best Practices
- Assert visible outcomes, not internal state
- Verify elements appear/disappear as expected
- Check text content changes appropriately
- Verify accessibility attributes are correct
- Test that appropriate ARIA attributes update
- Use jest-dom matchers for clearer assertions:
  ```jsx
  // ❌
  expect(element.textContent).toBe('hello')
  
  // ✅
  expect(element).toHaveTextContent('hello')
  ```

### User Interaction Testing
- Use `userEvent` over `fireEvent` when possible (more realistic)
- Test complete user workflows, not just isolated actions
- Simulate realistic user behavior (click, type, tab, etc.)
- Test keyboard navigation and accessibility
- Verify proper focus management

### Async Testing
- Use `findBy*` queries for elements that appear asynchronously
- Properly await async operations
- Test loading states and transitions
- Verify error states are handled correctly
- Use `waitFor` only when necessary and always wait for specific assertions:
  ```jsx
  // ❌
  await waitFor(() => {})
  expect(something).toBeTrue()
  
  // ✅
  await waitFor(() => expect(something).toBeTrue())
  ```

### Mocking
- Minimize mocks - prefer testing actual behavior
- Mock external dependencies only when necessary
- Use MSW (Mock Service Worker) for API mocking
- Keep mocks as close to real behavior as possible
- Reset mocks between tests

### Test Structure
- Arrange: Set up component with necessary props/context
- Act: Perform user interactions
- Assert: Verify the expected outcomes
- Follow the AAA (Arrange-Act-Assert) pattern
- Keep tests focused on a single behavior

### Component Testing Scope
- **Unit tests**: Test isolated components with mocked dependencies
- **Integration tests**: Test components working together
- **E2E-like tests**: Test complete user flows across multiple components
- Prefer integration tests over unit tests for most components
- Test complex user flows that span multiple components

### Form Testing
- Test form submissions and validations
- Verify error messages appear correctly
- Test field interactions (focus, blur, change)
- Verify form state updates correctly
- Test accessibility of form elements

### Routing Tests
- Test navigation between routes
- Verify correct components render for each route
- Test URL parameter handling
- Test protected routes and authentication flows
- Verify history manipulation works correctly

### Context and State Management
- Test how components interact with context
- Verify state changes reflect in the UI
- Test state transitions and side effects
- Avoid testing the state management library itself
- Focus on how state affects user experience

### Error Handling
- Test error boundaries
- Verify error states render correctly
- Test recovery from errors

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [waqarkalim/holiday-optimizer](https://github.com/waqarkalim/holiday-optimizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
