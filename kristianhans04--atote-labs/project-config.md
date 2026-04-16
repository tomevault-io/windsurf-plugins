---
trigger: always_on
description: 1. Security Best Practices
---

1. Security Best Practices

- **MANDATORY** security measures for every feature:
    - **SQL Injection Prevention:** Use parameterized queries, never string concatenation.
    - **XSS Protection:** Sanitize all user input and escape output.
    - **CSRF Protection:** Implement anti-CSRF tokens in all forms and API requests.
    - **Rate Limiting:** Enforce limits on login (e.g., 5/min) and API access (e.g., 100/hour).
    - **DDoS Protection:** Use request throttling, firewalls, and CAPTCHA for public endpoints.
    - **Input Validation:** Whitelist allowed parameters and enforce data formats.
    - **Authentication:** Secure sessions and account lockout after repeated failures.
    - **Authorization:** Always check user permissions before granting access.
    - **File Upload Security:** Validate file types, scan for malware, and restrict file sizes.
    - **Dependency Security:** Regularly audit dependencies for vulnerabilities.
    - **Content Security Policy (CSP):** Apply strict CSP headers.
    - **Secure Headers:** Implement strict HTTP security headers.
    - **API Security:** Use token-based authentication (e.g., JWT) with expiration and refresh tokens.
    - **Password Security:** Hash and salt all credentials securely.
    - **Sensitive Data Logging:** Exclude sensitive fields from logs.
    - **Session Security:** Set cookies with `secure`, `httpOnly`, and `sameSite` attributes.
    - **Directory Traversal Protection:** Sanitize and validate file paths.
    - **Mass Assignment Protection:** Explicitly control allowed data inputs.
    - **Information Disclosure Prevention:** Use custom error pages; no stack traces in production.
    - **Clickjacking Protection:** Use `X-Frame-Options` or `frame-ancestors` headers.

2. API Keys & Secrets Management

- **NEVER** commit API keys or secrets to repositories.
- Use environment variables or secure key management systems.
- Rotate keys regularly.
- Use separate keys for development, staging, and production.
- Implement least-privilege access (e.g., read-only vs. write).
- Encrypt secrets at rest and in transit.

3. Database Protection – NEVER DESTROY DATA

- **NEVER** run destructive database commands such as:
    - Dropping the database.
    - Resetting the database.
    - Reloading schema.
    - Any command that causes data loss.
- **ONLY** use safe migrations or schema updates.
- **ALWAYS** backup before major changes.
- **NEVER** truncate or delete production data.
- If data needs cleanup, use soft deletes or archiving strategies.

4. Testing & CI/CD Requirements – MANDATORY

- **MUST** write comprehensive tests for all code:
    - Unit tests for core logic and utilities.
    - Integration tests for APIs and components.
    - Functional tests for user workflows.
    - Security tests for authentication and authorization.
    - Performance tests for critical features.
- **Test Coverage Requirements:**
    - Minimum 80% overall test coverage.
    - 100% for payment and security-critical code.
- Write tests before or alongside implementation.
- **CI/CD Pipeline MUST PASS:**
    - All tests must pass before merging.
    - Code linting and style checks must pass.
    - Security scans for dependencies must pass.
    - Database migrations must be reversible.

5. Data Structures & Algorithms (DSA) Best Practices

- **Prioritize Efficiency:** Always consider the time and space complexity of your algorithms. Strive for optimal solutions.
- **Big O Notation:** Understand and use Big O notation to analyze and compare the efficiency of different approaches.
- **Data Structure Selection:** Choose data structures that fit the problem. For example:
    - Use `HashMap` (or `dict` in Python) for fast lookups (O(1) on average).
    - Use `Array` for ordered data with fast index-based access.
    - Use `Set` for storing unique elements and fast membership testing.
    - Use `Queue` for FIFO (First-In, First-Out) processing.
    - Use `Stack` for LIFO (Last-In, First-Out) processing.
- **Algorithm Optimization:**
    - Avoid nested loops that can lead to O(n^2) or worse complexity if a more optimal algorithm exists.
    - Consider trade-offs between time and space complexity. Sometimes, using more memory can lead to a faster algorithm.
    - Use sorting and searching algorithms efficiently.
- **Code Reviews:** During code reviews, pay close attention to the efficiency of the implemented algorithms and data structures.

6. Code Organization & Modern Practices

- **JavaScript or Frontend Code:**
    - Use modular architecture (imports/exports).
    - Follow ES6+ syntax and best practices.
    - Avoid inline scripts.
- **CSS/Styling:**
    - Use utility-first or modular CSS approaches.
    - Keep styles organized by component or feature.
    - Avoid inline styles unless necessary.
- **HTML/Templates:**
    - Use reusable components or partials.
    - No business logic in templates.
    - No emojis in any view.
- **Services/Modules:**
    - Follow the Single Responsibility Principle.
    - Keep clear and well-defined public interfaces.

7. Documentation Best Practices

- All documentation belongs in a dedicated `docs/` directory.
- Never place documentation files in the project root.
- **Recommended structure:**
    - `docs/` – Main documentation directory.
    - `docs/api/` – API documentation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KristianHans04) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
