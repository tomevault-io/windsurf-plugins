---
trigger: always_on
description: validateStatus: (status: number) => status < 400,
---

# OpenAPI TypeScript Request - AI Agent Guide

## 🤖 Tool Overview

OpenAPI TypeScript Request is a powerful code generator that converts Swagger2/OpenAPI3/Apifox specifications into TypeScript/JavaScript client code with advanced features like React Query integration, mock services, type labels, and multi-client support.

**Core Capabilities:**

- Generate TypeScript/JavaScript request functions from OpenAPI specs
- Support multiple HTTP clients (axios, fetch, uniapp, taro, node-fetch)
- Generate React Query / Vue Query hooks
- Create mock services with realistic data
- Generate Chinese type labels for forms
- Support enum descriptions and translations
- Generate JSON Schemas for validation

## 🎯 Quick AI Decision Tree

### 1. Project Type Detection

```typescript
// Detect from package.json dependencies
if (has("react") && has("@tanstack/react-query")) {
  recommend: { isGenReactQuery: true, reactQueryMode: "react" }
}
else if (has("vue") && has("@tanstack/vue-query")) {
  recommend: { isGenReactQuery: true, reactQueryMode: "vue" }
}
else if (has("@tarojs/taro") || has("@dcloudio/uni-app")) {
  recommend: { requestLibPath: "@/request" }
}
else if (isNodeProject) {
  recommend: { requestLibPath: "node-fetch" }
}
```

### 2. Complete Configuration Reference

#### Core Settings

| Configuration | Type | Default | Description | When to Use |
| --- | --- | --- | --- | --- |
| `schemaPath` | string | - | OpenAPI/Swagger spec URL or file path | Use when providing OpenAPI spec URL/file path (mutually exclusive with apifoxConfig) |
| `apifoxConfig` | object | - | Apifox configuration | Use when fetching spec from Apifox (mutually exclusive with schemaPath) |
| `serversPath` | string | `"./src/apis"` | Output directory for generated files | Custom output location |
| `requestLibPath` | string | `"axios"` | HTTP client library path | Custom request client |
| `full` | boolean | `true` | Full replacement vs incremental | Incremental updates |
| `enableLogging` | boolean | `false` | Enable debug logging | Troubleshooting |

#### Filtering & Tags

| Configuration | Type | Default | Description | When to Use |
| --- | --- | --- | --- | --- |
| `priorityRule` | `"include"\|"exclude"\|"both"` | `"include"` | Filter strategy | Large APIs |
| `filterCaseInsensitive` | boolean | `false` | Case-insensitive tag/path matching | Mixed case APIs |
| `includeTags` | `(string\|RegExp)[]` | - | Only generate these tags | Selective generation |
| `includePaths` | `(string\|RegExp)[]` | - | Only generate these paths | Path-based filtering |
| `excludeTags` | `(string\|RegExp)[]` | - | Skip these tags | Exclude unwanted APIs |
| `excludePaths` | `(string\|RegExp)[]` | - | Skip these paths | Exclude specific endpoints |

#### Code Generation Options

| Configuration | Type | Default | Description | When to Use |
| --- | --- | --- | --- | --- |
| `isGenJavaScript` | boolean | `false` | Generate JS instead of TS | JS-only projects |
| `isGenReactQuery` | boolean | `false` | Generate React/Vue Query hooks | Query-based apps |
| `reactQueryMode` | `"react"\|"vue"` | `"react"` | Query hook framework | Vue projects |
| `isDisplayTypeLabel` | boolean | `false` | Generate Chinese field labels | Form applications |
| `isGenJsonSchemas` | boolean | `false` | Generate JSON Schema validation | Data validation |
| `isOnlyGenTypeScriptType` | boolean | `false` | Only types, no request functions | Type-only needs |
| `isCamelCase` | boolean | `true` | Use camelCase naming | Consistent naming |
| `isSupportParseEnumDesc` | boolean | `false` | Parse enum descriptions | Labeled enums |
| `supportParseEnumDescByReg` | `string \| RegExp` | - | Custom regex for parsing enum descriptions. If set, replaces default parseDescriptionEnum method. Example: `/([^\s=<>/&;]+(?:\s+[^\s=<>/&;]+)*)\s*=\s*(\d+)/g` matches "普通 = 0" or "SampleMaker = 1" | Custom enum description formats |
| `isSplitTypesByModule` | boolean | `false` | Split types by module, generates {module}.type.ts, common.type.ts, enum.ts, types.ts | Large projects with many types |

#### Request Customization

| Configuration | Type | Default | Description | When to Use |
| --- | --- | --- | --- | --- |
| `requestOptionsType` | string | `"{ [key: string]: unknown }"` | Type for request options | Custom options |
| `requestImportStatement` | string | - | Custom import statement | Custom request setup |
| `apiPrefix` | string\|function | - | API path prefix | Namespaced APIs |
| `dataFields` | string[] | - | Response data field path | Nested response data |

#### Internationalization

| Configuration | Type | Default | Description | When to Use |
| --- | --- | --- | --- | --- |
| `isTranslateToEnglishTag` | boolean | `false` | Translate Chinese tags to English | Chinese API docs |
| `nullable` | boolean | `false` | Use null instead of optional | Strict null checks |

#### Development & Testing

| Configuration | Type | Default | Description | When to Use |
| --- | --- | --- | --- | --- |
| `mockFolder` | string | - | Mock data output directory | Development with mocks |
| `authorization` | string | - | API documentation auth token | Protected docs |
| `timeout` | number | `60000` | Request timeout in ms | Slow APIs |

#### Advanced Options

| Configuration | Type | Default | Description | When to Use |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openapi-ui/openapi-ts-request](https://github.com/openapi-ui/openapi-ts-request) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
