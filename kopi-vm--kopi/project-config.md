---
trigger: always_on
description: All documentation output in this project must be written in English, including:
---

# AGENTS.md

## Documentation Language Policy

All documentation output in this project must be written in English, including:

- Code comments
- Commit messages
- Architecture Decision Records (ADRs)
- README files
- API documentation
- Error messages
- User-facing documentation
- Test descriptions
- TODO comments
- Any other written documentation

## Project Overview

Kopi is a JDK version management tool written in Rust that integrates with your shell to seamlessly switch between different Java Development Kit versions. It fetches JDK metadata from foojay.io and provides a simple, fast interface similar to tools like volta, nvm, and pyenv.

Key features:

- Automatic JDK version switching based on project configuration
- Multiple JDK vendor support (AdoptOpenJDK, Amazon Corretto, Azul Zulu, etc.)
- Shell integration via shims for transparent version management
- Project-specific JDK pinning via `.kopi-version` or `.java-version` files
- Fast performance using Rust

### User Documentation

The user-facing documentation for Kopi is maintained in a separate repository at `../kopi-vm.github.io/`. This repository uses MkDocs to generate and publish documentation for end users.

## Agent Operating Environment

### Planning and Tooling Discipline

- Use the planning tool for any task that is not trivially simple; plans must include at least two steps and be updated as work progresses. Skip planning only for the simplest \~25% of tasks.
- At the start of each session, run `serena__activate_project` to bind Serena to the current repository context.
- After activation, run `serena__check_onboarding_performed`; if it indicates onboarding is incomplete, execute `serena__onboarding` before proceeding.
- At the start of each task, run `bun scripts/trace-status.ts --write` to regenerate `docs/traceability.md` and identify any traceability gaps before continuing.
- Before insert/replace/delete edits, invoke `serena__think_about_task_adherence`; after gathering significant context, call `serena__think_about_collected_information` to confirm understanding.
- Follow the Traceable Development Lifecycle (TDL) for non-trivial work and ensure traceability artifacts stay in sync.

### Approvals and Safety

- Exercise caution with destructive operations (`rm`, `git reset`, etc.); seek explicit user direction before proceeding.
- If unexpected repository changes appear, pause and confirm next steps with the user.

## Developer Principles

### Memory Safety Over Micro-optimization

- Prioritize memory safety and correctness over micro-optimizations
- Accept reasonable overhead (e.g., cloning small strings) to avoid memory leaks
- Follow Rust's ownership model strictly - avoid `unsafe` code and memory leaks from techniques like `Box::leak()`
- When faced with lifetime complexity, prefer simpler solutions that may use slightly more memory but are correct
- Example: Clone strings for HTTP headers instead of using `Box::leak()` to create static references

### Code Clarity

- Write clear, readable code that is easy to understand and maintain
- Use descriptive variable and function names
- Add comments for complex logic, but prefer self-documenting code
- Structure code to minimize cognitive load for future developers

### Clean Code Maintenance

- Remove unused variables, parameters, and struct members promptly
- When refactoring, trace through all callers to eliminate unnecessary parameters
- Keep structs lean by removing fields that are no longer used
- Use `cargo clippy` to identify unused code elements
- Example: If a function parameter like `arch` is no longer used in the implementation, remove it from the function signature and update all callers

### Prefer Functions Over Structs Without State

- When there's no state to manage, prefer implementing functionality as standalone functions rather than defining structs
- Only create structs when you need to maintain state, implement traits, or group related data
- This keeps the code simpler and more straightforward
- Example: For utility operations like file validation or string parsing, use functions directly instead of creating a struct with methods

### External API Testing

- When writing code that calls external Web APIs, implement at least one unit test that includes the actual JSON response obtained from calling the API with curl
- Store the JSON response as a string within the test code
- This ensures that the parsing logic is tested against real API responses
- Example:

```rust
#[test]
fn test_parse_foojay_api_response() {
    // JSON response obtained from: curl https://api.foojay.io/disco/v3.0/packages?version=21
    let json_response = r#"{
        "result": [
            {
                "id": "abcd1234",
                "distribution": "temurin",
                "major_version": 21,
                ...
            }
        ]
    }"#;

    let packages: Vec<Package> = serde_json::from_str(json_response).unwrap();
    assert_eq!(packages[0].distribution, "temurin");
}
```

### Avoid Generic "Manager" Naming

- When the name "manager" appears in file names, structs, traits, or similar constructs, consider more specific and descriptive alternatives
- "Manager" is often too abstract and doesn't clearly communicate the responsibility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kopi-vm/kopi](https://github.com/kopi-vm/kopi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
