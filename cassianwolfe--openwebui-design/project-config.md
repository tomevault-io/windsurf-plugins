---
trigger: always_on
description: Dependency management and versioning standards
---


# Cursor Rules: Dependency Management Standards

rules:
  - id: dependencies.compliance.production_grade
    description: |
      All dependency management must meet federal and government-regulated agency production standards.
      - Dependencies must support production-ready, fully functional, enterprise-grade code
      - No dependencies with known vulnerabilities or incomplete implementations permitted
      - All dependencies must be validated for security compliance (NIST, FISMA, FedRAMP)
      - Dependency updates must be auditable, traceable, and compliant with regulatory requirements
      - Security audits must pass before adding or updating dependencies
      - All dependencies must be documented with rationale and security considerations
      - Dependency management must handle all edge cases and error conditions explicitly
      - Version pinning must ensure reproducible, secure builds
    severity: error
  - id: dependencies.versioning.strategy
    description: |
      Dependency versioning must follow consistent strategy.
      - Pin exact versions for production dependencies
      - Use version ranges for development dependencies when appropriate
      - Document version pinning rationale
      - Use semantic versioning for dependency versions
      - Update dependencies regularly and test updates
      - Document dependency update procedures
    severity: error

  - id: dependencies.versioning.pinning
    description: |
      Critical dependencies must be pinned to specific versions.
      - Pin security-sensitive dependencies to exact versions
      - Pin framework dependencies (FastAPI, Pydantic, etc.) to compatible versions
      - Use version constraints (>=, <=) for non-critical dependencies
      - Document version constraints and compatibility requirements
      - Test with minimum and maximum supported versions
    severity: error

  - id: dependencies.security.updates
    description: |
      Security updates must be applied promptly.
      - Monitor dependencies for security vulnerabilities
      - Apply security patches within 48 hours when available
      - Test security updates before deploying
      - Document security update procedures
      - Maintain security update log
    severity: error

  - id: dependencies.security.audit
    description: |
      Dependency security audits must be conducted regularly.
      - Run dependency vulnerability scans weekly
      - Use automated tools (safety, pip-audit, Dependabot)
      - Review and address vulnerability reports
      - Document audit procedures and tools
      - Maintain audit history
    severity: error

  - id: dependencies.optional.management
    description: |
      Optional dependencies must be clearly documented and managed.
      - Use optional dependency groups (dev, ml, etc.)
      - Document optional dependencies and their purposes
      - Provide clear installation instructions
      - Handle missing optional dependencies gracefully
      - Test with and without optional dependencies
    severity: error

  - id: dependencies.optional.graceful_degradation
    description: |
      Code must handle missing optional dependencies gracefully.
      - Check for optional dependencies before use
      - Provide clear error messages when optional dependencies are missing
      - Document optional dependency requirements
      - Use try/except for optional imports
      - Provide fallback behavior when possible
    severity: error

  - id: dependencies.documentation.requirements
    description: |
      Dependency requirements must be documented.
      - Document all dependencies and their purposes
      - Document minimum Python version requirements
      - Document optional dependencies and installation
      - Document dependency update procedures
      - Keep dependency documentation current
    severity: error

  - id: dependencies.quality.review
    description: |
      New dependencies must be reviewed before addition.
      - Review dependency licenses for compliance
      - Review dependency maintenance status
      - Review dependency security history
      - Review dependency size and impact
      - Document rationale for new dependencies
    severity: error

  - id: dependencies.quality.minimal
    description: |
      Dependencies should be minimal and necessary.
      - Avoid adding dependencies for single-use functionality
      - Prefer standard library over external dependencies
      - Consider maintenance burden of dependencies
      - Review and remove unused dependencies regularly
      - Document dependency removal procedures
    severity: warning

  - id: dependencies.quality.compatibility
    description: |
      Dependency compatibility must be maintained.
      - Test dependency compatibility regularly
      - Document known compatibility issues
      - Maintain compatibility matrix
      - Test with different dependency versions
      - Document upgrade paths for major version changes
    severity: warning

examples:
  - description: "Dependency versioning in pyproject.toml"
    language: toml
    code: |
      [project]
      dependencies = [
          # Core dependencies - pinned for stability
          "fastapi>=0.104.0,<0.105.0",
          "uvicorn[standard]>=0.24.0,<0.25.0",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cassianwolfe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
