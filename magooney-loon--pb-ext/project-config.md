---
trigger: always_on
description: Enables `inferTypeFromExpression` and `analyzeValueExpression` to resolve variables assigned from helper function calls (e.g., `candles := formatCandlesFull(records)` → type `[]map[string]any`).
---

# API Documentation System — Agent Guide

This package (`core/server/api/`) is the OpenAPI documentation engine for pb-ext. It parses Go source at startup via AST analysis, extracts handler metadata and type schemas, and serves versioned OpenAPI 3.0.3 specs.

## File Map

### Types
| File | What it owns |
|---|---|
| `types_ast.go` | All AST data structures: `ASTParser`, `StructInfo`, `FieldInfo`, `ASTHandlerInfo`, `MapKeyAdd`, `ParamInfo`, `ParseError`, `PocketBasePatterns`, logger interface, `ASTParserInterface` |
| `types_api.go` | `APIEndpoint`, `APIDocs`, `APIDocsConfig`, `AuthInfo`, `HandlerInfo` — the route/endpoint model |
| `types_openapi.go` | Full OpenAPI 3.0 type hierarchy: `OpenAPISchema`, `OpenAPIPathItem`, `OpenAPIOperation`, `OpenAPIComponents`, `OpenAPIParameter`, etc. |

### AST Parser (split by responsibility)
| File | What it owns |
|---|---|
| `ast.go` | Entry points: `NewASTParser`, `DiscoverSourceFiles`, `ParseFile`, `EnhanceEndpoint`, all public interface methods (`GetAllStructs`, `GetAllHandlers`, `ClearCache`, etc.) |
| `ast_func.go` | Handler/function analysis: `extractHandlers`, `extractFuncReturnTypes`, `extractHelperFuncParams`, `extractParamsFromBody`, `extractQueryParameters`, `analyzeHelperFuncBody`, `isPocketBaseHandler`, `analyzePocketBaseHandler`, `analyzePocketBasePatterns`, `analyzePocketBaseCall`, `trackVariableAssignment`, `handleBindBody`, `handleJSONResponse`, all query/header/path detection helpers |
| `ast_struct.go` | Struct analysis and schema generation: `extractStructs`, `parseStruct`, `generateStructSchema`, `flattenEmbeddedFields`, `generateFieldSchema`, `generateSchemaForEndpoint`, `generateSchemaFromType`, `deepCopySchema` |
| `ast_metadata.go` | Value/type resolution: `analyzeMapLiteralSchema`, `parseMapLiteral`, `analyzeValueExpression`, `resolveTypeToSchema`, `schemaFromMakeArg`, `analyzeCompositeLitSchema`, `parseArrayLiteral`, `extractVariableDeclarations`, `extractLocalVariables`, `extractVarDecl`, `resolveTypeAlias`, `NewPocketBasePatterns` |
| `ast_file.go` | File-level utilities: `newFileSet`, `getModulePath`, `parseImportedPackages`, `parseDirectoryStructs` |

### Registry
| File | What it owns |
|---|---|
| `registry.go` | `APIRegistry` struct, constructor, `RegisterEndpoint`, helpers |
| `registry_routes.go` | `RegisterRoute`, `RegisterExplicitRoute`, `BatchRegisterRoutes`, `enhanceEndpointWithAST`, `createEndpointFromAnalysis` |
| `registry_spec.go` | `GetDocsWithComponents`, `buildPaths`, `endpointToOperation`, `extractPathParameters`, `collectRefsFromSchema` (pruning), `generateOperationId`, `buildTags` |

### Other
| File | What it owns |
|---|---|
| `schema.go` | `SchemaGenerator` — request/response schema inference, `GenerateComponentSchemas`, `promoteHandlerResponseSchemas` |
| `schema_conversion.go` | Go type → OpenAPI conversion: `ConvertGoTypeToOpenAPISchema`, `ConvertParamInfoToOpenAPIParameter`, validation tag parsing |
| `version_manager.go` | `APIVersionManager`, `VersionedAPIRouter`, `VersionedRouteChain`, `PrefixedRouter`, multi-version routing, per-version registries, `ServeSwaggerUI` (SwaggerDark CSS via `strings.NewReplacer`) |
| `discovery.go` | `RouteAnalyzer`, `MiddlewareAnalyzer`, `PathAnalyzer` — runtime route analysis utilities |
| `debug_dump.go` | `BuildDebugData()` — serves the `/api/docs/debug/ast` endpoint with full pipeline introspection |
| `utils.go` | String helpers: handler name extraction, camelCase/snake_case, description/tag generation, path conversion |
| `openapi_embedded_loader.go` | Spec loading from disk, caching, env var override (`PB_EXT_OPENAPI_SPECS_DIR`, `PB_EXT_DISABLE_OPENAPI_SPECS`) |
| `spec_generator.go` | Spec generation and validation for build-time generation |

### Tests
| File | What it covers |
|---|---|
| `ast_test.go` | Core parser lifecycle, `ParseFile`, `EnhanceEndpoint`, `DiscoverSourceFiles`, benchmarks |
| `ast_struct_test.go` | Struct/schema/type extraction and JSON schema generation |
| `ast_func_test.go` | Handler scenarios (46+ handlers), func return type resolution, `funcBodySchemas`, append-based resolution, direct and indirect parameter detection |
| `ast_file_test.go` | Cross-package struct resolution via import following |
| `registry_test.go` | `APIRegistry` route registration and OpenAPI output |
| `schema_test.go` | `SchemaGenerator` and component schema generation |
| `version_manager_test.go` | `APIVersionManager` and versioned routing |
| `discovery_test.go` | Route/middleware/path analysis |
| `utils_test.go` | String helper utilities |
| `openapi_embedded_loader_test.go` | Spec loading from disk, caching, deep copy, env var handling |
| `spec_generator_test.go` | Spec generation and validation |

## Pipeline Overview

```
Source files (// API_SOURCE)
  |
  v
ASTParser.DiscoverSourceFiles()
  |
  |-- Phase 1: Parse API_SOURCE files
  |     |
  |     v
  |   ASTParser.ParseFile()  (for each API_SOURCE file)
  |     |-- extractStructs()            two-pass: register all structs, then generate JSONSchemas
  |     |-- extractVariableDeclarations()  global var tracking

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [magooney-loon/pb-ext](https://github.com/magooney-loon/pb-ext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
