---
trigger: always_on
description: > **⚠️ DEVCONTAINER ENVIRONMENT NOTICE**
---

# CARLOS EMR - Healthcare Electronic Medical Records System

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

**PROJECT IDENTITY**: Always refer to this system as "CARLOS EMR" or "CARLOS" in all user-facing content

## Project Origin & Naming Conventions

**CARLOS** (Clinical Assisting Recording Ledger Open Source) is a fork of the OpenO EMR project, maintained at `github.com/carlos-emr/carlos`.

**Project Lineage:**
- **CARLOS** ← forked from **OpenO EMR** ← forked from **OSCAR McMaster**
- **No affiliation** with OpenOSP organization or McMaster University
- Repository: `github.com/carlos-emr/carlos`

**Critical Naming Distinctions:**
- **Display Name**: Use "CARLOS" or "CARLOS EMR" in all user-facing content (UI, documentation, README files, issue templates, workflows)
- **Code Namespaces**: Package names remain as `io.github.carlos_emr.carlos.*` for backward compatibility - DO NOT change these internal paths
- **Legacy References**: You may encounter "OpenO", "Open-O", "OpenOSP", or "OSCAR" in code comments, JavaDoc, and git history - these refer to the upstream project heritage, NOT organizational affiliations

**When to Use Which Name:**
- ✅ **Use "CARLOS EMR"**: README files, documentation headers, GitHub templates, workflow descriptions, user-facing messages
- ✅ **Keep as-is**: Java packages (`io.github.carlos_emr.carlos.*`), internal class references, database schema names, Spring bean names
- ⚠️ **Update carefully**: Comments referencing project name (use "CARLOS"), but preserve technical accuracy for upstream references

## Core Context

**Domain**: Canadian healthcare EMR system with multi-jurisdictional compliance (BC, ON, generic)
**Stack**: Java 21, Spring 7.0.6, Struts 7.1.1, Hibernate 7.2.7, Maven 3, Tomcat 11.0, MariaDB/MySQL
**Regulatory**: HIPAA/PIPEDA compliance REQUIRED - PHI protection is CRITICAL


## Fax Provider Feature Context (AI + Dev)

- Provider-specific fax transport is now selected by `FaxConfig.providerType` (`MIDDLEWARE` or `SRFAX`).
- Admin configuration path is the existing UI: **Administration > Faxes > Configure Fax**.
- Fax configuration requires `_admin.fax` write rights; scheduler controls use `_admin.fax.restart`.
- SRFax duplicate prevention policy is unread/read flag based (unread-only pull + mark-as-read), not remote delete.
- See `docs/fax-provider-configuration-and-ux.md` for implementation and operational details.

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
- OWASP Encoder for ALL user inputs (see [OWASP Encoding](#owasp-encoding--xss-prevention) below)
- Parameterized queries ONLY - never string concatenation
- ALL actions MUST include `SecurityInfoManager.hasPrivilege()` checks
- PHI (Patient Health Information) must NEVER be logged or exposed
- **Use `PathValidationUtils` for ALL file path operations** (see below)

### OWASP Encoding — XSS Prevention

The project includes two OWASP Encoder libraries (`pom.xml`):
- **`encoder`** (1.4.0) — Java static methods: `Encode.forHtml()`, etc.
- **`encoder-jakarta-jsp`** (1.4.0) — JSP EL functions: `${e:forHtml()}`, etc. (Jakarta EE edition)

**Taglib declaration** (required once per JSP that uses EL functions):
```jsp
<%@ taglib uri="owasp.encoder.jakarta" prefix="e" %>
```

> **Note**: The project uses `encoder-jakarta-jsp` (Jakarta EE), **not** the legacy `encoder-jsp`.
> The Jakarta edition registers its TLD under `owasp.encoder.jakarta`, not the legacy URI
> `https://www.owasp.org/index.php/OWASP_Java_Encoder_Project`. Using the wrong URI will
> cause JSPC compilation failures in CI.

**Quick-Reference — All Encoding Contexts:**

| Context | EL Function (preferred in JSP) | Java / Scriptlet |
|---------|-------------------------------|------------------|
| HTML body | `${e:forHtml(value)}` | `Encode.forHtml(value)` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/carlos-emr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
