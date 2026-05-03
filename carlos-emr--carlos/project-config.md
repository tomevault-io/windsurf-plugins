---
trigger: always_on
description: > **Source of truth**: `CLAUDE.md` at repository root. Last synced: April 2026.
---

# CARLOS EMR - GitHub Copilot Instructions

> **Source of truth**: `CLAUDE.md` at repository root. Last synced: April 2026.
> **Specialized agents**: See `.github/agents/` for domain-specific guidance (security, backend, testing, database, frontend).
> **Path-specific rules**: See `.github/instructions/` for auto-applied rules on JSP, Action, test, and SQL files.

**PROJECT IDENTITY**: Always refer to this system as "CARLOS EMR" or "CARLOS" in user-facing displays and documentation.

## Core Context

**Domain**: Canadian healthcare EMR system with multi-jurisdictional compliance (BC, ON, generic)
**Stack**: Java 21, Spring 7.0.6, Struts 7.1.1, Hibernate 7.2.7, Maven 3, Tomcat 11.0, MariaDB/MySQL
**Architecture**: Multi-layered healthcare web application with complex medical database schema
**Regulatory**: HIPAA/PIPEDA compliance REQUIRED - PHI protection is CRITICAL

### Full Tech Stack (April 2026)

| Category | Technology | Version |
|----------|-----------|---------|
| Language | Java | 21 |
| Framework | Spring Framework | 7.0.6 |
| Security | Spring Security | 7.0.4 |
| Web | Struts | 7.1.1 |
| ORM | Hibernate | 7.2.7 |
| Server | Apache Tomcat | 11.0 |
| Database | MariaDB/MySQL | - |
| CSRF | OWASP CSRFGuard | 4.5 (Jakarta) |
| XSS | OWASP Encoder | 1.4.0 (Jakarta) |
| FHIR | HAPI FHIR | 8.4.0 |
| Web Services | Apache CXF | 4.1.5 |
| Rules | Drools | 10.1.0 |
| Caching | Caffeine | 3.2.3 |
| UI | Bootstrap | 5.3.0 (CDN) |
| Testing | JUnit 5 + AssertJ + H2 | - |

## Essential Commands

```bash
# Build & Deploy
make clean                           # Clean project and remove deployed app
make install                         # Build and deploy without tests
make install --run-tests             # Build, test, and deploy (all tests)
make install --run-unit-tests        # Build and run only unit tests
make install --run-integration-tests # Build and run only integration tests

# Server & Database
server start/stop/restart            # Tomcat management
server log                           # Tail application logs
db-connect                           # Connect to MariaDB as root
debug-on / debug-off                 # Toggle DEBUG/INFO logging levels
```

## Package Namespace

**CRITICAL**: Use `io.github.carlos_emr.carlos.*` for ALL new code.

| Package | Location |
|---------|----------|
| DAOs | `io.github.carlos_emr.carlos.commn.dao.*` (note: "commn" NOT "common") |
| Forms DAOs | `io.github.carlos_emr.carlos.commn.dao.forms.*` |
| Models | `io.github.carlos_emr.carlos.commn.model.*` |
| ProviderDao (exception) | `io.github.carlos_emr.carlos.dao.ProviderDao` |
| Test Utilities | `org.oscarehr.common.dao.*` (backward compatibility) |
| Legacy (DO NOT USE for new code) | `org.oscarehr.*`, `oscar.*` |

## Critical Security Requirements

**MANDATORY for ALL code changes** -- see `carlos-security` agent for full details.

1. **OWASP Encoder** for ALL user input output
   - JSP (preferred): `${e:forHtml(value)}` with taglib `<%@ taglib uri="owasp.encoder.jakarta" prefix="e" %>`
   - Java: `Encode.forHtml(value)`
   - Context-specific variants: `forHtmlAttribute`, `forJavaScript`, `forCssString`, `forUri`, `forUriComponent`
2. **Parameterized queries ONLY** -- never string concatenation in SQL
3. **`SecurityInfoManager.hasPrivilege()`** checks in ALL actions (FIRST operation)
4. **`PathValidationUtils`** for ALL file path operations with user input
5. **PHI must NEVER** be logged or exposed in error messages
6. **CSRF tokens** are auto-injected by CSRFGuard 4.5 -- no manual handling needed

## Struts2 "2Action" Pattern

All new Struts2 actions use `*2Action.java` naming. See `carlos-backend` agent for full template and categories.

```java
public class Feature2Action extends ActionSupport {
    HttpServletRequest request = ServletActionContext.getRequest();
    HttpServletResponse response = ServletActionContext.getResponse();
    private SecurityInfoManager securityInfoManager = SpringUtils.getBean(SecurityInfoManager.class);

    public String execute() {
        if (!securityInfoManager.hasPrivilege(LoggedInInfo.getLoggedInInfoFromSession(request), "_object", "r", null)) {
            throw new SecurityException("missing required security object");
        }
        return "success";
    }
}
```

**Imports**: Use `org.apache.struts2.ActionSupport` (NOT `com.opensymphony.xwork2.*`)

**Struts config is modular** -- 17 domain-specific `struts-*.xml` files. Add new actions to the appropriate domain file, NOT `struts.xml`.

## Spring Integration

```java
private SomeManager someManager = SpringUtils.getBean(SomeManager.class);
```

## Testing

See `carlos-testing` agent for the full test framework guide with all 13 integration pitfalls.

| Base Class | Type | Use When |
|-----------|------|----------|
| `CarlosTestBase` | Integration (H2 DB) | DAOs, Spring context, database queries |
| `CarlosUnitTestBase` | Unit (mocked) | Managers, business logic, no DB |

**BDD naming**: `should<Action>_<preposition><Condition>()` -- ONE underscore, camelCase, `should` prefix.

**Critical**: ALWAYS read the actual DAO/Manager interface before writing tests. Never invent methods.

## Healthcare Domain


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carlos-emr/carlos](https://github.com/carlos-emr/carlos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
