---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md — OpenSearch Security Plugin

Guidance for AI coding agents working in this repository.

## Keeping This File Up to Date

This file should be updated automatically as part of any change that affects the information documented here. Do not treat it as a separate maintenance task — include the `AGENTS.md` update in the same commit or PR as the relevant code change. Examples of changes that should trigger an update:

- Adding, removing, or renaming source packages or top-level directories
- Changing the build system, required JDK version, or key Gradle tasks
- Adding or retiring test suites or testing conventions
- Changing code style tooling or static analysis configuration
- Updating contribution workflows (branching strategy, backport process, commit conventions, etc.)
- Adding new security-specific patterns or constraints agents should be aware of

When in doubt, update the file.

## Repository Overview

OpenSearch Security is a plugin that adds authentication, authorization, TLS encryption, audit logging, and multi-tenancy to OpenSearch. The plugin intercepts all requests at the REST and transport layers before they reach OpenSearch action handlers.

See [ARCHITECTURE.md](ARCHITECTURE.md) for detailed flow diagrams.

## Repository Structure

```
src/
  main/java/org/opensearch/security/
    action/         # Custom OpenSearch actions registered by the plugin
    auditlog/       # Audit and compliance logging framework
    auth/           # Authentication backends (basic, PKI, proxy, LDAP, Kerberos, JWT, OIDC, SAML)
    authtoken/      # On-behalf-of (OBO) token issuance for extensions
    compliance/     # Field-level security and compliance (GDPR/HIPAA/PCI/SOX) features
    configuration/  # Security index management, config hot-reload
    dlic/           # Dynamic configuration REST API (roles, users, role-mappings, etc.)
    filter/         # REST and transport request/response filter pipeline
    hasher/         # Password hashing utilities
    http/           # HTTP/HTTPS layer integration
    httpclient/     # Internal HTTP client for auth backend calls
    identity/       # Subject and identity abstractions
    opensaml/       # OpenSAML integration utilities
    privileges/     # Authorization engine (role evaluation, DLS/FLS)
    queries/        # DLS query building
    resources/      # Resource sharing and access control
    rest/           # REST action handlers and route registration
    securityconf/   # Security configuration model (roles, mappings, users)
    setting/        # Plugin settings definitions
    ssl/            # TLS configuration for transport and HTTP layers
    state/          # Cluster security state management
    support/        # Shared support utilities
    tools/          # Admin tools (securityadmin)
    transport/      # Transport layer interception and enforcement
    user/           # User model and attribute handling
    util/           # General utilities
  test/             # Unit tests
  integrationTest/  # Integration tests (require a running OpenSearch cluster)
spi/                # Security Plugin Interface — public extension points for other plugins
libs/
  opensaml/         # Vendored/shaded OpenSAML library
sample-resource-plugin/  # Example plugin using the resource-sharing SPI
config/             # Default demo configuration (certs, roles, users, mappings)
tools/              # Shell scripts: securityadmin.sh, install_demo_configuration.sh
scripts/            # CI/CD helper scripts
bwc-test/           # Backwards-compatibility tests
checkstyle/         # Checkstyle rule configuration
formatter/          # Eclipse JDT formatter configuration
```

## Build

**Minimum JDK: 21.** `JAVA_HOME` must be set.

The plugin version must match the target OpenSearch version. Check `build.gradle`:

```groovy
opensearch_version = System.getProperty("opensearch.version", "3.8.0-SNAPSHOT")
```

```bash
./gradlew clean assemble          # build the plugin zip
./gradlew check                   # all verification tasks (unit tests, integration, static analysis)
./gradlew precommit               # precommit checks only (run before every commit)
./gradlew spotlessApply           # auto-fix Java formatting
```

## Testing

### Unit Tests

- Located in `src/test/`
- Run with: `./gradlew test`
- HTML results: `build/reports/tests/test/index.html`
- Run a specific test:
  ```bash
  ./gradlew test --tests "org.opensearch.security.auth.BackendRegistryTests"
  ```
- Repeat a test for reliability using the `@Repeat` annotation:
  ```java
  @Rule public RepeatRule repeatRule = new RepeatRule();

  @Test
  @Repeat(10)
  public void testMethod() { ... }
  ```

### Integration Tests

- Located in `src/integrationTest/`
- Subdomain suites: `api`, `auditlog`, `dlsfls`, `grpc`, `hash`, `http`, `privileges`, `rest`, `ssl`, `support`, `systemindex`, `user`, `util`
- Run all integration tests: `./gradlew integrationTest`
- Run a specific suite (as defined in `build.gradle`):
  ```bash
  ./gradlew ciSecurityIntegrationTest   # all *Integ* tests
  ./gradlew dlicDlsflsTest              # Document- and Field-Level Security
  ./gradlew dlicRestApiTest             # REST Management API
  ./gradlew sslTest                     # SSL/TLS tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opensearch-project/security](https://github.com/opensearch-project/security) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
