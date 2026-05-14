---
trigger: always_on
description: **Purpose:** Ensures all documentation is accurate, complete, and synchronized with actual codebase. Prevents documentation drift, validates code examples, and maintains high-quality user experience through reliable documentation.
---

# Documentation Rules - Accuracy & Completeness

**Purpose:** Ensures all documentation is accurate, complete, and synchronized with actual codebase. Prevents documentation drift, validates code examples, and maintains high-quality user experience through reliable documentation.

**When to use:** All documentation writing, code example creation, API documentation, user guides, and any content that references code functionality.

## Code Snippet Accuracy

**Rule: Every code snippet in documentation must be verified to compile and work with the current codebase.**
Why: Broken documentation examples frustrate users, waste developer time, and damage project credibility. Accurate examples ensure users can successfully follow documentation.

**Code Snippet Validation Process:**
```bash
# 1. Extract all code snippets from documentation
find docs/ -name "*.md" -exec grep -l "```rust" {} \;

# 2. Create temporary test files for each snippet
# 3. Verify compilation with current dependencies
cargo check --manifest-path snippet_test/Cargo.toml

# 4. Run doc tests to verify examples work
cargo test --doc --all-features

# 5. Integration test with actual codebase APIs
cargo test --test doc_example_validation
```

**Snippet Requirements:**
- **Must Compile**: Every ````rust` block must compile successfully
- **Must Execute**: Examples with `assert!` must run and pass
- **Current Dependencies**: Use exact versions from project Cargo.toml
- **Complete Context**: Include necessary imports and setup code
- **Error Handling**: Show proper error handling patterns

**Example of Correct Code Snippet:**
```rust
/// # Examples
/// ```
/// use my_crate::{UserManager, CreateUserRequest, UserError};
/// 
/// # tokio_test::block_on(async {
/// let manager = UserManager::new().await?;
/// let request = CreateUserRequest {
///     email: "test@example.com".to_string(),
///     name: "Test User".to_string(),
/// };
/// 
/// let user = manager.create_user(request).await?;
/// assert_eq!(user.email(), "test@example.com");
/// # Ok::<(), UserError>(())
/// # });
/// ```
```

## API Documentation Synchronization

**Rule: API documentation must exactly match the current function signatures, types, and behavior.**
Why: Incorrect API documentation leads to integration failures, confusion, and wasted development time. Documentation must be the authoritative source of truth.

**API Sync Verification:**
```bash
# Generate documentation and check for warnings
cargo doc --all-features --no-deps 2>&1 | grep -i "warning\|error"

# Verify no broken intra-doc links
cargo doc --all-features --no-deps --document-private-items

# Check that public API documentation exists
cargo doc --all-features --document-private-items --open
# Manually verify all public items have documentation
```

**Required Documentation Elements:**
```rust
/// Brief description of what the function does.
///
/// Longer description explaining the purpose, use cases, and any important
/// implementation details that affect usage.
///
/// # Arguments
/// * `param1` - Description of first parameter, including type constraints
/// * `param2` - Description of second parameter and expected values
///
/// # Returns
/// Description of return value, including all possible variants for Result types:
/// - `Ok(User)` - Successfully created user with validated data
/// - `Err(UserError::InvalidEmail)` - Email format validation failed
/// - `Err(UserError::DatabaseError)` - Database operation failed
///
/// # Errors
/// This function will return an error if:
/// - Email format is invalid (checked against RFC 5322)
/// - Database connection fails or times out
/// - User with email already exists in system
///
/// # Panics
/// This function panics if:
/// - Internal invariants are violated (should never happen in normal usage)
/// - System resources are exhausted (extremely rare)
///
/// # Examples
/// ```
/// # use my_crate::*;
/// # tokio_test::block_on(async {
/// let user = create_user("valid@example.com", "John Doe").await?;
/// assert_eq!(user.email(), "valid@example.com");
/// # Ok::<(), UserError>(())
/// # });
/// ```
///
/// # Safety
/// This function is thread-safe and can be called concurrently.
/// 
/// # Performance
/// Expected execution time: <5ms for standard inputs.
/// Memory usage: O(1) relative to input size.
pub async fn create_user(email: &str, name: &str) -> Result<User, UserError> {
    // Implementation
}
```

## Documentation Completeness Verification

**Rule: All public APIs, modules, and user-facing features must have complete documentation.**
Why: Incomplete documentation creates knowledge gaps, forces users to read source code, and reduces project adoption and usability.

**Completeness Checklist:**
```markdown
## Public API Documentation Audit

### Crate Level
- [ ] lib.rs has comprehensive module documentation
- [ ] README.md covers installation, basic usage, and key features
- [ ] CHANGELOG.md documents all user-facing changes
- [ ] Cargo.toml has accurate description and keywords

### Module Level  
- [ ] Every public module has module documentation (//!)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
