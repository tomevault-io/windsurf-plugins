---
trigger: always_on
description: elif.rs is "Rust Made Simple" - a web framework designed for exceptional developer experience and AI-native development. It emphasizes convention over configuration, zero boilerplate, and intuitive APIs.
---

# Cursor AI Rules for elif.rs Framework

## Project Context
elif.rs is "Rust Made Simple" - a web framework designed for exceptional developer experience and AI-native development. It emphasizes convention over configuration, zero boilerplate, and intuitive APIs.

## AI Assistant Guidelines

### 1. Response Style
- **Be concise and direct** - Skip unnecessary explanations
- **Focus on code, not commentary** - Let the code speak for itself
- **One problem, one solution** - Don't over-engineer or add unrelated improvements
- **Follow existing patterns** - Study the codebase before making changes
- **Ask clarifying questions** when requirements are unclear

### 2. Code Understanding Priority
1. **Read CLAUDE.md first** - Contains project-specific patterns and conventions
2. **Study existing implementations** - Look at similar features before implementing
3. **Check recent commits** - Understand current development patterns
4. **Look for TODO comments** - They often indicate intended improvements
5. **Examine test files** - They show expected behavior and usage patterns

### 3. Framework Philosophy (Critical for AI Understanding)
- **Convention Over Configuration**: Sensible defaults, minimal setup
- **Zero Boilerplate**: Simple, obvious APIs like `router()`, `response()`
- **AI-Friendly**: Code should be intuitive for both humans and AI
- **Pure Framework Types**: Never expose Axum, Hyper, etc. to end users
- **Spec-First**: Generate code from specifications

### 4. Code Quality Standards
- Use descriptive function/variable names that explain intent
- Prefer `Result<T, HttpError>` for HTTP operations
- Use `#[derive(Default)]` for controllers to support auto-registration
- Implement comprehensive error messages with helpful hints
- Include usage examples in macro documentation
- **Never add comments unless specifically requested**

### 5. Implementation Guidelines
- **Start small**: Implement minimal viable solution first
- **Test incrementally**: Add tests as you implement, not after
- **Use existing patterns**: Don't invent new ways to solve solved problems
- **Maintain backward compatibility**: Unless explicitly breaking change is needed
- **Consider AI generation**: Will future AI be able to understand and modify this code?

## Framework Patterns (Study These!)

### 1. Zero-Boilerplate Bootstrap
```rust
use elif::prelude::*;

#[elif::bootstrap]  // Fully automatic startup
async fn main() -> Result<(), HttpError> { Ok(()) }

#[elif::bootstrap(addr = "0.0.0.0:8080")]  // With parameters
async fn main() -> Result<(), HttpError> { Ok(()) }

#[elif::bootstrap(AppModule)]  // Backward compatible
async fn main() -> Result<(), HttpError> { Ok(()) }
```

### 2. Controller System (Declarative)
```rust
#[derive(Default)] // Required for auto-registration
#[controller(path = "/api/users")]
pub struct UserController;

impl UserController {
    #[get("/{id}")]
    pub async fn show(&self, id: u32) -> HttpResult<ElifResponse> {
        Ok(ElifResponse::ok().json(&format!("User {}", id))?)
    }
    
    #[post("")]
    #[middleware("auth")]
    pub async fn create(&self, req: ElifRequest) -> HttpResult<ElifResponse> {
        Ok(ElifResponse::created().json(&"Created")?)
    }
}
```

### 3. Response Patterns
```rust
// Simple responses
Ok(ElifResponse::ok().json(&data)?)
Ok(ElifResponse::created().json(&user)?)
Ok(ElifResponse::not_found().json("User not found")?)

// Error handling  
Err(HttpError::not_found("User not found"))
Err(HttpError::bad_request("Invalid data"))
```

## Critical AI Instructions

### 1. Before Writing Any Code
1. **Read the specific files involved** - Use Read tool to understand current implementation
2. **Search for similar patterns** - Use Grep/Task to find existing examples 
3. **Check recent changes** - Look at git commits to understand context
4. **Understand the error/issue first** - Don't jump to solutions immediately

### 2. Code Modification Rules
- **Edit existing files, don't create new ones** unless absolutely necessary
- **Follow exact indentation and formatting** of existing code
- **Use the same error handling patterns** as surrounding code
- **Match the existing variable naming style** in the file
- **Don't add extra features** - solve only the specific problem asked

### 3. Testing Strategy  
- **Write tests AFTER implementation** - Implement first, then test
- **Use existing test patterns** - Look at other test files in the same directory
- **Test both success and error cases** - Especially for macro code
- **Use trybuild for macro UI tests** - Critical for compile-time validation

### 4. Error Messages & Debugging
- **Study existing error formats** - Match the style and helpfulness level
- **Include specific context** - What operation failed and why
- **Suggest solutions** - Don't just report the problem
- **Use consistent error types** - `HttpError` for HTTP, `ElifError` for core

### 5. Macro Development (Special Rules)
- **Study the existing macro structure** before modifying
- **Use `cargo expand` to debug** - Understand what code is generated
- **Preserve backward compatibility** - Support both old and new syntax

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krcpa/elif.rs](https://github.com/krcpa/elif.rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
