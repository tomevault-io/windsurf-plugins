---
trigger: always_on
description: These instructions are the concise, must-follow rules for working in SRAT. Keep changes minimal, readable, and aligned with existing patterns.
---

<!-- DOCTOC SKIP -->

# SRAT Copilot Instructions

These instructions are the concise, must-follow rules for working in SRAT. Keep changes minimal, readable, and aligned with existing patterns.

## Non‑negotiable rules

- **Read the file header first**: Always read the top comment/header of any file you modify; file‑specific rules override everything else.
- **No git writes**: Never run `git add/commit/push` unless the user explicitly asks.
- **Follow instruction files**: Use the specialized guidance in `.github/instructions/` for Go, Python, React, Markdown, frontend tests, and backend command execution.
- **Mandatory for backend execution**: When implementing or migrating backend command execution, always follow `.github/instructions/backend-command-execution.instructions.md`.
- **Ask for clarification**: If a user request is ambiguous or could lead to unintended consequences, ask for clarification before proceeding.
- **Respect existing code**: Follow the established architecture, style, and patterns of the codebase. Avoid introducing new abstractions or styles unless necessary.
- **Prioritize maintainability**: Write clear, readable code that other developers can easily understand and maintain. Avoid clever or complex solutions when a straightforward approach will do.
- **Add tests**: When fixing bugs or adding features, include tests that cover the new behavior and edge cases. Follow the testing guidelines in the instruction files.
- **Test your changes**: Always run the relevant tests after making changes to ensure you haven't introduced regressions. Follow the testing guidelines in the instruction files.
- **Document your changes**: If your change affects the behavior of the system, update the relevant documentation and add comments to your code where necessary to explain non-obvious logic or decisions.
- **Verify before finalizing**: Before finalizing any code changes, review your work to ensure it adheres to the above rules and the specific guidelines in the instruction files. If you're unsure about any aspect of your changes, ask for a review or feedback from a human developer. Always aim for high-quality, maintainable code that aligns with the project's standards and goals.
- **Commit precheck**: Ensure that all pre-commit checks (linters, formatters, security scanners) pass before finalizing your changes. If any checks fail, address the issues and re-run the checks until they pass successfully.

## Repo at a glance

- **Languages**: Go 1.26 back-end, TypeScript React frontend (Bun), Python 3.12+ Home Assistant integration.
- **Architecture**: API handlers → services → generated GORM helpers → SQLite (embedded). Frontend uses MUI + RTK Query. Custom component is WebSocket‑only.

## back-end (Go) essentials

- Use **context‑aware logging** (`slog.*Context`, `tlog.*Context`) when a real `context.Context` is already in scope. Never manufacture a context for logging.
- Go 1.26 rules: use `new(expr)` for pointer values, use `any` (not `interface{}`), use `WaitGroup.Go`, prefer `errors.AsType[T]` (standard library).
- Prefer direct persistence in services using `dbom` + GORM (and generated query helpers when available) over introducing new per-entity repository layers, unless a clear documented exception is required.
- Use **generated converters** (`converter.<Type>ToDtoConverterImpl{}` from `converter/`) for all DTO↔DBOM mapping in services. Never write manual `toDTO`/`toDBOM` helper functions — they diverge silently from the generated impl.
- Do **not** edit vendored code unless using the patch workflow (`backend/patches/` + `mise run //backend:patch`).

## Frontend essentials

- Use Bun toolchain (`frontend/`). Build outputs go to `backend/src/web/static`.
- **Do not** edit `frontend/src/store/sratApi.ts` or `backend/docs/openapi.*` directly—update Go and run `cd frontend && bun run gen`.
- **Never** manually add types to `frontend/src/store/wsApi.ts`. All types must come from `sratApi.ts`. WS-only event payload types that have no REST endpoint need a doc-stub handler in `backend/src/api/system.go` (tagged `"system","internal"`). 

  **Doc-Stub Pattern:** A handler that always returns an error but declares the DTO(s) in its response type signature. This anchors the types into the OpenAPI schema, which code-generates them into frontend TypeScript. Example:
  
  ```go
  // HandleCommandEvents is a documentation-only stub that anchors command event schemas
  // into the OpenAPI spec so they code-generate into TypeScript types.
  // Actual command events are delivered over WebSocket.
  func (s *SystemHandler) HandleCommandEvents(ctx context.Context, input *struct{}) (*struct {
    Body struct {
      Started    *dto.CommandStartedNotification    `json:"started,omitempty"`
      Output     *dto.CommandOutputNotification     `json:"output,omitempty"`
      Terminated *dto.CommandTerminatedNotification `json:"terminated,omitempty"`
    }
  }, error) {
    return nil, huma.Error500InternalServerError("Use WebSocket for events", nil)
  }
  ```
  
  After adding a doc-stub, run `cd frontend && bun run gen` to code-generate the types into `sratApi.ts`, then import and use them in `wsApi.ts` or other files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dianlight/srat](https://github.com/dianlight/srat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
