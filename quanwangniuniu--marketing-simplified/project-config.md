---
trigger: always_on
description: Security and privacy (no secrets in logs, validate input, least privilege).
---


# Security & privacy (always)

- **Never log secrets** (tokens, passwords, full headers).
- **Validate untrusted input** on the backend (treat everything from the client as hostile).
- **Least privilege**: permissions/roles should default to "deny" unless explicitly allowed.

---
> Source: [quanwangniuniu/marketing-simplified](https://github.com/quanwangniuniu/marketing-simplified) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
