---
trigger: always_on
description: **solsec** (Solana Smart Contract Security Toolkit) is a comprehensive Rust-based CLI tool for analyzing Solana smart contracts for security vulnerabilities. The project includes static analysis, fuzz testing, reporting capabilities, a plugin system, and an optional React-based UI.
---

file_patterns:
  - "**"
---
# Project Rules and Documentation

## Project Overview

**solsec** (Solana Smart Contract Security Toolkit) is a comprehensive Rust-based CLI tool for analyzing Solana smart contracts for security vulnerabilities. The project includes static analysis, fuzz testing, reporting capabilities, a plugin system, and an optional React-based UI.

## Project Structure

```
solana-smart-contract-security-toolkit/
├── src/
│   ├── main.rs              # CLI entry point
│   ├── cli.rs               # Command-line interface handling
│   ├── analyzer.rs          # Static analysis engine with built-in rules
│   ├── fuzz.rs              # Fuzzing engine with IDL integration
│   ├── report.rs            # Multi-format report generation
│   └── plugin.rs            # Plugin system and rule traits
├── examples/                # Security vulnerability examples
│   ├── integer_overflow/    # Integer overflow vulnerabilities
│   ├── missing_signer_check/ # Missing authorization examples
│   ├── unchecked_account/   # Unsafe account access patterns
│   ├── reentrancy/          # Reentrancy vulnerability examples
│   └── README.md            # Comprehensive examples documentation
├── ui/                      # Optional React TypeScript UI
│   ├── src/
│   ├── package.json
│   └── vite.config.ts
├── .github/workflows/ci.yml # GitHub Actions CI/CD
├── .git/hooks/pre-commit    # Native git hook for code quality
├── Cargo.toml
├── README.md
└── rules.md                 # This file
```

## Core Architecture

### CLI Commands

1. **`solsec scan`** - Static analysis of Solana programs (now generates JSON & HTML by default!)
2. **`solsec fuzz`** - Automated fuzz testing with IDL generation
3. **`solsec plugin`** - Manage security rule plugins

### Built-in Security Rules

The analyzer includes 4 core security rules:

1. **`integer_overflow`** (Medium) - Detects potential integer overflow vulnerabilities
2. **`missing_signer_check`** (High) - Identifies missing signer validation
3. **`unchecked_account`** (Critical) - Finds accounts used without proper validation
4. **`reentrancy`** (High) - Detects potential reentrancy vulnerabilities

### Analyzer Improvements

**Path Validation & Error Handling**:
- **File Existence Checking**: Validates paths exist before analysis
- **File Type Validation**: Warns about non-Rust files
- **Directory Analysis**: Counts and reports Rust files found
- **Proper Error Messages**: Clear, actionable feedback instead of misleading "0 issues found"
- **Colored Error Logging**: ERROR messages appear in red with timestamps
- **Exit Codes**: Returns proper exit codes for different error conditions

**Examples**:
```bash
# Before: misleading output
solsec scan nonexistent.rs → "Found 0 issues" 

# After: clear error messaging  
solsec scan nonexistent.rs → "[ERROR] Path does not exist: nonexistent.rs" + exit code 1
solsec scan file.py → "[WARN] File is not a Rust source file (.rs): file.py"
solsec scan empty_dir/ → "[WARN] No Rust source files (.rs) found in directory: empty_dir"
```

### Plugin System

- **Rule Trait**: All security rules implement the `Rule` trait
- **Plugin Interface**: FFI-safe plugin loading with `#[no_mangle]` functions
- **Extensibility**: Custom rules can be loaded as dynamic libraries

## Code Quality Standards

### Clippy Configuration

**CRITICAL**: The project uses `cargo clippy --all-targets --all-features -- -D warnings`

This means ALL clippy warnings are treated as errors. The following practices MUST be followed:

#### Fixed Clippy Issues and Best Practices

1. **Use `is_some_and()` instead of `map_or(false, |x| condition)`**
   ```rust
   // ❌ Bad
   if path.extension().map_or(false, |ext| ext == "rs") {
   
   // ✅ Good  
   if path.extension().is_some_and(|ext| ext == "rs") {
   ```

2. **Move regex compilation outside loops**
   ```rust
   // ❌ Bad - creates regex in every iteration
   for line in lines {
       if Regex::new(r"pattern").unwrap().is_match(line) {
   
   // ✅ Good - compile once, use many times
   let regex = Regex::new(r"pattern")?;
   for line in lines {
       if regex.is_match(line) {
   ```

3. **Remove unnecessary borrows in function calls**
   ```rust
   // ❌ Bad
   Command::new("cargo").args(&["test", "arg"])
   
   // ✅ Good
   Command::new("cargo").args(["test", "arg"])
   ```

4. **Use iterator methods properly**
   ```rust
   // ❌ Bad - unnecessary enumerate
   for (_idx, item) in items.iter().enumerate() {
   
   // ✅ Good
   for item in items.iter() {
   ```

5. **Handle dead code properly**
   - **Don't use `#[allow(dead_code)]` as first resort**
   - **Actually use the code** by integrating it into the system
   - **Only allow dead code** for FFI interfaces and public APIs meant for external use

### Code Integration Examples

When fixing dead code warnings, integrate functionality:

```rust
// Example: Using rule_settings field properly
for rule in &self.rules {
    let _rule_config = self.config.rule_settings.get(rule.name());
    // Use rule-specific configuration
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hasip-timurtas/solsec](https://github.com/hasip-timurtas/solsec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
