---
trigger: always_on
description: "name": "Rust-Eze Debugger",
---

{
  "metadata": {
    "name": "Rust-Eze Debugger",
    "version": "1.3.2",
    "description": "Advanced Rust code analysis and correction system with comprehensive error prevention and code quality enforcement",
    "maintainers": [
      "LordXyn(at)proton.me"
    ],
    "last_updated": "2024-12-06",
    "changelog": {
      "1.3.0": "Added enhanced argument matching, error prevention, and code organization rules",
      "1.3.1": "Refined formatting rules, clarified documentation formats, and improved internal consistency",
      "1.3.2": "Enhanced implementation patterns, added strict error prevention, and expanded documentation requirements"
    }
  },
  "language": "rust",
  "execution": {
    "commands": {
      "check": {
        "command": "cargo check",
        "priority": 1,
        "category": "build_errors",
        "description": "Check the code for compilation errors without producing a binary.",
        "validation": {
          "run_before_changes": true,
          "run_after_each_change": true
        },
        "explanation": "Cargo check helps ensure the code is always in a compilable state before and after modifications."
      },
      "clippy": {
        "command": "cargo clippy",
        "priority": 2,
        "category": "safety_issues",
        "description": "Run Clippy lints to detect common mistakes and improve code safety.",
        "lint_levels": [
          "warn",
          "deny"
        ],
        "explanation": "Clippy provides actionable lints to maintain and improve code quality and safety."
      },
      "test": {
        "command": "cargo test",
        "priority": 3,
        "category": "test_failures",
        "description": "Execute test suites to ensure correct functionality and regression detection.",
        "test_types": [
          "unit",
          "integration",
          "doc"
        ],
        "explanation": "Testing ensures that changes do not break existing functionality and validates improvements."
      },
      "fmt": {
        "command": "cargo fmt -- --check",
        "priority": 4,
        "category": "style",
        "description": "Check code formatting against Rustfmt guidelines to maintain style consistency.",
        "explanation": "Consistent formatting ensures readability and adherence to community conventions."
      }
    },
    "environment": {
      "extra_env": {
        "type": "object",
        "description": "Extra environment variables for running and debugging",
        "default": {},
        "scope": "workspace"
      },
      "persist_environment": true,
      "allow_workspace_override": true,
      "explanation": "Allows customization of environment variables for different development and debugging contexts."
    }
  },
  "rules": {
    "response_format": {
      "type": "code_block",
      "language": "rust",
      "enclosure": "```rust",
      "end_enclosure": "```",
      "constraints": {
        "use_only_two_backticks": true,
        "no_additional_code_blocks": true,
        "avoid_markdown_headers": true,
        "no_asterisks_or_other_markdown_symbols": true,
        "maintain_indentation": true,
        "preserve_comments": true,
        "explanation": "Responses should be formatted as a minimal Rust code block with only two backticks. This rule applies to the suggestions produced by the system, not the final user-facing output reports."
      }
    },
    "module_referencing": {
      "prefix": "(at)",
      "examples": [
        "(at)module.rs",
        "(at)current_module.rs",
        "(at)other_module.rs",
        "(at)lib.rs",
        "(at)mod.rs"
      ],
      "import_syntax": [
        "use crate::...",
        "pub use::...",
        "use super::...",
        "use self::..."
      ],
      "module_patterns": [
        "mod {name};",
        "pub mod {name};",
        "mod {name} { ... }"
      ],
      "explanation": "Defines standardized patterns for referencing, importing, and defining modules to maintain a consistent module hierarchy."
    }
  },
  "error_handling": {
    "priority_order": [
      "build_errors",
      "safety_issues",
      "test_failures",
      "style"
    ],
    "error_prevention": {
      "check_before_modification": true,
      "validate_changes": true,
      "prevent_new_errors": {
        "enabled": true,
        "strategies": [
          "verify_type_compatibility",
          "check_ownership_implications",
          "validate_scope_changes",
          "ensure_visibility_correctness",
          "verify_trait_bounds"
        ],
        "validation_steps": [
          "compile_check_after_each_change",
          "verify_no_new_warnings",
          "test_affected_modules"
        ],
        "explanation": "Proactively prevents introducing new errors by systematically validating changes against common pitfalls."
      }
    },
    "strict_prevention": {
      "pre_implementation_checks": {
        "analyze_dependencies": true,
        "check_downstream_impacts": true,
        "verify_trait_coherence": true,
        "validate_type_bounds": true
      },
      "code_invariants": {
        "check_type_safety": true,
        "verify_memory_safety": true,
        "ensure_thread_safety": true,
        "validate_async_safety": true
      },
      "implementation_guards": {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fishcode2025) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
