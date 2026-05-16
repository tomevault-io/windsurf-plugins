---
trigger: always_on
description: This is a PHPStan extension that provides custom PHPStan rules with configurable options. Each rule has its own namespace under `src/Rules/` and follows a consistent architecture pattern.
---

# MeliorStan - AI Coding Instructions

## Project Overview
This is a PHPStan extension that provides custom PHPStan rules with configurable options. Each rule has its own namespace under `src/Rules/` and follows a consistent architecture pattern.

## Architecture Pattern
Each rule follows a 3-component structure:
- `{RuleName}Rule.php` - The main rule implementing `Rule<NodeType>`
- `Config.php` - Configuration class with boolean flags for rule options. This class is used to define the configuration options for the rule. Not all rules have configuration options, but those that do will have a `Config` class this that do not will not.
- Test files in `tests/Rules/{RuleName}/` with multiple configuration scenarios if applicable.
- Unlike most other PHPStan extensions, this one does not use a single monolithic rule class. Each rule is its own class with its own config. And a User would enable/disable each rule individually in their PHPStan config. We do not register all rules for the user in our extension.neon.

### Key Files
- `config/extension.neon` - Central dependency injection & parameter schema definition
- `tests/Rules/*/config/*.neon` - Per-test configuration files that override defaults
- `tests/Rules/*/Fixture/*.php` - Test fixtures with various naming patterns (one class/trait/interface/enum per file)

## Critical Development Workflows

### Testing
```bash
composer test              # Run all tests with paratest
./vendor/bin/phpunit tests/Rules/SpecificRule/  # Test specific rule
```

### Code Style
```bash
composer format            # Auto-fix code style with PHP-CS-Fixer
composer analyze           # Run PHPStan analysis
```

### Running Final Checks
Before finalizing changes, you need to ensure that formatting has been applied, static analysis is passing, and all tests are successful. It is important to run these checks in order. First check formatting, then static analysis, and finally run all tests. This is because formatting may change line numbers in tests, which can cause test failures if not done first.

## Configuration Architecture
The extension uses Neon dependency injection with a hierarchical configuration system:

1. **Schema Definition** (`config/extension.neon`): Defines parameter structure and defaults
2. **Test Overrides** (`tests/Rules/*/config/*.neon`): Override specific parameters for test scenarios
3. **Service Registration**: Config classes are auto-wired using parameter references

### Critical Pattern: Config Parameter Mapping
In `config/extension.neon`, ensure Config service arguments match the correct parameter namespace:
```neon
- factory: Orrison\MeliorStan\Rules\CamelCaseParameterName\Config
  arguments:
    - %meliorstan.camel_case_parameter_name.allow_consecutive_uppercase%  # Not property_name!
    - %meliorstan.camel_case_parameter_name.allow_underscore_prefix%
```

### Rule Registration for New Rules
When creating a new rule, you must add BOTH schema definition AND service registration to `config/extension.neon`:

1. **Add parameter schema** (in `parametersSchema` section):
```neon
parametersSchema:
    meliorstan: structure([
        # ... existing rules
        new_rule_name: structure([
            config_option: bool(),
        ]),
    ])
```

2. **Add default parameters** (in `parameters` section):
```neon
parameters:
    meliorstan:
        # ... existing rules
        new_rule_name:
            config_option: false
```

3. **Add service registration** (in `services` section):
```neon
services:
    # ... existing services
    -
        factory: Orrison\MeliorStan\Rules\NewRuleName\Config
        arguments:
            - %meliorstan.new_rule_name.config_option%
```

**IMPORTANT**: The main rule class itself is NOT registered in extension.neon. Users must register the rule in their own PHPStan configuration.

### Test Configuration Files
Test config files must include BOTH the extension config AND rule registration:

```neon
includes:
    - ../../../../config/extension.neon

rules:
    - Orrison\MeliorStan\Rules\NewRuleName\NewRuleNameRule

parameters:
    meliorstan:
        new_rule_name:
            config_option: true  # Override for this test
```

**Critical**: Without the `rules:` section, tests will fail with "Service of type ... not found" error.

### Config Parameter Naming Conventions
When adding new rules to `config/extension.neon`, follow these naming patterns:

1. **Schema keys**: Use snake_case matching the rule directory name:
   ```neon
   boolean_get_method_name: structure([    # Directory: BooleanGetMethodName
       check_parameterized_methods: bool(),
   ])
   ```

2. **Parameter references**: Must match the schema key exactly:
   ```neon
   - factory: Orrison\MeliorStan\Rules\BooleanGetMethodName\Config
     arguments:
       - %meliorstan.boolean_get_method_name.check_parameterized_methods%
   ```

3. **Config method names**: Use camelCase with "get" prefix:
   ```php
   public function getCheckParameterizedMethods(): bool
   ```

## Rule Implementation Patterns

### Node Type Selection
- **ClassMethod**: Use `ClassMethod::class` for method name rules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Orrison/MeliorStan](https://github.com/Orrison/MeliorStan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
