---
trigger: always_on
description: enableSSL: boolean;
---

# Claude Development Guidelines for Obsidian MCP Plugin

## Project Context

This is a hybrid Obsidian plugin that combines:
- **Local REST API functionality** (from coddingtonbear's plugin)
- **Semantic MCP operations** (from aaronsb/obsidian-semantic-mcp)
- **Direct Obsidian API integration** for enhanced performance

The critical architectural pattern is **preserving the ObsidianAPI abstraction layer** while replacing HTTP calls with direct Obsidian plugin API calls. This allows reuse of all existing MCP server logic while gaining performance benefits.

## Code Quality Guidelines

### SOLID Principles Application

- **Single Responsibility**: 
  - `ObsidianAPI` class handles only vault operations abstraction
  - `MCPServer` class handles only MCP protocol operations
  - `HTTPServer` class handles only REST endpoint management
  - Plugin main class handles only Obsidian plugin lifecycle

- **Open/Closed**: 
  - ObsidianAPI interface remains stable for extensions
  - MCP operations extensible through semantic router pattern
  - HTTP endpoints extensible without modifying core logic

- **Liskov Substitution**: 
  - New ObsidianAPI implementation must be drop-in replacement
  - All method signatures and return types must match exactly
  - Error handling behavior must be preserved

- **Interface Segregation**: 
  - Separate interfaces for vault operations, search operations, and workspace operations
  - MCP protocol separated from HTTP REST protocol
  - Plugin settings separated from server configuration

- **Dependency Inversion**: 
  - Depend on Obsidian Plugin API abstractions, not concrete implementations
  - MCP server depends on ObsidianAPI interface, not specific implementation
  - HTTP server depends on operation interfaces, not direct vault access

### Architecture Patterns

#### Critical Abstraction Layer
```typescript
// This interface MUST remain stable
interface IObsidianAPI {
  getFile(path: string): Promise<ObsidianFileResponse>;
  listFiles(directory?: string): Promise<string[]>;
  searchSimple(query: string): Promise<any[]>;
  // ... all existing methods preserved
}

// Implementation changes from HTTP to direct API
class ObsidianAPI implements IObsidianAPI {
  constructor(private app: App) {} // Direct plugin access
  
  async getFile(path: string): Promise<ObsidianFileResponse> {
    // Direct vault access instead of HTTP call
    const file = this.app.vault.getAbstractFileByPath(path);
    // ... implementation
  }
}
```

#### Performance-Critical Patterns
- **Caching Layer**: Implement intelligent caching for frequently accessed files
- **Lazy Loading**: Load heavy operations only when needed
- **Batch Operations**: Combine multiple vault operations where possible
- **Memory Management**: Proper cleanup of file handles and event listeners

#### Error Handling Patterns
```typescript
// Preserve exact error types and messages from HTTP implementation
class VaultError extends Error {
  constructor(message: string, public code: string, public status: number) {
    super(message);
  }
}

// Maintain compatibility with existing error handling
async getFile(path: string): Promise<ObsidianFileResponse> {
  try {
    const file = this.app.vault.getAbstractFileByPath(path);
    if (!file) {
      throw new VaultError(`File not found: ${path}`, 'ENOENT', 404);
    }
    // ... rest of implementation
  } catch (error) {
    // Transform plugin errors to match HTTP API errors
    throw this.transformError(error);
  }
}
```

## Development Workflow

### BRAT Development Release Process

When developing with Obsidian BRAT (Beta Reviewer's Auto-update Tool) for plugin side-loading:

#### Development vs Release Workflow

**Development (no releases created):**
- Push commits to main freely - no automatic releases triggered
- Test locally with `npm run build`
- Iterate as needed without polluting release history

**Creating a Release (manual trigger):**

Via GitHub UI:
1. Go to **Actions** → **Create Release**
2. Click **Run workflow**
3. Optionally add release notes
4. Click **Run**

Via CLI:
```bash
# Simple release
gh workflow run release.yml

# With release notes
gh workflow run release.yml -f release_notes="Fixed VS Code compatibility, improved search"
```

#### Version Updates
- **ONLY update `package.json` version** - `sync-version.mjs` automatically syncs to `manifest.json` and `version.ts`
- DO NOT manually update `manifest.json` - the automation handles this
- Bump version in `package.json` before triggering a release

#### Tag Management
- Release workflow creates tags automatically (no 'v' prefix)
- If a tag already exists for that version, the release is skipped
- To re-release same version, delete existing tag first:
  ```bash
  git tag -d X.Y.Z && git push origin :refs/tags/X.Y.Z
  ```

#### BRAT User Experience
- Users install via: `aaronsb/obsidian-mcp-plugin` in BRAT
- BRAT checks GitHub releases for updates automatically  
- Version detection relies on `manifest.json` version field
- Release assets (main.js, manifest.json, styles.css) are auto-generated by workflow

#### Version Naming Convention
- **Major releases**: `X.Y.Z` (e.g., 0.4.4) - NO 'v' prefix
- **Patch releases**: `X.Y.Za`, `X.Y.Zb` (e.g., 0.4.4a, 0.4.4b) - NO 'v' prefix
- **Pre-releases**: All marked as prerelease until stable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aaronsb/obsidian-mcp-plugin](https://github.com/aaronsb/obsidian-mcp-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
