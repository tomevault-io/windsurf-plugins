---
trigger: always_on
description: Standards for Python code metrics implementation
---

# Python Code Metrics Standards

<rule>
name: python_code_metrics
description: Guidelines for measuring and monitoring code quality metrics in Python projects
filters:
  - type: file_extension
    pattern: "\.py$"

actions:
  - type: suggest
    message: |
      ## Python Code Metrics Guidelines

      ### Complexity Metrics
      1. Cyclomatic Complexity:
         - Aim for a maximum of 10 per function/method
         - Use tools like `mccabe` or `radon` for measurement
         - Refactor complex functions into smaller ones
         - Document exceptions with clear rationale
         - Set CI/CD quality gates for complexity

      2. Cognitive Complexity:
         - Measures how difficult code is to understand
         - Keep below 15 for functions/methods
         - Avoid deeply nested control structures
         - Prefer early returns over nested conditions
         - Use SonarQube or similar tools for measurement

      3. Maintainability Index:
         - Composite metric of Halstead Volume, Cyclomatic Complexity, and LOC
         - Higher values (>20) indicate more maintainable code
         - Monitor trends over time
         - Set minimum thresholds for new code
         - Use `radon` for Python-specific measurement

      ### Size Metrics
      1. Lines of Code (LOC):
         - Keep functions under 50 lines
         - Keep classes under 300 lines
         - Keep modules under 1000 lines
         - Exclude comments and blank lines in measurements
         - Track LOC trends over time

      2. Function/Method Count:
         - Keep classes with fewer than 20 methods
         - Identify classes that may violate Single Responsibility Principle
         - Monitor growth over time
         - Consider splitting large classes
         - Use `pylint` or custom tools for measurement

      3. Parameter Count:
         - Limit to 5 parameters per function/method
         - Use data classes or dictionaries for related parameters
         - Consider refactoring functions with many parameters
         - Document exceptions with clear rationale
         - Set linting rules to enforce limits

      ### Duplication Metrics
      1. Code Duplication:
         - Aim for less than 5% duplicated code
         - Use tools like `pylint` or `CPD` for detection
         - Extract common code into shared functions
         - Create utility modules for repeated patterns
         - Set CI/CD quality gates for duplication

      2. Duplication Detection:
         ```python
         # Example configuration for duplicate-code-detection
         # in .pylintrc
         [SIMILARITIES]
         # Minimum lines number of a similarity
         min-similarity-lines=4
         # Ignore comments when computing similarities
         ignore-comments=yes
         # Ignore docstrings when computing similarities
         ignore-docstrings=yes
         # Ignore imports when computing similarities
         ignore-imports=yes
         ```

      ### Dependency Metrics
      1. Coupling:
         - Minimize dependencies between modules
         - Use dependency injection for testability
         - Track fan-in and fan-out metrics
         - Identify highly coupled modules
         - Refactor to reduce coupling

      2. Cohesion:
         - Aim for high cohesion within modules
         - Group related functionality together
         - Measure LCOM (Lack of Cohesion of Methods)
         - Refactor classes with low cohesion
         - Use tools like `pymetrics` for measurement

      ### Test Metrics
      1. Coverage:
         - Aim for minimum 80% code coverage
         - Use `coverage.py` for measurement
         - Focus on critical path coverage
         - Track coverage trends over time
         - Set CI/CD quality gates for coverage

      2. Test Quality:
         - Measure test execution time
         - Track flaky tests
         - Measure mutation testing scores
         - Use tools like `pytest-cov` and `mutmut`
         - Set quality gates for test reliability

      ### Technical Debt
      1. Debt Quantification:
         - Use SonarQube or similar tools
         - Measure debt in time (days) to fix
         - Track debt ratio (debt/total code)
         - Set maximum debt thresholds
         - Create remediation plans for high debt areas

      2. Debt Monitoring:
         - Track debt trends over time
         - Allocate time for debt reduction
         - Prioritize high-impact debt
         - Document known debt with clear rationale
         - Include debt metrics in team reviews

      ### Metric Collection and Reporting
      1. Automation:
         - Integrate metrics into CI/CD pipeline
         - Generate reports automatically
         - Track metrics over time
         - Set up dashboards for visibility
         - Alert on significant regressions

      2. Quality Gates:
         ```yaml
         # Example GitHub Action for quality gates
         name: Code Quality
         on: [push, pull_request]
         
         jobs:
           quality:
             runs-on: ubuntu-latest
             steps:
             - uses: actions/checkout@v2
             - name: Set up Python
               uses: actions/setup-python@v2
               with:
                 python-version: 3.9
             - name: Install dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ramiluisto) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
