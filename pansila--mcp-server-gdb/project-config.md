---
trigger: always_on
description: "name": ".cursorrules",
---

{
  "metadata": {
    "name": ".cursorrules",
    "pseudonym": "Rust-Eze",
    "version": "1.7.2",
    "description": "Advanced Rust code analysis and correction system with enhanced context-specific error resolution strategies",
    "maintainers": [
      "LordXyn@proton.me"
    ],
    "author_github": "Lord Xyn <https://github.com/arcmoonstudios/Rust-Eze>",
    "last_updated": "2024-12-06",
    "changelog": {
      "1.3.0": "Added enhanced argument matching, error prevention, and code organization rules",
      "1.3.1": "Refined formatting rules, clarified documentation formats, and improved internal consistency",
      "1.4.0": "Introduced common error pattern recognition and heuristic-driven suggestions for complex compiler errors",
      "1.5.0": "Expanded patterns to handle no-variant errors, conflicting implementations, `?` operator misuse, and missing structure fields",
      "1.6.0": "Added patterns for invalid type category usage, invalid trait references, and invalid self parameter usage",
      "1.7.0": "Enhanced to handle trait bound failures, ambiguous items, and additional missing `default` method scenarios",
      "1.7.1": "Re-implemented prior correcting implementations that were erroneously removed, added author and github links to metadata",
      "1.7.2": "Added extensive performance optimization, IDE integration, LSP support, and memory management configurations"
    }
  },
  "language": "rust",
  "performance": {
    "caching": {
      "enable": true,
      "max_cache_size": "512MB",
      "cache_duration": "24h",
      "cache_invalidation": "on_file_change"
    },
    "parallel_processing": {
      "enable": true,
      "max_threads": 4,
      "priority_tasks": ["error_analysis", "code_completion"]
    },
    "lazy_loading": {
      "enable": true,
      "modules": ["documentation", "advanced_analysis"]
    }
  },
  "ide_integration": {
    "cursor_specific": {
      "completion_triggers": [".", "::", "(", "{", "["],
      "hover_documentation": true,
      "inline_hints": true,
      "code_actions": {
        "quick_fixes": true,
        "refactorings": true
      }
    },
    "keybindings": {
      "quick_fix": "Alt+Enter",
      "show_documentation": "Ctrl+Q",
      "goto_definition": "Ctrl+Click"
    }
  },
  "lsp": {
    "rust_analyzer": {
      "enable": true,
      "checkOnSave": true,
      "procMacro": {
        "enable": true
      },
      "cargo": {
        "allFeatures": true,
        "loadOutDirsFromCheck": true
      }
    },
    "diagnostics": {
      "enable": true,
      "experimental": true
    }
  },
  "file_watching": {
    "enable": true,
    "watch_patterns": ["**/*.rs", "Cargo.toml", "Cargo.lock"],
    "ignore_patterns": ["target/**", ".git/**"],
    "auto_reload": {
      "enable": true,
      "delay_ms": 500
    }
  },
  "memory_management": {
    "max_memory": "2GB",
    "garbage_collection": {
      "enable": true,
      "interval": "5m"
    },
    "buffer_size": {
      "analysis": "256MB",
      "completion": "128MB"
    }
  },
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
        "explanation": "Ensures code is always compilable, catching basic errors early."
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
        "explanation": "Lints help maintain high code quality and identify potential safety risks."
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
        "explanation": "Testing verifies that changes do not break existing behavior and meet intended functionality."
      },
      "fmt": {
        "command": "cargo fmt -- --check",
        "priority": 4,
        "category": "style",
        "description": "Check code formatting against Rustfmt guidelines to maintain style consistency.",
        "explanation": "Ensures code remains readable and follows community formatting conventions."
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
      "explanation": "Enable flexible environment settings for different development and CI contexts."
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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pansila/mcp_server_gdb](https://github.com/pansila/mcp_server_gdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
