---
trigger: always_on
description: Coding conventions and architectural patterns for the GenAI IDP Accelerator. All constructs are **experimental** — breaking changes are acceptable without deprecation.
---

# Agent Development Conventions

Coding conventions and architectural patterns for the GenAI IDP Accelerator. All constructs are **experimental** — breaking changes are acceptable without deprecation.

## Code Organization

### Module Structure

Every feature is a self-contained module:

```
src/{feature-name}/
├── index.ts                         # Exports all public components
├── {feature-name}.ts                # Main construct
├── {feature-name}-table.ts          # Feature-specific table (if needed)
└── functions/
    ├── index.ts
    └── {function-name}-function.ts
```

- ✅ Each feature gets its own folder with its own `index.ts`
- ✅ Lambda functions go in `functions/` subfolder
- ❌ No feature files in root `src/`; no new Lambdas in `src/internal/functions/`

### Package Layout

```
src/
├── document-discovery/             # Core: config generation from samples
├── reporting/                      # Core: evaluation metrics
├── processing-environment-api/     # Auxiliary features + GraphQL API
│   ├── agent-analytics/
│   ├── agent-companion-chat/
│   ├── error-analyzer/
│   ├── mcp-integration/
│   ├── test-studio/
│   └── functions/                  # API resolver functions
├── functions/                      # Shared/utility functions
├── internal/                       # Internal utilities
├── hitl/                           # Human-in-the-loop
├── custom-prompt-generator/
└── index.ts                        # Main exports
```

## Architectural Separation

Classify every new feature as **core processing** or **auxiliary**.

**Core processing** — directly impacts document processing workflows. Integrated via `ProcessingEnvironment` constructor props:
- `DocumentDiscovery`, `ReportingEnvironment`

**Auxiliary** — provides UI, testing, diagnostics, or querying. Integrated via `enable()`:
- `KnowledgeBaseQuery`, `Evaluation`, `ChatWithDocument`, `AgentAnalytics`, `AgentCompanionChat`, `ProgressMonitor`, `UserManagement`, `DocumentDiscovery` (API+UI side)

```typescript
// Core — constructor props
const environment = new ProcessingEnvironment(this, 'Env', {
  inputBucket, outputBucket, workingBucket,
  configurationTable, trackingTable, api, reportingEnvironment,
});

// Auxiliary — construct independently, then enable
const evaluation = new Evaluation(this, 'Eval', { ... });
api.enable(evaluation);
webApp.enable(evaluation);

// ❌ Never pass features as constructor props on API or WebApp
```

### Adding New Features

1. Classify as core or auxiliary
2. Implement as standalone construct (no API/UI dependency at construction)
3. Implement `IApiFeature` if it needs GraphQL resolvers
4. Implement `IWebAppFeature` if it needs UI settings or CORS
5. Feature-specific resources (buckets, tables) are constructor props on the feature
6. Update samples to use `api.enable()` / `webApp.enable()`

## Feature Integration Pattern

Plugin architecture: features are constructed independently, then enabled in API and/or WebApp.

### Interfaces

```typescript
export interface IApiFeature {
  enableInApi(api: IProcessingEnvironmentApi): void;
}
export interface IWebAppFeature {
  enableInWebApp(webApp: IWebApplication): void;
}
```

Two interfaces (not one combined) gives compile-time type safety — `webApp.enable(apiOnlyFeature)` is a type error.

### Usage

```typescript
// Target-driven (preferred for multiple features)
api.enable(knowledgeBaseQuery);
api.enable(evaluation);
webApp.enable(knowledgeBaseQuery);

// Feature-driven (preferred for one feature across targets)
kbQuery.enableInApi(api);
kbQuery.enableInWebApp(webApp);
```

### Implementation Template

```typescript
export class MyFeature extends Construct implements IApiFeature, IWebAppFeature {
  private readonly myResource: IMyResource;

  constructor(scope: Construct, id: string, props: MyFeatureProps) {
    super(scope, id);
    this.myResource = props.myResource; // Feature owns its resources
  }

  public enableInApi(api: IProcessingEnvironmentApi): void {
    const fn = new MyFunction(api as Construct, 'Fn', { ... });
    const ds = api.addLambdaDataSource('DS', fn);
    ds.createResolver('Resolver', { typeName: 'Query', fieldName: 'myQuery' });
  }

  public enableInWebApp(webApp: IWebApplication): void {
    webApp.addSetting('MyFeatureEnabled', 'true');
    webApp.addCorsBucket(this.myResource);
  }
}
```

### Resource Ownership

- Feature-specific resources → constructor props on the feature
- Shared resources (e.g. `uploadResolverFunction`) → exposed on `IProcessingEnvironmentApi`
- Features must NOT depend on API/WebApp at construction time — only at `enable()` time

### Feature Matrix

| Feature            | `IApiFeature` | `IWebAppFeature` |
|--------------------|:---:|:---:|
| DocumentDiscovery  | ✅  | ✅  |
| KnowledgeBaseQuery | ✅  | ✅  |
| Evaluation         | ✅  | ✅  |
| ChatWithDocument   | ✅  | ❌  |
| AgentAnalytics     | ✅  | ❌  |
| ProgressMonitor    | ✅  | ❌  |
| UserManagement     | ✅  | ❌  |
| AgentCompanionChat | ✅  | ❌  |

### WebApplication SSM Integration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cdklabs/genai-idp](https://github.com/cdklabs/genai-idp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
