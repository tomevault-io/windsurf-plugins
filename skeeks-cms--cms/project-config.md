---
trigger: always_on
description: This is the shared `skeeks/cms` package: a reusable Yii2 content-management core consumed by many SkeekS projects. Changes here affect every consuming project, so keep package-level behavior conservative, reusable and backward-aware.
---

# SkeekS CMS

This is the shared `skeeks/cms` package: a reusable Yii2 content-management core consumed by many SkeekS projects. Changes here affect every consuming project, so keep package-level behavior conservative, reusable and backward-aware.

## Package role

- The package provides CMS models, controllers, migrations, widgets, components and configuration under `src/`.
- Site structure and pages are represented by CMS tree models; publications, news, products and similar records are content elements.
- Content types and their dynamic properties allow projects to add fields through administration without creating a dedicated model for every content kind.
- User access is governed by the existing CMS/RBAC mechanisms.
- The package is a Yii2 extension configured through `src/config/common.php`, `src/config/web.php` and `src/config/console.php`.

Official links:

- Website: `https://cms.skeeks.com`
- Source: `https://github.com/skeeks-cms/cms`
- Issues: `https://github.com/skeeks-cms/cms/issues`

## Architecture and engineering rules

- Keep browser and administration behavior in backend/admin controllers.
- Keep protocol and API integrations separate from backend UI controllers.
- Put shared business behavior and write operations in services or models.
- Keep transport controllers narrow: parse requests, authenticate, authorize and format responses.
- Do not duplicate model validation or bypass Yii validation, permissions, ownership checks or transactions.
- Prefer existing package and project conventions over generic Yii assumptions.
- Avoid unrelated refactors in this shared package.
- Inspect current implementations and usages before changing public model or component contracts.

## Important CMS domains

Use `ast-index` to discover the current definitions and contracts instead of relying on memory. Common starting points are:

- `CmsTree` for site sections and pages;
- `CmsContent` for configured content containers;
- `CmsContentType` for section/content type metadata;
- `CmsContentElement` for publications and other content records;
- storage-file models and services for uploaded files;
- task models and services for CRM task behavior.

The exact namespace, active implementation and extension points must be verified before editing.

## MCP and OAuth ownership

MCP and OAuth are deliberately outside this package:

- `skeeks/cms-mcp` owns the MCP endpoint, tool contracts, providers and MCP-specific application services.
- `skeeks/cms-oauth2-server` owns OAuth controllers, models, migrations, resources and scopes.
- Do not add MCP or OAuth transport code back to `skeeks/cms`.
- Read the target package's `AGENTS.md` completely before changing MCP or OAuth behavior.

For MCP, tools must remain thin and delegate validation, model access, transactions and business rules to services. There are no delete tools: create drafts first and publish explicitly.

For a site with `skeeks/cms-mcp` installed, the canonical endpoint is `https://<site-domain>/cms/mcp`. In Codex, both `url` and `oauth_resource` must contain that exact site-specific URL. Never configure the obsolete `/cms/mcp-task/create` route. Use the actual target site's public domain rather than assuming `skeeks.com`.

The same authorized tools are available to non-MCP AI clients at
`https://<site-domain>/cms/rest-api`. The catalog has three identifiers:
`api_version`, `server_version` and the OAuth/RBAC-specific `tools_revision`.
Persist the `/tools` response outside the chat and revalidate its private ETag;
reuse cached schemas after `304 Not Modified`. Do not download and reinterpret
the complete catalog for every user request. Use `/tools/index`,
`/tools/{tool_name}` or the `prefix`, `names` and `q` catalog filters when only
a small part of an uncached inventory is needed. Never share catalog caches
between OAuth credential stores.
The Windows REST helper is owned by `skeeks/cms-mcp` at
`scripts/skeeks-rest.ps1`; this core package only documents how agents locate
and invoke the installed helper.
Common reads should use the package's `scripts/skeeks-api.ps1` direct-first
client. Known operations execute immediately; catalog or schema discovery is a
fallback after an unknown-tool or validation error, not a mandatory preflight.

When the user asks to create a SkeekS CMS task through MCP, use `create_cms_task`. Default `executor_id` is `1` unless the user specifies another executor.

## AI-assisted content direction

CMS-facing services may support AI clients that:

- inspect site, theme, content-type and field metadata;
- create or edit sections and pages;
- create or edit publications;
- upload or select files and insert their URLs into content;
- validate drafts and publish explicitly;
- return a public URL for verification.

Preserve existing content structure unless the user requests a rewrite. Resolve explicit entities for updates, validate dynamic fields for the selected content type, and ask the user when a required type or required field cannot be inferred safely.

## Code search


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skeeks-cms/cms](https://github.com/skeeks-cms/cms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
