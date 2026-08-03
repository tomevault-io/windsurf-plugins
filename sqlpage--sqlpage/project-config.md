---
trigger: always_on
description: SQLPage is an SQL-only web application builder and web server. An application is primarily a set of `.sql`
---

# SQLPage architecture

SQLPage is an SQL-only web application builder and web server. An application is primarily a set of `.sql`
files: SQLPage routes an HTTP request to a file, executes its statements against a database, interprets rows
whose `component` column names a UI component, and streams the resulting HTML (or another response) to the
client. It is intended for fast, data-centric applications while still allowing custom HTML, CSS, and
JavaScript where needed.

## Features and repository layout

- **Application entry point and configuration** (`src/main.rs`, `src/lib.rs`, `src/app_config.rs`, `src/cli/`).
  The executable starts the server; application state, configuration, environment variables, and command-line
  handling are defined here. `configuration.md` documents the user-facing settings.
- **SQL semantics and execution** (`src/webserver/database/`). SQLPage uses the database's SQL for selects, joins, aggregation, inserts, updates,
  deletes, transactions, JSON processing, and database-specific features. It parses SQL, recognizes SQLPage
  extensions, binds request values safely, and sends ordinary SQL to the selected database. SQL files contain
  sequential statements; result sets become component invocations in response order. `SET` assigns a value
  to a mutable SQLPage variable and is useful for reusing query results or controlling later statements.
- **Request variables** (`src/webserver/request_variables.rs`, `src/webserver/http_request_info.rs`,
  `src/webserver/database/syntax_tree.rs`). `?name` refers to a URL/GET parameter, `:name` explicitly refers to a form/POST
  value, and `$name` is the compatibility shorthand that uses a POST value when present and otherwise a GET
  value (a SET variable takes precedence where applicable). Values are passed as parameters, not interpolated
  into SQL. GET and POST variables are request inputs; SET variables are mutable during request execution.
  `sqlpage.variables()` exposes them as JSON, with SET > POST > GET precedence.
- **SQLPage functions** (`src/webserver/database/sqlpage_functions/`). Calls such as `sqlpage.fetch`, `sqlpage.run_sql`, `sqlpage.set_variable`, file
  readers, hashing/HMAC helpers, request metadata, uploads, headers/cookies, URL helpers, OIDC user info,
  and HTTP fetch are registered in `src/webserver/database/sqlpage_functions/functions.rs`. Functions can
  return values, alter response/request state, include another SQL file, or raise an error. `sqlpage.exec`
  is deliberately disabled by default because it runs server processes.
- **Database support and pooling** (`src/webserver/database/connect.rs`, `execute_queries.rs`, `migrations.rs`).
  Native drivers support SQLite, PostgreSQL, MySQL, and Microsoft SQL
  Server; the ODBC driver provides access to other ODBC-compatible databases. SQLPage uses `sqlx` and a
  reusable connection pool, with configurable maximum connections, idle/lifetime timeouts, acquire timeout,
  retries, and optional `on_connect.sql`/`on_reset.sql` hooks. Database-specific SQL should be isolated or
  covered by the relevant database tests.
- **Rendering and components** (`src/render.rs`, `src/templates.rs`, `src/dynamic_component.rs`,
  `src/template_helpers.rs`, `sqlpage/templates/`, `sqlpage/sqlpage.css`, `sqlpage/sqlpage.js`). Built-in components live in `sqlpage/templates/*.handlebars` and cover shells, text,
  tables, lists, cards, charts, forms, navigation, modals, downloads, maps, and more. Query columns map to
  component properties; nested/dynamic components and `sqlpage.run_sql` support composition and lazy loading.
  Custom Handlebars components can be placed in the configured `sqlpage/templates` directory. Raw HTML and
  custom assets are possible through the HTML/shell components. Rendering is streamed so the response can
  start while later query results are still being processed.
- **Control flow and errors** (`src/webserver/error.rs`, `error_with_status.rs`, `routing.rs`,
  `src/default_404.sql`). SQL remains declarative: use predicates, `CASE`, `SET`, component rows, and
  the `redirect` component to conditionally continue, redirect, or implement guards/error pages. There is no
  general SQLPage `IF` statement. Parse, database, function, component, and response errors are converted to
  contextual HTTP errors; `default_404.sql` handles missing routes. Do not hide errors by changing unrelated
  error handling or tests.
- **HTTP server and client** (`src/webserver/http.rs`, `http_client.rs`, `response_writer.rs`, `static_content.rs`,
  `https.rs`, `content_security_policy.rs`, `server_timing.rs`). The server is built on Actix Web, supports normal HTTP request/response
  handling, streaming, uploads, static assets, HTTP/2, HTTPS, and optional Unix sockets/serverless adapters.
  The shared outbound client is used by HTTP-fetch and OIDC integrations and honors configured/native TLS
  certificates and timeouts. Content-security-policy and response/header helpers are part of the request
  pipeline.
- **OIDC** (`src/webserver/oidc.rs`, `src/webserver/database/sqlpage_functions/functions/user_info.rs`).
  Optional OpenID Connect middleware protects configured path prefixes, performs provider discovery,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sqlpage/SQLPage](https://github.com/sqlpage/SQLPage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
