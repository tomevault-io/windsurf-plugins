---
trigger: always_on
description: | Scenario | Action Required |
---

## Quick Reference: Testing Do's and Don'ts

| Scenario | Action Required |
|----------|----------------|
| Test fails due to my code changes | ⚠️ STOP - Inform user, await approval |
| Need to add new feature tests | ✅ Proceed - Add comprehensive tests |
| Test has typo in description | ✅ Proceed - Fix typo |
| Test expectation wrong due to bug fix | ⚠️ STOP - This reveals breaking change, inform user |
| Environment setup fails tests | ✅ Proceed - Fix environment/configuration |

## Testing Guidelines and Principles

### Core Testing Principles

**Test Preservation Policy**: Tests should never be modified when failing due to breaking changes. Test failures reveal important information about breaking changes that need user attention.

**User Approval Requirement**: Explicit user approval is required before modifying any test behavior, expectations, or logic.

**Breaking Change Protocol**: When code changes cause test failures, this indicates a potential breaking change that requires user awareness and approval.

### Test Modification Decision Tree

#### ✅ ALLOWED without explicit approval:
- **Typo fixes** in test descriptions or comments
- **Formatting improvements** (whitespace, indentation)
- **Add new tests** for new functionality
- **Fix test infrastructure** (imports, setup, environment)
- **Environment configuration** fixes (paths, variables, setup)

#### ❌ REQUIRES explicit user approval:
- **Change test expectations** (assertions, expected values)
- **Remove existing tests** (unless clearly obsolete)
- **Modify test logic** (conditions, loops, data)
- **Update test data** that changes expected behavior
- **Fix test failures** caused by code changes
- **Change test conditions** or validation criteria

### Test Failure Protocol

#### When tests fail due to code changes:
1. **STOP** - Do not modify tests
2. **ANALYZE** - Determine if failure indicates:
   - Genuine breaking change that needs user attention
   - Test implementation issue that needs user approval
   - Environment/configuration problem
3. **INFORM USER** - Clearly explain:
   - Why tests are failing
   - Impact of the code changes
   - Options for resolution
4. **AWAIT APPROVAL** - Do not proceed without explicit user consent

#### Communication Template:
```markdown
## Test Failure Alert

**Issue**: [Brief description of test failures]
**Cause**: Tests are failing due to code changes in [file/module]
**Impact**: [Describe what this breaking change affects]

**Options**:
1. **Approve test modification** - Update tests to match new behavior (confirms breaking change)
2. **Fix the code** - Modify implementation to maintain existing test expectations
3. **Document breaking change** - Update documentation to reflect new behavior

Please let me know which approach you'd prefer.
```

### Acceptable vs Unacceptable Test Modifications

#### ✅ Acceptable Test Modifications (Examples)
```typescript
// ✅ OK: Fix import path after file reorganization
import { safeValidate } from '../../utils/validationUtils.js';

// ✅ OK: Add new test for new feature
it('should handle new parameter validation', () => {
  const result = validateNewParameter({ valid: true });
  assert.strictEqual(result.success, true);
});

// ✅ OK: Fix typo in test description
it('should validate attribute creation', () => { // Fixed typo

// ✅ OK: Improve formatting for readability
const testData = {
  title: 'Test Note',
  type: 'text',
  content: 'Test content'
};
```

#### ❌ Unacceptable Test Modifications (Examples)
```typescript
// ❌ NOT OK: Change expected result due to code behavior change
assert.strictEqual(result.success, true); // Changed from false

// ❌ NOT OK: Remove validation test because it fails
// Removed: it('should reject invalid parameters', () => {...})

// ❌ NOT OK: Modify test data to pass failing tests
const testData = {
  // Changed from 'invalid' to 'valid' to make test pass
  status: 'valid'
};

// ❌ NOT OK: Update assertion to match new (broken) behavior
assert.ok(result.includes('new message')); // Changed from expected message
```

### Example: Test Reorganization (User-Approved)
The recent reorganization of validation tests from a 449-line monolithic file into 8 focused test files was performed with explicit user approval and serves as an example of major test restructuring that follows the proper protocol.

## Development Workflow

**Standard Development Process**:
1. **Edit Code** - Implement new features or fix bugs in source files
2. **Write Tests** - Add comprehensive tests for new functionality
3. **Update Documentation** - Sync documentation with code changes
4. **Build & Validate** - Run `npm run build` to ensure TypeScript compilation success

**Workflow Guidelines**:
- **Code First**: Focus on implementing the core functionality in TypeScript
- **Test Coverage**: Add tests that cover all new functionality, edge cases, and error conditions
- **Documentation Sync**: Update CLAUDE.md, tool descriptions, and any relevant documentation
- **Build Validation**: Always run `npm run build` to catch TypeScript errors before committing
- **Iterative Development**: Repeat the cycle for each feature or bug fix

**Quality Checks**:
- TypeScript compilation must pass (no errors)
- New functionality must have corresponding tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tan-yong-sheng/triliumnext-mcp](https://github.com/tan-yong-sheng/triliumnext-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
