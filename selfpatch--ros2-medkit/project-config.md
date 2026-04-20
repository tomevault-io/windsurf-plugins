---
trigger: always_on
description: **ros2_medkit** is a ROS 2 diagnostics gateway that exposes ROS 2 system information via a RESTful HTTP API aligned with the **SOVD (Service-Oriented Vehicle Diagnostics)** specification. It models robots as a diagnostic entity tree: **Area -> Component -> App**, with optional **Function** groupings.
---

# Copilot Instructions for ros2_medkit

## Project Overview

**ros2_medkit** is a ROS 2 diagnostics gateway that exposes ROS 2 system information via a RESTful HTTP API aligned with the **SOVD (Service-Oriented Vehicle Diagnostics)** specification. It models robots as a diagnostic entity tree: **Area -> Component -> App**, with optional **Function** groupings.

**Tech Stack**: C++17, ROS 2 Jazzy/Humble/Rolling, Ubuntu 24.04/22.04

## Package Structure

Multi-package colcon workspace under `src/`:

| Package | Purpose |
|---------|---------|
| `ros2_medkit_gateway` | HTTP gateway - REST server, discovery, entity management, handlers, plugin framework |
| `ros2_medkit_fault_manager` | Fault aggregation with SQLite, AUTOSAR DEM-style debounce, rosbag/snapshot capture |
| `ros2_medkit_fault_reporter` | Client library for nodes to report faults |
| `ros2_medkit_diagnostic_bridge` | Bridges `/diagnostics` topic to fault manager |
| `ros2_medkit_serialization` | Runtime JSON <-> ROS 2 message serialization via dynmsg |
| `ros2_medkit_msgs` | Custom service/message definitions |
| `ros2_medkit_integration_tests` | Python integration test framework + launch tests |
| `ros2_medkit_beacon_common` | Shared beacon validation, hint store, and entity mapper |
| `ros2_medkit_topic_beacon` | Push-based topic beacon discovery plugin |
| `ros2_medkit_param_beacon` | Pull-based parameter beacon discovery plugin |

## Architecture

```
GatewayNode (src/ros2_medkit_gateway/src/gateway_node.cpp)
├── DiscoveryManager      - Entity discovery (runtime / manifest / hybrid)
│   └── EntityCache       - Thread-safe in-memory entity store
├── DataAccessManager     - Topic sampling and publishing
├── OperationManager      - Service calls and action goals
├── ConfigurationManager  - ROS 2 parameter CRUD
├── FaultManager          - Fault query/clear, SSE streams (delegates to fault_manager package)
├── LogManager            - /rosout ring buffer + plugin delegation
├── PluginManager         - Dynamic plugin loading (.so), provider interfaces
└── RESTServer            - cpp-httplib HTTP server (separate thread)
    └── HandlerContext    - Shared validation, entity lookup, error/JSON helpers
```

### Entity Model (SOVD-aligned)

Defined in `include/ros2_medkit_gateway/models/`:

- **Area** - namespace grouping (`/powertrain`, `/chassis`, `root`)
- **Component** - groups Apps by namespace (synthetic in runtime mode, explicit in manifest)
- **App** - individual ROS 2 node
- **Function** - capability grouping (functional view, aggregates Apps)

Entity types: `SovdEntityType` enum in `entity_types.hpp`. Capability matrix in `entity_capabilities.hpp` (SOVD Table 8).

### SOVD Resource Collections

Each entity type supports specific collections:

| Collection | Server | Area | Component | App | Function |
|-----------|--------|------|-----------|-----|----------|
| configurations | x | | x | x | |
| data | x | | x | x | x (aggregated) |
| faults | x | | x | x | |
| operations | x | | x | x | x (aggregated) |
| logs | x | | x | x | |

## Code Style & Conventions

- **C++17** with `-Wall -Wextra -Wpedantic -Wshadow -Wconversion`
- **Formatting**: Google-based clang-format, 120 cols, 2-space indent, middle pointer alignment (`Type * ptr`)
- **Headers**: `#pragma once`, Apache 2.0 copyright (year 2026 for new files)
- **Namespace**: `ros2_medkit_gateway`
- **Error handling**: `tl::expected<T, std::string>` for fallible operations - NOT exceptions in handlers
- **Thread safety**: Always get fresh `EntityCache` via `ctx_.node()->get_thread_safe_cache()` (in handlers)
- **Header-only libs**: nlohmann::json, tl::expected, jwt-cpp, cpp-httplib

## Handler Pattern (follow for ALL new handlers)

Handlers live in `src/http/handlers/`, one file per group. All handlers take `HandlerContext&` in constructor.

```cpp
void ExampleHandlers::handle_request(const httplib::Request & req, httplib::Response & res) {
  // 1. Extract entity ID from regex match
  auto entity_id = req.matches[1].str();

  // 2. Validate entity (sends error response automatically if invalid)
  auto entity = ctx_.validate_entity_for_route(req, res, entity_id);
  if (!entity) return;  // Response already sent (error or forwarded to peer)

  // 3. Check collection support (static method)
  if (auto err = HandlerContext::validate_collection_access(*entity, ResourceCollection::DATA)) {
    HandlerContext::send_error(res, 400, ERR_COLLECTION_NOT_SUPPORTED, *err);
    return;
  }

  // 4. Get fresh thread-safe cache
  const auto & cache = ctx_.node()->get_thread_safe_cache();

  // 5. Business logic (use tl::expected for errors)
  auto result = some_operation();
  if (!result) {
    HandlerContext::send_error(res, 503, ERR_SERVICE_UNAVAILABLE, result.error());
    return;
  }

  // 6. Return JSON response
  HandlerContext::send_json(res, json{{"items", *result}});
}
```

### HandlerContext Key Methods

| Method | Returns | Purpose |
|--------|---------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [selfpatch/ros2_medkit](https://github.com/selfpatch/ros2_medkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
