---
trigger: always_on
description: VSCode extension that scans your workspace for API call patterns, estimates costs, shows diagnostics, and opens a full dashboard locally. Multi-provider AI chat and optional AI code review.
---

# ReCost - API Usage Analyzer (VSCode Extension)

VSCode extension that scans your workspace for API call patterns, estimates costs, shows diagnostics, and opens a full dashboard locally. Multi-provider AI chat and optional AI code review.

## Tech Stack

- **TypeScript** — extension backend (strict mode)
- **esbuild** — bundler for extension and webview
- **React 18** — sidebar webview UI
- **Vite** — dashboard bundler
- **Multi-provider AI chat** — ReCost AI (free, default), OpenAI, Anthropic, Gemini, xAI, Cohere, Mistral, Perplexity
- **OpenAI SDK** — AI review (optional, `gpt-4.1-mini` default)
- **web-tree-sitter** — AST parsing for JS/TS/Python (WASM, loaded at runtime)

## Project Structure

```
src/
  extension.ts            # Extension entry point
  api-client.ts           # HTTP client for remote ReCost API (rc- prefix key validation)
  local-server.ts         # Embedded HTTP server (serves dashboard + proxies local analysis)
  webview-provider.ts     # Sidebar webview provider (IPC handling, local pricing table for 40+ providers, cost estimation algorithm)
  messages.ts             # IPC message types (extension ↔ webview)
  key-management.ts       # Key service registry, validation snapshots, KeyServiceDescriptor; manages recost + all chat provider keys
  output.ts               # Lazy OutputChannel singleton ("ReCost Status")
  scan-results.ts         # Post-scan result builder: scope/provider enrichment, local pricing, filters unknown providers before remote submit
  workspace-file-access.ts # Safe workspace path resolver — guards against path traversal before file reads
  analysis/
    types.ts              # Analysis type definitions
  cli/
    scan.ts               # CLI scan runner — orchestrates workspace scan, remote submit, intelligence pipeline, and formatted output
    filesystem-adapter.ts # ScanFileAccess adapter backed by the real filesystem (vs the VSCode workspace API)
  ast/
    parser-loader.ts      # web-tree-sitter WASM loader (resolves from dist/../assets/parsers/)
    ast-scanner.ts        # AST-based API call scanner (JS/TS/Python)
    call-visitor.ts       # AST call expression visitor — walks tree, emits raw call sites
    frequency-analyzer.ts # Classifies call frequency: single, bounded-loop, unbounded-loop, polling, parallel, conditional, cache-guarded
    cross-file-resolver.ts # Resolves API calls through helper functions back to their origin file
    import-resolver.ts    # Resolves import paths for cross-file tracing
    waste/
      batch-detector.ts      # AST-based batch and N+1 waste detector
      cache-detector.ts      # AST-based cache guard waste detector
      concurrency-detector.ts # AST-based concurrency/parallel waste detector
  chat/
    prompts.ts            # AI prompt templates
    types.ts              # Shared chat types
    provider-registry.ts  # Provider registry & auth resolution (env var → SecretStorage fallback)
    index.ts              # executeChat() dispatcher
    errors.ts             # ChatAdapterError
    providers/            # Per-provider adapters (recost, openai, anthropic, gemini, xai, cohere, mistral, perplexity)
  scanner/
    patterns.ts           # API call detection regex patterns (fallback/augment to AST)
    patterns/             # 16 provider-specific pattern scanners (Firebase, GraphQL, OpenAI, Stripe, Anthropic, Bedrock, etc.) + registry.ts, types.ts, utils.ts
    core-scanner.ts       # Core single-file scanner (coordinates AST + regex passes)
    workspace-scanner.ts  # Workspace file scanner (orchestrates core-scanner across files)
    file-discovery.ts     # Workspace file discovery and glob filtering
    path-excludes.ts      # Default exclude path patterns
    endpoint-classification.ts  # Classifies endpoints as internal/external, detects 50+ provider hosts
    local-waste-detector.ts     # Detects waste patterns using AST signals (N+1, unbounded loops, polling without backoff, missing cache guards, unbatched parallel)
    python-waste-detector.ts    # Python-specific waste detection
    fingerprints/               # Per-method pricing fingerprints — JSON data files per provider + index.ts, registry.ts, types.ts
  simulator/              # Cost Simulator computation layer
    types.ts              # SimulatorInput, SimulatorResult, SavedScenario, scale presets (1K–100K)
    engine.ts             # Pure runSimulation() — frequency-class multipliers, free endpoint zeroing, dynamic confidence
    static-source.ts      # StaticDataSource adapter (EndpointRecord[] → SimulatorDataSource, passes frequencyClass + costModel)
    index.ts              # Barrel re-export
  intelligence/           # Intelligence layer — graph model of a scanned repo
    types.ts              # Shared interface contracts (FileNode, ApiCallNode, FindingNode, ProviderNode, RepoIntelligenceSnapshot, ScoredSnapshot, ReviewCluster, CompressedCluster, ExportedContext, etc.). Do NOT change without team sync.
    builder.ts            # buildSnapshot() — constructs RepoIntelligenceSnapshot from apiCalls + findings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [recost-dev/extension](https://github.com/recost-dev/extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
