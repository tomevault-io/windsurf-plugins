---
trigger: always_on
description: PHP & Drupal Development Standards and Best Practices
---

# Enhanced PHP & Drupal Development Standards

Defines comprehensive coding standards and best practices for PHP and Drupal development, with a focus on modern PHP features, Drupal 10+ standards, and modularity.

<rule>
name: enhanced_php_drupal_best_practices
description: Enforce PHP 8.3+ features, Drupal 10+ coding standards, and modularity
filters:
  - type: file_extension
    pattern: "\\.(php|module|inc|install|theme)$"
  - type: file_path
    pattern: "web/modules/custom/|web/themes/custom/"

actions:
  - type: enforce
    conditions:
      - pattern: "^(?!declare\\(strict_types=1\\);)"
        message: "Add 'declare(strict_types=1);' at the beginning of PHP files for type safety."

      - pattern: "(?<!\\bTRUE\\b)\\btrue\\b|(?<!\\bFALSE\\b)\\bfalse\\b|(?<!\\bNULL\\b)\\bnull\\b"
        message: "Use uppercase for TRUE, FALSE, and NULL constants."

      - pattern: "(?i)\\/\\/\\s[a-z]"
        message: "Ensure inline comments begin with a capital letter and end with a period."

      - pattern: "class\\s+\\w+\\s*(?!\\{[^}]*readonly\\s+\\$)"
        message: "Consider using readonly properties where immutability is required."

      - pattern: "public\\s+function\\s+\\w+\\([^)]*\\)\\s*(?!:)"
        message: "Add return type declarations for all methods to ensure type safety."

      - pattern: "extends\\s+\\w+\\s*\\{[^}]*public\\s+function\\s+\\w+\\([^)]*\\)\\s*(?!#\\[Override\\])"
        message: "Add #[Override] attribute for overridden methods for clarity."

      - pattern: "\\$\\w+\\s*(?!:)"
        message: "Use typed properties with proper nullability for better code maintainability."

      - pattern: "function\\s+hook_\\w+\\([^)]*\\)\\s*(?!:)"
        message: "Add type hints and return types for all hooks to leverage PHP's type system."

      - pattern: "new\\s+\\w+\\([^)]*\\)\\s*(?!;\\s*//\\s*@inject)"
        message: "Use proper dependency injection with services for better testability and modularity."

      - pattern: "extends\\s+FormBase\\s*\\{[^}]*validate"
        message: "Implement proper form validation in FormBase classes for security."

      - pattern: "function\\s+\\w+\\s*\\([^)]*\\)\\s*\\{[^}]*\\$this->t\\("
        message: "Use Drupal's t() function for strings that need translation."

      - pattern: "\\$this->config\\('\\w+'\\)"
        message: "Use ConfigFactory for configuration management."
      
      - pattern: "array\\s*\\("
        message: "Use short array syntax ([]) instead of array() for consistent code style."
      
      - pattern: "(?<!\\()\\s+\\(int\\)\\s*\\$"
        message: "Put a space between the (type) and the $variable in a cast: (int) $mynumber."
      
      - pattern: "\\n[\\t ]+\\n"
        message: "Remove whitespace from empty lines."
      
      - pattern: "\\s+$"
        message: "Remove trailing whitespace at the end of lines."
      
      - pattern: "^(?!.*\\n$)"
        message: "Ensure files end with a single newline character."
      
      - pattern: "if\\s*\\([^)]*\\)\\s*\\{[^{]*\\}\\s*else\\s*\\{"
        message: "Place the opening brace on the same line as the statement for control structures."
      
      - pattern: "\\$_GET|\\$_POST|\\$_REQUEST"
        message: "Never use superglobals directly; use Drupal's input methods."
      
      - pattern: "mysql_|mysqli_"
        message: "Use Drupal's database API instead of direct MySQL functions."
      
      - pattern: "\\t+"
        message: "Use 2 spaces for indentation, not tabs."
      
      - pattern: "function\\s+\\w+\\s*\\([^)]*\\)\\s*\\{[^}]*\\becho\\b"
        message: "Don't use echo; use return values or Drupal's messenger service."
      
      - pattern: "(?<!\\/\\*\\*)\\s*\\*\\s+@"
        message: "Use proper DocBlock formatting for documentation."
      
      - pattern: "\\bdie\\b|\\bexit\\b"
        message: "Don't use die() or exit(); throw exceptions instead."
      
      - pattern: "\\$entity->get\\([^)]+\\)->getValue\\(\\)"
        message: "Use $entity->get('field_name')->value instead of getValue() when possible."
      
      - pattern: "\\bvar_dump\\b|\\bprint_r\\b|\\bdump\\b"
        message: "Don't use debug functions in production code; use Drupal's logger instead."
      
      - pattern: "\\bnew\\s+DateTime\\b"
        message: "Use Drupal's DateTimeInterface and DrupalDateTime instead of PHP's DateTime."
      
      - pattern: "\\beval\\b"
        message: "Never use eval() as it poses security risks."
      
      - pattern: "function\\s+\\w+_menu_callback\\("
        message: "Use controller classes with route definitions instead of hook_menu() callbacks."
      
      - pattern: "\\/\\*\\*(?:[^*]|\\*[^/])*?@file(?:[^*]|\\*[^/])*?\\*\\/"
        message: "All PHP files must include proper @file documentation in the docblock."
      
      - pattern: "function\\s+\\w+\\s*\\((?:[^)]|\\([^)]*\\))*\\)\\s*\\{(?:[^}]|\\{[^}]*\\})*\\$_SESSION"
        message: "Use Drupal's user session handling instead of $_SESSION."
      
      - pattern: "function\\s+theme_\\w+\\("
        message: "Theme functions should be replaced with Twig templates in Drupal 8+."
      
      - pattern: "drupal_add_js|drupal_add_css"
        message: "Use #attached in render arrays instead of drupal_add_js() or drupal_add_css()."
      

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ivangrynenko) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
