---
trigger: always_on
description: **ALWAYS follow these instructions first and only fallback to additional search or context gathering if the information here is incomplete or found to be in error.**
---

# ChurnFlow MCP Server - GitHub Copilot Instructions

**ALWAYS follow these instructions first and only fallback to additional search or context gathering if the information here is incomplete or found to be in error.**

ChurnFlow MCP is a TypeScript-based Model Context Protocol server that provides ADHD-friendly productivity capture and routing capabilities to AI assistants. It manages markdown tracker files with YAML frontmatter and uses OpenAI GPT-4 for intelligent routing.

## Essential Setup Commands

**CRITICAL**: Always run these setup steps first on a fresh clone:

```bash
# Install dependencies - takes ~3-17 seconds
npm install

# Build TypeScript to dist/ - takes ~3 seconds, NEVER CANCEL
npm run build

# Test the build - takes ~5-9 seconds, NEVER CANCEL
npm test
```

## Required Configuration

Before the application will work, you MUST create a proper configuration:

1. **Create test configuration for development/testing**:
```bash
# Create test directories
mkdir -p /tmp/churn-test/Collections /tmp/churn-test/Tracking

# Create test crossref
echo '[{"tag":"test-tracker","trackerFile":"/tmp/churn-test/Tracking/test-tracker.md","collectionFile":"/tmp/churn-test/Collections/test.md","priority":1,"contextType":"project","active":true}]' > /tmp/churn-test/Tracking/crossref.json

# Create test tracker
cat > /tmp/churn-test/Tracking/test-tracker.md << 'EOF'
---
tag: "test-tracker"
friendlyName: "Test Tracker"
collection: "test"
contextType: "project"
mode: "active"
iterationType: "weekly"
active: true
priority: 1
---

# Test Tracker

## Action Items
- [ ] #task Example task

## Activity Log
- 2024-01-01: Created test tracker

## References
- Test reference
EOF

# Create working test config
cat > churn.config.json << 'EOF'
{
  "collectionsPath": "/tmp/churn-test/Collections",
  "trackingPath": "/tmp/churn-test/Tracking", 
  "crossrefPath": "/tmp/churn-test/Tracking/crossref.json",
  "aiProvider": "openai",
  "aiApiKey": "test-key",
  "confidenceThreshold": 0.7
}
EOF
```

2. **For OpenAI functionality**: Set `export OPENAI_API_KEY=your-actual-key`

## Build and Test Commands

**TIMING NOTE**: All build commands are fast. NEVER CANCEL any of these operations.

```bash
# Build - ~3 seconds
npm run build

# Test suite - ~5-9 seconds, 161 tests across 7 suites
npm test

# Test with coverage - ~8-12 seconds
npm run test:coverage

# Development watch mode
npm run dev

# Lint (currently missing config) - EXPECTED TO FAIL
npm run lint  # Will fail: "ESLint couldn't find a configuration file"

# Format code (works) - ~1 second  
npm run format  # Formats all TypeScript files with prettier
```

## Running the Applications

### CLI Application

```bash
# Initialize configuration (creates sample churn.config.json)
npm run cli init

# Check system status
npm run cli status

# Test capture (requires OPENAI_API_KEY for full functionality)
npm run cli capture "Test task to capture"

# Show help
npm run cli

# Brain dump mode (interactive)
npm run cli dump
```

### MCP Server (for AI Assistants)

```bash
# Start MCP server - starts instantly
npm run mcp

# Alternative: run directly with tsx
tsx src/index.ts

# Production mode (requires build first)
npm start
```

## Testing and Validation

### Quick Validation Workflow

Always run these steps after making changes:

```bash
# 1. Build and test - NEVER CANCEL, total ~8-12 seconds
npm run build && npm test

# 2. Validate CLI functionality
npm run cli status

# 3. Test MCP server startup (should start instantly)
timeout 5 npm run mcp
```

### Manual Validation Scenarios

**CRITICAL**: After making changes, ALWAYS test these end-to-end scenarios:

1. **CLI Capture Flow**:
   - Set up test config (use commands above)
   - Run: `npm run cli status` (should show 1 tracker loaded)
   - Test: `npm run cli capture "Test task"` (will use fallback without real API key)

2. **MCP Server Flow**:
   - Run: `npm run mcp` (should start instantly with success message)
   - Server should output: "ChurnFlow MCP Server v0.3.0 started successfully"

3. **Build System Flow**:
   - Clean: `rm -rf dist/`
   - Build: `npm run build` (should complete in ~10 seconds)
   - Verify: `ls dist/` (should contain compiled JS files)

## Codebase Navigation

### Key Directories

```
src/
├── cli.ts              # Command-line interface entry point
├── index.ts            # MCP server entry point  
├── core/               # Core business logic
│   ├── CaptureEngine.ts    # Main capture orchestration
│   ├── InferenceEngine.ts  # OpenAI GPT-4 integration
│   ├── TrackerManager.ts   # Markdown file management
│   ├── DashboardManager.ts # Task prioritization
│   └── ReviewManager.ts    # Review workflow system
├── types/              # TypeScript type definitions
└── utils/              # Utility functions

tests/                  # 161 tests across 7 suites
├── core/              # Core logic tests
├── types/             # Type definition tests
├── utils/             # Utility tests
└── fixtures/          # Mock data for testing
```

### Important Files

- `package.json`: Dependencies and npm scripts
- `tsconfig.json`: TypeScript configuration (ESM modules)
- `jest.config.js`: Test configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jgsteeler/churnflow-mcp](https://github.com/jgsteeler/churnflow-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
