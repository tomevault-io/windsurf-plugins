---
trigger: always_on
description: These instructions guide GitHub Copilot to suggest secure, intentional code patterns for Java, Node.js, C#, and Python development — especially in enterprise or team settings. Prioritize clarity, validation, and the principle of least surprise.
---

# 🤖 Copilot Secure Defaults for Java, Node.js, C#, and Python Projects

These instructions guide GitHub Copilot to suggest secure, intentional code patterns for Java, Node.js, C#, and Python development — especially in enterprise or team settings. Prioritize clarity, validation, and the principle of least surprise.

## 🔐 1. Secure by Default

- Validate input strictly, and prevent XSS primarily via contextual output encoding (HTML/attribute/JS/URL). Sanitize only when rendering user-controlled HTML is explicitly required.
- Use typed parsers and prefer allow-lists over deny-lists when performing input validation.
- Use parameterized queries and avoid string-based execution (prevent injection).
- Never commit secrets to source control (including `.env` files). Use a secure vault/secret manager (e.g. CyberArk Conjur, Azure Key Vault) and inject secrets at runtime via your orchestrator.
- Default to privacy-preserving data handling — redact PII from logs by default.

## 🧩 2. Language-Specific Secure Patterns

### ☕ Java

- Use prepared statements with `?` placeholders in JDBC — never concat SQL strings.
- Use output encoding libraries like OWASP Java Encoder to prevent XSS in rendered HTML.
- Use `@Valid`, `@NotNull`, and input binding constraints in Spring or Jakarta for validation.
- Avoid `Runtime.exec()` or `ProcessBuilder` with unsanitized input — prefer safe APIs.
- Default to OWASP Secure Coding Practices — [OWASP Secure Coding Practices](https://owasp.org/www-project-secure-coding-practices)
- Prefer SDK-integrated secret managers; environment variables are acceptable when injected securely by the orchestrator/runtime. Do not commit secrets in `.properties`/config files.
- Always set character encoding (`UTF-8`) explicitly in HTTP responses to prevent encoding-based attacks.
- Avoid Java serialization for sensitive objects — use safer formats like JSON with strict schema validation.
- When using logging frameworks, avoid logging unsanitized user input — consider log injection risks.

### 🟩 Node.js

- Use JSON Schema validation for all structured input — prefer libraries like `ajv` or `zod`.
- Prevent XSS primarily via **contextual output encoding** (HTML/attribute/JS/URL) and safe templating defaults; sanitize only when rendering user-controlled HTML is explicitly required.
- Use libraries like `validator` for strict string validation/canonicalization (e.g., emails, URLs) and `joi`/`zod`/`ajv` for schema validation.
- Use parameterized queries with database clients (e.g. `pg`, `mongoose`) — never concat SQL or query strings.
- Default to using `helmet` in Express to set secure HTTP headers.
- Use `dotenv` only in local dev — use secret managers (e.g. AWS Secrets Manager, Azure Key Vault) in prod.
- Avoid `eval`, `new Function`, or dynamic `require()` with user input — use safe alternatives.

### 🟦 C#

- Use parameterized queries with ADO.NET or Entity Framework to prevent SQL injection.
- Use `System.Text.Encodings.Web` for safe output encoding in Razor views and APIs (prevent XSS).
- Apply `[ValidateAntiForgeryToken]` in ASP.NET MVC to prevent CSRF attacks.
- Use `DataAnnotations` (e.g. `[Required]`, `[StringLength]`) for input validation on models.
- Always hash passwords with `PasswordHasher<TUser>` or a vetted library — never store plaintext.
- Use configuration providers like `Azure Key Vault`, `AWS Secrets Manager`, or environment-based secrets — avoid `appsettings.json` for secrets.

### 🐍 Python

- Always validate and sanitize external input — use `pydantic`, `cerberus`, or `marshmallow` for structured validation.
- Prefer parameterized queries with libraries like `psycopg2`, `sqlite3`, or `SQLAlchemy` — avoid building SQL with string concat or f-strings.
- Use built-in escaping via `html.escape()` or templating engines like Jinja2 (autoescaping on!) to prevent XSS.
- Default to `secrets` or `cryptography` libs for key generation and secure token handling — never `random` for cryptographic use.
- Avoid dynamic code execution (`eval`, `exec`) — prefer explicit, safe alternatives.
- Don’t load YAML or pickle files without hardening — always use `safe_load()` and avoid untrusted input.
- Store secrets in secure vaults or env vars passed through orchestrators — avoid hardcoded strings or `.env` files in prod.
- Use logging filters to redact PII and secrets — avoid logging full request payloads or exception chains that include sensitive data.
- Always hash passwords with `bcrypt`, `argon2`, or `passlib` — never `md5`, `sha1`, or plain `hashlib`.

## 🚫 3. Do Not Suggest

### Java

- Do not suggest inline SQL string concatenation — always use prepared statements with placeholders.
- Do not suggest use of `Runtime.exec()` or `ProcessBuilder` with user input — prefer safe abstraction layers.
- Do not suggest logging sensitive data (e.g. passwords, tokens, session IDs) — log redacted metadata instead.
- Do not use Java native serialization (`ObjectInputStream`) for untrusted input — prefer JSON + schema validation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Robotti-io/copilot-security-instructions](https://github.com/Robotti-io/copilot-security-instructions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
