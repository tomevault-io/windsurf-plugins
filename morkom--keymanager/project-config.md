---
trigger: always_on
description: Welcome to the **KeyManager** codebase. This document outlines the system architecture, code conventions, domain implementations, development workflows, and guidelines for both human developers and AI agents working on this
---

# KeyManager Developer Guide & Context Instructions (GEMINI.md)
Welcome to the **KeyManager** codebase. This document outlines the system architecture, code conventions, domain implementations, development workflows, and guidelines for both human developers and AI agents working on this
project.
---
## 1. System Overview & Tech Stack
KeyManager is a secure key, certificate, and XML/XSD schema management utility. It is designed as a modular monorepo composed of a modern Java backend and a responsive React frontend.
### Tech Stack Details:
*   **Backend**:
    *   **Runtime**: Java 25 (OpenJDK via Eclipse Temurin)
    *   **Framework**: Spring Boot 4.0.3 (Spring MVC, Spring Security, Spring Data JPA)
    *   **Database**: PostgreSQL 15 (alpine)
    *   **Cryptography**: Bouncy Castle `1.83` (bcprov-jdk18on, bcpkix-jdk18on, bcpg-jdk18on)
    *   **XML Generation/Parsing**: JLibs XSD (`3.0.1`), XercesImpl (`2.12.2`), Generex (`1.0.2`), JAXB API (`2.3.1`)
*   **Frontend**:
    *   **Framework**: React 18.2 (TypeScript 5.2, Vite 5.2)
    *   **UI Library**: Material-UI (MUI) v5 (using Emotion styled-components)
    *   **Routing**: React Router DOM v6.23
    *   **Localization**: i18next & react-i18next (English & Czech)
    *   **API Client**: Axios (v1.6.8)
*   **DevOps/Containerization**: Docker & Docker Compose
---
## 2. Directory Structure
  KeyManager/
  ├── pom.xml                        # Parent Maven configuration (Java 25, Boot 4.0.3)
  ├── Dockerfile                     # Multi-stage JDK 25 build to JRE 25 runtime
  ├── docker-compose.yml             # Local dev environment (DB, Backend, Frontend)
  ├── crl_db/                        # Mounted volume for Certificate Revocation List database
  ├── keystores/                     # Mounted volume for PKCS12 keystores (.p12)
  ├── src/
  │   ├── main/
  │   │   ├── java/dev/morkom/keymanager/
  │   │   │   ├── config/            # WebConfig, SecurityConfig, and profiles
  │   │   │   ├── controller/        # REST Controllers (DI via constructor)
  │   │   │   ├── dto/               # Request/Response models (using Java Records)
  │   │   │   ├── model/             # JPA Entities (User, Role, AuditEvent)
  │   │   │   ├── repository/        # Spring Data JPA repositories
  │   │   │   ├── security/          # JWT Filters & Utilities (commented out in dev/permitAll)
  │   │   │   ├── service/           # Crypto, CA, XML, SSH, and Audit services
  │   │   │   └── util/              # PKCS12, PEM, and XML Classpath resolvers
  │   │   └── resources/
  │   │       ├── application.properties
  │   │       └── xsd/               # Huge catalog of standard trade & energy XSD schemas
  │   └── test/                      # Java test suite (currently empty)
  └── frontend/
      ├── package.json               # Frontend package definition & lint rules
      ├── index.html
      ├── src/
      │   ├── App.tsx                # Client Routing, Theme & i18n orchestration

 ---

## 3. Core Domain Areas

### 3.1 Key & Certificate Management (PKI)
*   **Keystore Standard**: Uses **PKCS12** (`.p12`) format exclusively for keystores. Outdated JKS formats are avoided.
*   **Provider Registration**: The Bouncy Castle provider must be registered explicitly prior to cryptographic actions. In `PemUtils` and `CaService` this is done in static blocks or constructors:
      Security.addProvider(new BouncyCastleProvider());

*   **Operations**:
    *   *Root CA Generation*: Creating self-signed CA certificates with customizable algorithms, validity, extensions.
    *   *CSR Generation*: Creating private keys & Certificate Signing Requests (PEM format).
    *   *Certificate Signing*: Signing generated CSRs using a configured local CA keystore and outputting either public PEM certificates or full PKCS12 bundles.
    *   *Revocation*: Generating CRLs and revoking certificates based on hex serial numbers and RFC 5280 reason codes.
### 3.2 XML & XSD Engine
*   **Prepackaged XSDs**: Located in `src/main/resources/xsd/`. Many schemas have nested relative paths/includes.
*   **High-Fidelity Resource Resolution**: To correctly resolve imports and inclusions across complex XSD files packed inside a Spring Boot Fat-JAR, use the customized `LSResourceResolver` (`ClasspathResourceResolver`).
*   **Mock XML Generation**: Uses JLibs `XSInstance` to traverse XSD syntax. A custom pattern-based value generator utilizing `Generex` is used to generate valid sample XML strings that strictly satisfy complex regex
constraints, lengths, formats, and XML-specific character bounds.
### 3.3 Symmetric Encryption & Raw Tools
*   Supports Bcrypt hashing and verification.
*   Supports symmetric block ciphers (e.g. AES-128-CBC, AES-256-CBC) with customizable keys and initialization vectors.
*   Supports Base64 and Hex text encoding/decoding.
---
## 4. Development & Architectural Conventions
### 4.1 Backend Code Style
*   **Dependency Injection**: Use **constructor-based injection** instead of field-based `@Autowired`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Morkom/KeyManager](https://github.com/Morkom/KeyManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
