---
trigger: always_on
description: Guidance on working with this PurelyMail MCP Server project.
---

Guidance on working with this PurelyMail MCP Server project.

## Project Overview
MCP (Model Context Protocol) server for PurelyMail email service. Exposes PurelyMail's API as tools for AI assistants using TypeScript and swagger-codegen for type safety.

## Tech Stack
- **Runtime**: Node.js 20 on Nix
- **Language**: TypeScript 5.3+ with ES modules
- **MCP SDK**: @modelcontextprotocol/sdk 0.6+
- **API Client**: openapi-fetch with openapi-typescript (lightweight, type-safe)
- **Package Manager**: npm
- **Testing**: Vitest
- **Environment**: Nix flakes for reproducible builds

## Critical Files
- `swagger-spec.js` - Original PurelyMail swagger spec (fetched from https://news.purelymail.com/api/swagger-spec.js as of September 7, 2025)
- `purelymail-api-spec.json` - Swagger specification (source of truth)
- `endpoint-descriptions.md` - Generated endpoint documentation
- `src/types/purelymail-api.ts` - Generated TypeScript types (DO NOT EDIT - regenerated from swagger)
- `flake.nix` - Nix environment and dependencies
- `.mcp.json` - Claude Code MCP client configuration

## Known Issues

See `docs/KNOWN_ISSUES.md` for detailed documentation of current limitations and workarounds.

## Release Management

See `docs/versioning-guidelines.md` for semantic versioning practices and release procedures.

### Version Update Script

Use `scripts/update-version.sh` to update versions across `package.json` and `flake.nix`:

**Interactive Mode** (with gum UI):
```bash
./scripts/update-version.sh 2.0.0
nix run .#update-version -- 2.0.0
```
- Shows beautiful colored UI
- Previews diff before applying
- Offers to create git tag
- Safe with automatic rollback on errors

**Non-Interactive Mode** (for automation):
```bash
./scripts/update-version.sh -y 2.0.0
```
- Outputs "No changes needed" or diff
- Never creates git tags (used by GitHub Actions)
- Suitable for CI/CD pipelines

**Automated Release Process**:
1. Update version: `./scripts/update-version.sh 2.0.0`
2. Push tag: `git push origin v2.0.0`
3. GitHub Actions automatically publishes to npm

## Commands
```bash
# Development
nix develop              # Enter dev environment
npm run fetch:swagger    # Download latest swagger spec from PurelyMail
npm run generate:types   # Regenerate TypeScript types from swagger
npm run generate:docs    # Update endpoint-descriptions.md
npm run update:api       # Complete API update (fetch + generate types + docs)
npm run dev             # Run server in development mode
npm run test:mock       # Test with mock data
npm run inspector       # Launch MCP Inspector

# Version Management
npm run version:update 2.0.0            # Update version (npm script)
nix run .#update-version -- 2.0.0       # Update version (via nix)
./scripts/update-version.sh 2.0.0       # Update version (direct script)
./scripts/update-version.sh -y 2.0.0    # Update version (non-interactive)

# Building
nix build              # Build production package
npm run build          # Compile TypeScript only
```

## Code Conventions

### TypeScript
- Use ES module imports (`import/export`), NOT CommonJS
- Explicit return types for all functions
- Prefer interfaces over type aliases for objects
- Use `.js` extensions in imports (even for TS files)

### File Structure
```
src/
  index.ts              # Entry point - minimal logic
  tools/
    tool-generator.ts   # Generates tools from swagger
  mocks/
    mock-client.ts     # Mock implementation
  tests/
    *.test.ts          # Vitest tests
generated-client/       # DO NOT MODIFY - codegen output
```

### Error Handling
- Never throw raw strings - use Error objects
- Include operation context in errors: `new Error(\`Failed to execute ${operationId}: ${details}\`)`
- Log errors to stderr, not stdout (stdout is for MCP protocol)

## Development Workflow

### Updating from PurelyMail API Changes

**Manual Updates:**
1. Run `npm run update:api` to fetch latest spec and regenerate everything
2. Test with `MOCK_MODE=true npm run inspector` to verify tool registration  
3. Update mock responses in `mock-client.ts` if new endpoints were added
4. Test with real API to ensure compatibility

**Automated Updates:**
- GitHub Actions runs twice weekly (Tuesday/Friday) to check for API changes
- Creates PR automatically if changes are detected
- Includes regenerated types and documentation
- Review checklist provided in PR description

### Testing Changes
1. Always test with mocks first: `npm run test:mock`
2. Use MCP Inspector to validate tool registration
3. Test each tool's input/output schema
4. Only test with real API after mocks work

### Debugging
- Use `console.error()` for debug output (NOT console.log)
- Set `DEBUG=mcp:*` for protocol debugging
- Check `generated-client/` for actual API method signatures

## API Integration Rules

### Authentication
- API key from `PURELYMAIL_API_KEY` environment variable (use `${PURELYMAIL_API_KEY}` syntax in `.mcp.json`)
- Never commit API keys
- Support both mock and real modes

### Tool Design (v2.0+ Architecture)
- **One tool per operation**: Each OpenAPI operation maps to a dedicated MCP tool
- **Tool naming**: Convert operationId to snake_case (e.g., "Create User" → "create_user")
- **No action parameter**: Tool name directly indicates the action

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gui-wf/purelymail-mcp-server](https://github.com/gui-wf/purelymail-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
