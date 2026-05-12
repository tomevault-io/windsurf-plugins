---
trigger: always_on
description: * Don't be an overachiever. Focus on the task, aiming for perfection in the execution rather than in adding extras.
---

# AGENTS.md

* Don't be an overachiever. Focus on the task, aiming for perfection in the execution rather than in adding extras.
* Implement the solution using many small, isolated, single-purpose files. Your primary goal is to minimize the Lines of Code (LoC) in each file. Code duplication is explicitly allowed to maintain this structure.
* Follow the Architectural Governance and Project Requirements Change process.
* Use `deepwiki` MCP for knowledge about specific package implementations details when needed.
* Use the file `.deepwiki` as a repository for relevant DeepWiki repositories.
* Continuously keep the `.gitignore` accurate to not bloat the repository.
* `CHANGELOG.md` is managed by release-please and **MUST NOT** be manually updated.

## Project Structure

The project follows the Go standard project layout (see CR-0021):

```
outlook-mcp/
  cmd/
    outlook-local-mcp/
      main.go                  # Entry point: config load, subsystem init, lifecycle
  internal/
    config/                    # Config struct, LoadConfig, ValidateConfig
    auth/                      # Browser/device code auth, token cache, auth record, account registry, account resolver
    logging/                   # InitLogger, SanitizingHandler, PII masking, MultiHandler, file logging
    audit/                     # Audit logging subsystem, AuditWrap middleware
    graph/                     # Graph API utilities: errors, retry, timeout, serialization, enums, recurrence
    validate/                  # Input validation helpers
    observability/             # OpenTelemetry metrics and tracing, WithObservability middleware
    server/                    # RegisterTools, ReadOnlyGuard, AwaitShutdownSignal
    tools/                     # 4 aggregate domain tools dispatching verb sets: calendar (15 verbs), mail (5-13 verbs gated by MailEnabled/MailManageEnabled), account (7 verbs), system (5-6 verbs gated by auth_code: status, complete_auth, help, list_docs, search_docs, get_docs); see CR-0056, CR-0058, CR-0060, CR-0061
  docs/
    ...
```

**Build:** `go build ./cmd/outlook-local-mcp/`
**Install:** `go install github.com/desek/outlook-local-mcp/cmd/outlook-local-mcp@latest`
**New code** must be placed in the appropriate `internal/` package, not the repository root.

## Documentation Standards

All code **MUST** be extensively documented using Go doc comments:

* **Every package**: Include a package-level docstring in a `doc.go` file or the main package file describing the package's purpose and how it fits into the system.
* **Every function/method**: Include a docstring describing:
    * **Purpose**: What the function does and why it exists.
    * **Parameters**: Meaning of each parameter.
    * **Return value**: What is returned.
    * **Side effects**: Any mutations, API calls, or state changes.
    * **Errors**: Conditions under which an error is returned.
* **Every struct/interface**: Include a docstring describing the type's role and intent.
* **Every exported field**: Document the purpose and intent of each exported field.
* **Complex logic**: Add inline comments to explain non-obvious algorithms or business rules. Reference related ADRs where applicable.

### Docstring Style

* Use standard Go doc comment style (start with the name of the symbol).
* Focus on **intent and purpose** — explain *why*, not just *what*.
* Keep comments concise but complete.
* Update docstrings whenever the implementation changes.

## Design Principles

All code **MUST** adhere to the following design principles consistently:

* **SOLID**:
    * **Single Responsibility**: Each package, struct, or function must have one reason to change.
    * **Open/Closed**: Code must be open for extension but closed for modification.
    * **Liskov Substitution**: Interfaces should be satisfied by types without altering correctness.
    * **Interface Segregation**: Prefer small, focused interfaces over large, general-purpose ones.
    * **Dependency Inversion**: Depend on abstractions (interfaces), not concretions.
* **Composition over Inheritance**: Use embedding and composition to build complex types.
* **DRY (Don't Repeat Yourself)**: Extract shared logic into reusable abstractions. Note: code duplication for file isolation is acceptable in Go if it avoids unnecessary package dependencies.
* **KISS (Keep It Simple, Stupid)**: Choose the simplest solution. Avoid over-engineering and premature abstraction.
* **Law of Demeter**: A unit should only talk to its immediate collaborators.

## MCPB Extension Manifest

All MCP tools **MUST** be registered in `extension/manifest.json` under the `tools` array. When adding or removing a tool in `internal/server/server.go`, the corresponding entry in the manifest **MUST** be updated to match. The manifest is used by Claude Desktop to discover available tools.

## Tool Naming Convention

As of CR-0060 (v0.6.0) the MCP surface is four aggregate domain tools, each dispatched by a required `operation` verb. New work **MUST** add a verb to the appropriate domain registry, not a new top-level MCP tool.

Aggregate tools and their domains:

* `calendar` -- Calendar and event verbs
* `mail` -- Mail message, folder, and draft verbs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [desek/outlook-local-mcp](https://github.com/desek/outlook-local-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
