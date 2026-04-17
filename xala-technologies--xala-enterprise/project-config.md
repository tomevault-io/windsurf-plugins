---
trigger: always_on
description: HIGHEST PRIORITY: Internal monorepo dependencies MUST use workspace protocol (workspace:*). External consumption of packages MUST use GitHub Packages with explicit versioning. Never use explicit versions for internal dependencies.
---

HIGHEST PRIORITY: Internal monorepo dependencies MUST use workspace protocol (workspace:*). External consumption of packages MUST use GitHub Packages with explicit versioning. Never use explicit versions for internal dependencies.

MANDATORY: All packages MUST analyze, utilize, and use other internal packages wherever needed. Always prefer shared package usage over local or ad-hoc implementations. This is a critical architectural enforcement.

Here's your consolidated, comprehensive, and framework-agnostic **CursorAI Ruleset** adjusted strictly according to your previous guidelines, ensuring no project specifics are included—only clear and reusable rules aligned with your architecture and standards:

---

# 🧠 **CursorAI Rules (Framework-Agnostic, Standardized)**

## 📦 **Package & Dependency Management**

* Always use `pnpm` for package management.
* **INTERNAL DEPENDENCIES**: Internal dependencies within the monorepo MUST always use workspace protocol (`workspace:*`).
* **EXTERNAL CONSUMPTION**: When packages are consumed externally, they MUST be installed from GitHub Packages with explicit versioning (`^1.0.4`).
* **PUBLISHING**: Private packages hosted on GitHub Packages must be authenticated using GitHub PAT in `.npmrc`.
* **Forbidden** to install duplicate packages that exist within the workspace.
* Regularly run `pnpm install --frozen-lockfile`.

**CLARIFICATION**:
- ✅ **Monorepo Internal**: `"@xala-technologies/foundation": "workspace:*"` (for packages within this repo)
- ✅ **External Projects**: `"@xala-technologies/foundation": "^1.0.4"` (for external consumers)
- ❌ **Never Mix**: Don't use explicit versions for internal dependencies in the same monorepo

---

## 🧱 **Domain-Driven Design (DDD) Rules**

* Clearly separate **Domain**, **Application**, **Infrastructure**, and **Interfaces** layers.
* **Domain Layer**: Only contains domain entities, events, and pure business logic (no frameworks, no IO).
* **Application Layer**: Contains use-cases orchestrating domain logic. No direct external integration allowed.
* **Infrastructure Layer**: Technical implementations (e.g., repositories, DB connections, Redis).
* **Interfaces Layer**: DTOs, controllers, and external communication interfaces.
* Dependencies always point inward (Interfaces → Application → Domain ← Infrastructure).

---

## 🎨 **UI & Design Token Rules**

* UI components **must exclusively use shared design tokens** from a global design system (`@xala-technologies/design-system`).
* No inline CSS or hardcoded styling values; always reference centralized tokens.
* Always build UI using provided shared UI components (`@xala-technologies/ui`).
* Strictly follow accessibility standards (WCAG 2.1 AA minimum).

---

## 🖥️ **Framework-Agnostic Pages & Components**

* Page-level components must **never contain direct HTML elements (`div`, `span`) or framework-specific syntax**.
* Pages must only compose high-level, semantic UI components from the shared UI package (`@xala-technologies/ui`).
* Components must remain portable and **agnostic to frontend frameworks** (React, Vue, Angular, etc.).
* Use composable, generic, framework-independent primitives (e.g., `Container`, `Section`, `Flex`, `Stack`).

---

## 🌐 **Internationalization & Localization**

* Localization strings **must** be defined centrally in the shared localization package (`@xala-technologies/localization`).
* No hardcoded text strings permitted in UI or components.
* Always support at least two languages (default: English, secondary: Norwegian).

---

## 🛠️ **Development & Build Process**

* **Forbidden**: Running standalone dev commands like `pnpm dev` directly in sub-packages.
* **Required**: Always initiate builds from root using `pnpm build`.
* Verify all functionalities through browser inspection or provided automated tests—**never** manually restart running processes without explicit instructions.
* Strictly update `.cursor-updates` file after every major change, summarizing actions clearly.

---

## 🔐 **Security & Compliance**

* All projects must explicitly follow compliance standards:

  * GDPR (data handling, user consent, anonymization).
  * NSM (Norwegian National Security Authority guidelines).
  * DigDir (Digitalisation Directorate architecture principles).
  * ISO27001 (Security Management).
* Secret tokens or sensitive data **must never** be committed to repositories or `.env` files directly.
* All secrets and sensitive configuration must be managed via secure environments (e.g., Vault, GitHub secrets).

---

## 🇳🇴 **Norwegian Standards First + International Flexibility**

* **MANDATORY COMPLIANCE HIERARCHY**:
  1. **Norwegian Standards (PRIMARY - STRICT COMPLIANCE)**:
     * NSM (Nasjonal sikkerhetsmyndighet) - Security standards
     * DigDir (Digitaliseringsdirektoratet) - Digital architecture principles
     * ISO 27001, ISO 14289, ISO 9241 - Norwegian implementations
     * WCAG 2.2 AA + Universal Design (Universell utforming)
  2. **International Standards (SECONDARY - FLEXIBLE COMPATIBILITY)**:
     * ISO standards (international variants)
     * GDPR (EU-wide compliance)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Xala-Technologies) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
