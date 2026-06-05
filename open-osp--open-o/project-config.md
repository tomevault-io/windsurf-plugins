---
trigger: always_on
description: > **⚠️ DEVCONTAINER ENVIRONMENT NOTICE**
---

# OpenO EMR - Healthcare Electronic Medical Records System

> **⚠️ DEVCONTAINER ENVIRONMENT NOTICE**
>
> The `.claude/settings.json` in this repository grants **extensive pre-approved permissions**
> optimized for **isolated devcontainer development only**. These settings assume:
> - Sandboxed Docker environment with no external network access to production systems
> - Development database with synthetic/test data (no real PHI)
> - Disposable environment that can be safely reset
>
> **DO NOT** use these defaults in shared servers, production environments, or any system
> with access to real patient data. Review and restrict permissions in `.claude/settings.json`
> if running outside an isolated devcontainer.

**PROJECT IDENTITY**: Always refer to this system as "OpenO EMR" or "OpenO" - NOT "OSCAR EMR" or "OSCAR McMaster"

## Core Context

**Domain**: Canadian healthcare EMR system with multi-jurisdictional compliance (BC, ON, generic)
**Stack**: Java 21, Spring 5.3.39, Hibernate 5.x, Maven 3, Tomcat 9.0.97, MariaDB/MySQL
**Regulatory**: HIPAA/PIPEDA compliance REQUIRED - PHI protection is CRITICAL

## Essential Commands

```bash
# Development Workflow
make clean                    # Clean project and remove deployed app
make install                  # Build and deploy without tests
make install --run-tests      # Build, test, and deploy (all tests)
make install --run-modern-tests     # Build and run only modern tests (JUnit 5)
make install --run-legacy-tests     # Build and run only legacy tests (JUnit 4)
make install --run-unit-tests       # Build and run only modern unit tests
make install --run-integration-tests # Build and run only modern integration tests
server start/stop/restart     # Tomcat management
server log                    # Tail application logs

# Database & Environment
db-connect                   # Connect to MariaDB as root
debug-on / debug-off         # Toggle DEBUG/INFO logging levels

gh pr create                 # GitHub pull request creation
```

## Critical Security Requirements

**MANDATORY for all code changes:**
- Use `Encode.forHtml()`, `Encode.forJavaScript()` for ALL user inputs
- Parameterized queries ONLY - never string concatenation
- ALL actions MUST include `SecurityInfoManager.hasPrivilege()` checks
- PHI (Patient Health Information) must NEVER be logged or exposed
- **Use `PathValidationUtils` for ALL file path operations** (see below)

### PathValidationUtils - File Path Security

**ALWAYS use PathValidationUtils** (`ca.openosp.openo.utility.PathValidationUtils`) for file operations involving user input. It prevents path traversal attacks consistently across the codebase.

**Key Methods:**
```java
// For user-provided filenames (sanitizes and validates)
File safeFile = PathValidationUtils.validatePath(userFilename, allowedDir);

// For validating existing file paths
PathValidationUtils.validateExistingPath(file, allowedDir);

// For validating uploaded files from Struts2/Tomcat
PathValidationUtils.validateUpload(uploadedFile);

// For complete upload validation (source + destination)
File dest = PathValidationUtils.validateUpload(sourceFile, filename, destDir);

// For checking if file is in allowed temp directory
if (PathValidationUtils.isInAllowedTempDirectory(file)) { ... }
```

**Migration from old patterns:**
```java
// OLD (inconsistent, error-prone)
if (!file.getCanonicalPath().startsWith(baseDir.getCanonicalPath() + File.separator)) {
    throw new SecurityException("Invalid path");
}

// NEW (consistent, robust)
PathValidationUtils.validateExistingPath(file, baseDir);
```

**Full documentation**: `docs/path-validation-utils.md`

## Package Structure (2025 Migration)

**CRITICAL**: Use NEW namespace `ca.openosp.openo.*` for ALL code
- **Old**: `org.oscarehr.*`, `oscar.*` → **New**: `ca.openosp.openo.*`
- **Note**: May encounter old names in comments/documentation; git history shows "renamed" files
- **DAO Classes**: `ca.openosp.openo.commn.dao.*` (note: "commn" not "common")
- **Forms DAOs**: `ca.openosp.openo.commn.dao.forms.*`
- **Models**: `ca.openosp.openo.commn.model.*`
- **Exception**: `ProviderDao` at `ca.openosp.openo.dao.ProviderDao`
- **Test Utilities**: Remain at `org.oscarehr.common.dao.*` for backward compatibility

## Struts2 Migration Pattern ("2Action")

**CRITICAL PATTERN**: All new Struts2 actions use `*2Action.java` naming convention

### 2Action Structure Template:
```java
public class Example2Action extends ActionSupport {
    HttpServletRequest request = ServletActionContext.getRequest();
    HttpServletResponse response = ServletActionContext.getResponse();

    private SecurityInfoManager securityInfoManager = SpringUtils.getBean(SecurityInfoManager.class);

    public String execute() {
        // MANDATORY security check
        if (!securityInfoManager.hasPrivilege(LoggedInInfo.getLoggedInInfoFromSession(request), "_object", "r", null)) {
            throw new SecurityException("missing required sec object");
        }
        // Business logic
        return "success";
    }
}
```

### 2Action Categories:
1. **Simple Execute**: Single `execute()` method (e.g., `AddTickler2Action`)
2. **Method-Based**: Route via `method` parameter (e.g., `CaseloadContent2Action`)
3. **Inheritance-Based**: Extend `EctDisplayAction` for encounter components


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-osp/Open-O](https://github.com/open-osp/Open-O) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
