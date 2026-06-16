---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this module is

Magento 2 implementation of the Model Context Protocol (MCP, spec version `2025-06-18`). Ships the **transport, auth, ACL, audit, and tool registry** — a single `POST /mcp` endpoint that speaks JSON-RPC 2.0 over HTTP with bearer auth. Domain tools live in satellite modules (`Magebit_McpOrderTools`, `Magebit_McpCatalogTools`, `Magebit_McpCustomerTools`, `Magebit_McpCmsTools`); this repo ships only the core `system.store.list`, `system.store.info`, `system.config.get` tools.

The repo is checked out as a Magento module at `app/code/Magebit/Mcp`. The Magento root is `/var/www/demo` — Composer, `bin/magento`, and `vendor/bin/*` all run from there, not from this directory. Read the root `README.md` for protocol-level detail and client-onboarding snippets — this file complements it with architecture and workflow notes.

## Commands

All commands run from the Magento root (`/var/www/demo`).

```bash
# Enable / rebuild after XML or DI changes
bin/magento module:enable Magebit_Mcp
bin/magento setup:upgrade
bin/magento setup:di:compile
bin/magento cache:flush

# Module-specific CLI
bin/magento magebit:mcp:tools:list            # Every registered tool, ACL, write mode
bin/magento magebit:mcp:tools:validate-acl    # CI gate: tool ACLs must resolve + names must match regex
bin/magento magebit:mcp:token:create --admin-user=<u> --name='<label>'
bin/magento magebit:mcp:token:{list,revoke,delete}

# Static analysis — PHPStan level 9 on this module
vendor/bin/phpstan analyse app/code/Magebit/Mcp -c app/code/Magebit/Mcp/phpstan.neon

# Unit tests (module-scoped)
vendor/bin/phpunit -c dev/tests/unit/phpunit.xml.dist app/code/Magebit/Mcp/Test/Unit
# Single test class
vendor/bin/phpunit -c dev/tests/unit/phpunit.xml.dist app/code/Magebit/Mcp/Test/Unit/Model/JsonRpc/DispatcherTest.php
# Single method
vendor/bin/phpunit -c dev/tests/unit/phpunit.xml.dist --filter testDispatchMethodNotFound app/code/Magebit/Mcp/Test/Unit/Model/JsonRpc/DispatcherTest.php
```

Local smoke-testing via MCP Inspector: see `dev/inspector/README.md`. For self-signed `*.docker` hostnames the Inspector needs `NODE_TLS_REJECT_UNAUTHORIZED=0` per-process.

## Request pipeline (Controller/Index/Index.php)

Every `POST /mcp` runs this fixed sequence. Every stage has an audit row flushed from the `finally` block — unauthenticated attempts leave a trail:

1. **Origin header** — `Model/Validator/OriginValidator`, DNS-rebinding defense, allowlist in store config.
2. **Bearer auth** — `Model/Auth/TokenAuthenticator`. 401 + `WWW-Authenticate: Bearer` on failure. Tokens are HMAC-SHA256 hashed at rest (keyed by the install's `crypt/key`).
3. **Body parse** — JSON-RPC envelope. Body capped at 256 KiB (`MAX_BODY_BYTES`) so unauthenticated attackers can't OOM the FPM worker pre-auth.
4. **`Mcp-Protocol-Version` header** — `Model/Validator/ProtocolVersionValidator` (required on every request *after* `initialize`; tolerates folded duplicates).
5. **Dispatch** — `Model/JsonRpc/Dispatcher` routes by method to the DI-registered handler, passing `AuthenticatedContext`.
6. **Audit** — `Model/AuditLog/AuditLogger` writes the row; `PiiRedactor` replaces sensitive fields with 16-char HMAC fingerprints keyed by `crypt/key` before write. This happens even on auth failure.

The controller bypasses layout and writes directly to the HTTP response. It implements `CsrfAwareActionInterface` to opt out of Magento's form-key CSRF — bearer auth is the CSRF gate.

## Prompts

In addition to Tools, the module ships the MCP **Prompts** primitive — a `/`-menu of pre-built workflows that surface in clients like Claude Web / Claude Desktop. Resources are intentionally not implemented (their unique value is subscriptions, which need streaming transport).

**Architecture mirrors Tools:** `Api/PromptInterface` declares the contract, `Model/Prompt/PromptRegistry` is a DI-array registry validating that the di.xml key matches `getName()` (regex same as tools), and `etc/di.xml` registers each prompt under its canonical name. Satellite modules wire their own prompts the same way.

**Handlers:** `Model/JsonRpc/Handler/PromptsListHandler` and `…/PromptsGetHandler` serve `prompts/list` and `prompts/get`. The list handler filters out write-requiring prompts when either `magebit_mcp/general/allow_writes` or the token's own `allow_writes` is off — the menu never offers an option that `prompts/get` would just reject. `Model/Prompt/PromptRenderer` does dumb `{{argument_name}}` substitution; missing required args raise `INVALID_PARAMS` before reaching the renderer, missing optional args substitute to empty string.

**Audit:** `prompts/get` writes one row to `magebit_mcp_audit_log` with `method='prompts/get'` and `prompt_name=<name>` (new column added to the schema). Argument values flow through the same `PiiRedactor` pass as tool arguments.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [magebitcom/magento2-mcp-module](https://github.com/magebitcom/magento2-mcp-module) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
