---
trigger: always_on
description: This package is part of WP-CLI, the official command line interface for WordPress. For a detailed explanation of the project structure and development workflow, please refer to the main @README.md file.
---

# Instructions

This package is part of WP-CLI, the official command line interface for WordPress. For a detailed explanation of the project structure and development workflow, please refer to the main @README.md file.

## Best Practices for Code Contributions

When contributing to this package, please adhere to the following guidelines:

* **Follow Existing Conventions:** Before writing any code, analyze the existing codebase in this package to understand the coding style, naming conventions, and architectural patterns.
* **Focus on the Package's Scope:** All changes should be relevant to the functionality of the package.
* **Write Tests:** All new features and bug fixes must be accompanied by acceptance tests using Behat. You can find the existing tests in the `features/` directory. There may be PHPUnit unit tests as well in the `tests/` directory.
* **Update Documentation:** If your changes affect the user-facing functionality, please update the relevant inline code documentation.

### Building and running

Before submitting any changes, it is crucial to validate them by running the full suite of static code analysis and tests. To run the full suite of checks, execute the following command: `composer test`.

This single command ensures that your changes meet all the quality gates of the project. While you can run the individual steps separately, it is highly recommended to use this single command to ensure a comprehensive validation.

### Useful Composer Commands

The project uses Composer to manage dependencies and run scripts. The following commands are available:

* `composer install`: Install dependencies.
* `composer test`: Run the full test suite, including linting, code style checks, static analysis, and unit/behavior tests.
* `composer lint`: Check for syntax errors.
* `composer phpcs`: Check for code style violations.
* `composer phpcbf`: Automatically fix code style violations.
* `composer phpstan`: Run static analysis.
* `composer phpunit`: Run unit tests.
* `composer behat`: Run behavior-driven tests.

### Coding Style

The project follows the `WP_CLI_CS` coding standard, which is enforced by PHP_CodeSniffer. The configuration can be found in `phpcs.xml.dist`. Before submitting any code, please run `composer phpcs` to check for violations and `composer phpcbf` to automatically fix them.

## Documentation

The `README.md` file might be generated dynamically from the project's codebase using `wp scaffold package-readme` ([doc](https://github.com/wp-cli/scaffold-package-command#wp-scaffold-package-readme)). In that case, changes need to be made against the corresponding part of the codebase.

### Inline Documentation

Only write high-value comments if at all. Avoid talking to the user through comments.

## Testing

The project has a comprehensive test suite that includes unit tests, behavior-driven tests, and static analysis.

* **Unit tests** are written with PHPUnit and can be found in the `tests/` directory. The configuration is in `phpunit.xml.dist`.
* **Behavior-driven tests** are written with Behat and can be found in the `features/` directory. The configuration is in `behat.yml`.
* **Static analysis** is performed with PHPStan.

All tests are run on GitHub Actions for every pull request.

When writing tests, aim to follow existing patterns. Key conventions include:

* When adding tests, first examine existing tests to understand and conform to established conventions.
* For unit tests, extend the base `WP_CLI\Tests\TestCase` test class.
* For Behat tests, only WP-CLI commands installed in `composer.json` can be run.

### Behat Steps

WP-CLI makes use of a Behat-based testing framework and provides a set of custom step definitions to write feature tests.

> **Note:** If you are expecting an error output in a test, you need to use `When I try ...` instead of `When I run ...` .

#### Given

* `Given an empty directory` - Creates an empty directory.
* `Given /^an? (empty|non-existent) ([^\s]+) directory$/` - Creates or deletes a specific directory.
* `Given an empty cache` - Clears the WP-CLI cache directory.
* `Given /^an? ([^\s]+) (file|cache file):$/` - Creates a file with the given contents.
* `Given /^"([^"]+)" replaced with "([^"]+)" in the ([^\s]+) file$/` - Search and replace a string in a file using regex.
* `Given /^that HTTP requests to (.*?) will respond with:$/` - Mock HTTP requests to a given URL.
* `Given WP files` - Download WordPress files without installing.
* `Given wp-config.php` - Create a wp-config.php file using `wp config create`.
* `Given a database` - Creates an empty database.
* `Given a WP install(ation)` - Installs WordPress.
* `Given a WP install(ation) in :subdir` - Installs WordPress in a given directory.
* `Given a WP install(ation) with Composer` - Installs WordPress with Composer.
* `Given a WP install(ation) with Composer and a custom vendor directory :vendor_directory` - Installs WordPress with Composer and a custom vendor directory.
* `Given /^a WP multisite (subdirectory|subdomain)?\s?(install|installation)$/` - Installs WordPress Multisite.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [afragen/wp-debugging](https://github.com/afragen/wp-debugging) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
