---
trigger: always_on
description: >-
---


# Paladin Plugin Development Rules

This file contains architectural patterns, conventions, and critical rules for developing plugins and plugin managers in the Paladin system. Always follow these patterns when implementing or modifying plugin code.

## Core Architecture Principles

### Plugin System Overview
- **Plugins are separate processes** communicating via bidirectional gRPC streams
- **Isolation**: Plugin crashes don't affect core system
- **Single long-lived connection** per plugin instance
- **Correlation IDs** link requests to responses in both directions
- All plugins must follow the **Bridge Pattern** structure

### Communication Pattern
```
Plugin Process ◄─── gRPC Stream (bidirectional) ───► Plugin Manager (Core)
```
- Messages use `Header` with `plugin_id`, `message_id`, `correlation_id`, `message_type`
- Message types: `REGISTER`, `REQUEST_TO_PLUGIN`, `RESPONSE_FROM_PLUGIN`, `REQUEST_FROM_PLUGIN`, `RESPONSE_TO_PLUGIN`, `ERROR_RESPONSE`

## Bridge Pattern (CRITICAL)

### Standard Bridge Structure (ALL plugins must follow)
```go
type YourPluginBridge struct {
    plugin     *plugin[prototk.YourMessage]      // Plugin instance reference
    pluginType string                             // Plugin type identifier
    pluginName string                             // Plugin name from config
    pluginId   string                             // Plugin UUID as string
    toPlugin   managerToPlugin[prototk.YourMessage]  // Manager → Plugin channel
    manager    YourPluginCallbacks                // Plugin → Manager callbacks
}
```

**Key Rules:**
- ✅ **ALL plugin bridges MUST have these exact fields** (even unidirectional plugins)
- ✅ **ALL plugins MUST implement `Initialized()` method** (calls `plugin.notifyInitialized()`)
- ✅ **ALL plugins MUST implement `RequestReply()` method**
  - Bidirectional plugins: Route callbacks to manager
  - Unidirectional plugins: Return no-op `func(...) {}, nil` to conform

### Required Methods

1. **`Initialized()`** - REQUIRED for all plugins
   - Called by manager after plugin completes initialization
   - **CRITICAL**: Must NEVER call plugin API functions before `Initialized()` is called
   - Pattern: `func (br *Bridge) Initialized() { br.plugin.notifyInitialized() }`

2. **`RequestReply()`** - REQUIRED for all plugins
   - Bidirectional: Routes plugin callbacks to manager methods
   - Unidirectional: Returns no-op but maintains structure

### Initialization Sequence (MUST follow strictly)
1. Plugin connects to manager (gRPC stream established)
2. Plugin sends REGISTER message
3. Manager creates bridge with standard fields
4. Manager registers bridge with specific manager (DomainManager, RPCAuthManager, etc.)
5. Manager sends CONFIGURE request to plugin
6. Plugin responds with configuration success
7. Manager sends INIT request (for domain/transport plugins)
8. Plugin responds with init success
9. Manager calls `bridge.Initialized()`
10. `WaitForInit()` unblocks

**CRITICAL ANTI-PATTERN TO AVOID:**
```go
// ❌ WRONG - Will cause hangs!
br := &domainBridge{...}
pm.domainManager.DomainRegistered(pluginName, pluginID, bridge)
br.ConfigureDomain(ctx, config)  // NEVER call before Initialized()!

// ✅ CORRECT
br := &domainBridge{...}
pm.domainManager.DomainRegistered(pluginName, pluginID, bridge)
// Let manager configure via CONFIGURE request, then call Initialized()
```

## Plugin Types

### 1. Domain Plugins
- **Purpose**: Blockchain domain logic (EVM, Hyperledger Fabric, etc.)
- **Files**: `core/go/internal/plugins/domains.go`
- **Manager Method**: `ConnectDomain()`, `DomainRegistered()`
- **Type**: Bidirectional

### 2. Transport Plugins
- **Purpose**: Inter-node communication
- **Files**: `core/go/internal/plugins/transports.go`
- **Manager Method**: `ConnectTransport()`, `TransportRegistered()`
- **Type**: Bidirectional

### 3. Registry Plugins
- **Purpose**: On-chain registries for contracts and nodes
- **Files**: `core/go/internal/plugins/registries.go`
- **Manager Method**: `ConnectRegistry()`, `RegistryRegistered()`
- **Type**: Bidirectional

### 4. Signing Module Plugins
- **Purpose**: Cryptographic keys and signing operations
- **Files**: `core/go/internal/plugins/signing_modules.go`
- **Manager Method**: `ConnectSigningModule()`, `SigningModuleRegistered()`
- **Type**: Bidirectional

### 5. RPC Auth Plugins
- **Purpose**: RPC request authentication/authorization
- **Files**: `core/go/internal/plugins/rpcauth.go`
- **Manager Method**: `ConnectRPCAuthPlugin()`, `RPCAuthorizerRegistered()`
- **Type**: Unidirectional (but still uses standard bridge structure)

## Implementation Patterns

### Protobuf Message Structure
All plugin messages in `toolkit/proto/protos/service.proto` follow this pattern:
```protobuf
message YourMessage {
  Header header = 1;
  
  // Manager → Plugin
  oneof request_to_yourplugin {
    YourRequest1 request1 = 1010;
  }
  
  // Plugin → Manager (responses)
  oneof response_from_yourplugin {
    YourResponse1 response1 = 1011;
  }
  
  // Plugin → Manager (requests/callbacks)
  oneof request_from_yourplugin {
    YourManagerRequest1 mgr_request1 = 2010;
  }
  
  // Manager → Plugin (responses to callbacks)
  oneof response_to_yourplugin {
    YourManagerResponse1 mgr_response1 = 2011;
  }
}
```

**Always regenerate protobuf code after modifying `.proto` files:**
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LFDT-Paladin/paladin](https://github.com/LFDT-Paladin/paladin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
