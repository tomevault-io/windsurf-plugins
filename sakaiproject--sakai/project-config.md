---
trigger: always_on
description: - `mvn install` - Build the project
---

# Sakai Development Guide

## Build Commands
- `mvn install` - Build the project
- `mvn clean install sakai:deploy -Dmaven.tomcat.home=/path/to/tomcat` - Deploy to Tomcat
- `mvn test -Dtest=TestClassName` - Run a single test class
- `mvn test -Dtest=TestClassName#testMethodName` - Run a single test method

## Frontend Commands
- `cd webcomponents/tool/src/main/frontend` - Change to frontend directory
- `npm run lint` - Run ESLint for JavaScript
- `npm run bundle` - Bundle JS files using esbuild
- `npm run analyze` - Run lit-analyzer for static type checking

## Architecture
- **Request-Scoped State**: Do not use `ThreadLocal`, static fields, or singleton bean fields to pass request/import/user/site-specific state through service calls. Sakai runs on pooled application-server threads, so thread-bound state can leak across requests if cleanup is missed. Prefer explicit return values, operation-scoped helper instances, DTO/result objects, or method parameters.

### Service Boundaries
- **Business Logic**: Put business rules, parsing, import/copy semantics, workflow decisions, and cross-object coordination in services, not controllers, JSF listeners, Wicket panels, entity producers, or repositories.
- **Controllers and Listeners**: Keep request/response handling, navigation, model setup, and framework glue in the UI layer; delegate behavior to services.
- **Repositories**: Keep repositories focused on persistence of mapped entities and queries. Create adapters, DTO assembly, and tool-specific workflow objects in services.
- **Singleton State**: Do not store request-specific state such as user id, site id, locale, timezone, or request parameters in singleton Spring beans/controllers.
- **Strategy Consistency**: Avoid mixing competing solutions for the same problem, such as backend locks plus UI state throttling, unless both are explicitly required and documented.

### Kernel
- **Core Services**: The Kernel provides core services that should be used by all tools
- **User Management**: Services for fetching and managing User objects
- **Email Service**: Centralized email sending functionality
- **Authorization**: Security and permission services
- **Content Hosting**: File and resource management
- **Session Management**: User session handling
- **Service Location**: Use the Kernel's service location mechanisms to access these services
- **New Services**: New core services should be added to the Kernel, not to individual tools
- **Existing Helpers First**: Prefer existing Sakai services/utilities before adding local helpers or new infrastructure, including `ServerConfigurationService`, `LocaleService`, `SiteService.siteReference()`, `UserDirectoryService`, `SchedulingService`, optional site lookup helpers, and Sakai's standard Jackson/XML utilities.
- **Configuration**: Put configurable behavior in Sakai configuration, with defaults in the normal default properties path when appropriate; avoid tool-local ad hoc config files.
- **Background Work**: Prefer Kernel scheduling/executor services over creating new thread pools. Verify session, user, site, and security context assumptions for work running outside a request thread.

### Web Components
- **Strategic Direction**: Web components are the strategic direction for Sakai frontend development
- **Lit Library**: Web components are built using the Lit library (lit.dev)
- **Component Creation**: Create reusable, encapsulated components with their own styling and behavior
- **Shadow DOM**: Leverage Shadow DOM for style encapsulation
- **Custom Elements**: Define custom HTML elements for Sakai-specific functionality
- **Integration**: Web components can be integrated into both new and existing tools

## Java Frameworks
- **Legacy Frameworks**: The codebase contains multiple Java frameworks from different eras
- **Spring**: Crucial framework used throughout the codebase for dependency injection, MVC, and services
- **Hibernate**: Critical ORM framework for database interactions, essential for future development
- **Persistence Semantics**: Prefer JPA operations such as `persist` and `merge` when working in JPA-oriented code. Return the managed entity from `merge`, and avoid Hibernate-specific methods unless the surrounding code intentionally uses Hibernate APIs.
- **JSF 2.3**: JavaServer Faces is used in many tools
- **Wicket**: Used in several tools for component-based web development
- **ThymeLeaf**: Preferred template engine for new development
- **Apache Velocity**: Used in older parts of the codebase
- **RSF (Reasonable Server Faces)**: Avoid using this legacy framework for new development
- **Framework Selection**: For new tools, prefer Spring MVC/Boot with Hibernate and ThymeLeaf
- **Modernization**: When making substantial changes to a tool, consider migrating to more modern frameworks

## UI Framework
- **Bootstrap**: Bootstrap 5.2 is the preferred UI framework for styling
- **Responsive Design**: Ensure all UI components work across different screen sizes
- **Components**: Leverage Bootstrap 5 components for consistent UI/UX

- **Modern JavaScript**: Target evergreen browsers; assume ES2022+ features and browser APIs like `fetch` keepalive are present.
- **Lean Code Paths**: Avoid legacy branches, UA sniffing, or fallbacks unless a specific evergreen gap is documented.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sakaiproject/sakai](https://github.com/sakaiproject/sakai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
