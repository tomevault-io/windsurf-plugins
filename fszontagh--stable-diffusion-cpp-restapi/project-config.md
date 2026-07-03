---
trigger: always_on
description: Enables dynamic tensor offloading during generation to reduce VRAM usage. Uses the forked sd.cpp from `feature/vram-offloading` branch.
---

# CLAUDE.md - Project Context for Claude Code

## Project Overview

**sdcpp-restapi** is a REST API server wrapping stable-diffusion.cpp for image/video generation. It provides:
- HTTP REST API for txt2img, img2img, txt2vid, upscaling
- WebSocket for real-time progress updates
- Vue.js WebUI for browser-based interaction
- Queue-based job processing with soft-delete recycle bin

## Repository Structure

```
/data/sdcpp-restapi/
├── src/                    # C++ backend source
│   ├── main.cpp           # Entry point, server setup
│   ├── request_handlers.cpp # HTTP route handlers + API registry setup
│   ├── api_registry.cpp   # OpenAPI 3.1 schema generator
│   ├── queue_manager.cpp  # Job queue with recycle bin
│   ├── model_manager.cpp  # Model loading/unloading
│   ├── mcp_server.cpp     # MCP (Model Context Protocol) server
│   └── sd_wrapper.cpp     # sd.cpp integration
├── include/               # C++ headers
│   ├── mcp_server.hpp     # MCP server class declaration
│   ├── api_schema.hpp     # Schema field types, SchemaBuilder fluent API
│   ├── api_registry.hpp   # ApiRegistry class (route + schema registration)
│   ├── api_schemas.hpp    # Master include for all schema structs
│   └── api_schemas/       # Schema struct definitions by domain
│       ├── common.hpp     # Shared enums, SuccessResponse, JobCreatedResponse
│       ├── generation.hpp # GenerationRequestBase, Txt2Img/Img2Img/Txt2Vid/Upscale
│       ├── models.hpp     # LoadModelRequest, LoadOptions, download schemas
│       ├── queue.hpp      # Queue listing, job status, recycle bin schemas
│       ├── health.hpp     # Health, memory, options response schemas
│       ├── settings.hpp   # Preview, architecture, settings schemas
│       └── assistant.hpp  # Assistant chat/status schemas
├── webui/                 # Vue.js frontend
│   ├── src/views/         # Page components
│   ├── src/components/    # Reusable components
│   ├── src/stores/app.ts  # Pinia state store
│   └── src/api/client.ts  # API client with types
├── docs/                  # Documentation (served at /docs endpoint)
│   ├── API.md             # Complete API reference
│   ├── LIBRARY_REFERENCE.md # Developer library reference
│   ├── DEPENDENCIES-*.md  # Platform-specific build dependencies
│   └── plans/             # Design documents
├── CMakeLists.txt         # Build configuration
└── config.example.json    # Server configuration template
```

## Build Commands

```bash
# Standard build
mkdir build && cd build
cmake ..
make -j$(nproc)

# With experimental VRAM offloading (uses forked sd.cpp)
cmake -DSD_EXPERIMENTAL_OFFLOAD=ON ..
make -j$(nproc)

# WebUI build
cd webui
npm install
npm run build
```

## Experimental Features

### VRAM Offloading (SD_EXPERIMENTAL_OFFLOAD)

Enables dynamic tensor offloading during generation to reduce VRAM usage. Uses the forked sd.cpp from `feature/vram-offloading` branch.

**To enable:**
```bash
cmake -DSD_EXPERIMENTAL_OFFLOAD=ON ..
```

**Offload Modes:**
- `none`: Keep all components on GPU (default, fastest)
- `cond_only`: Offload only conditioning (LLM/CLIP) after use
- `cond_diffusion`: Offload conditioning + diffusion, keep VAE
- `aggressive`: Offload each component after use (saves most VRAM)
- `layer_streaming`: Stream layers one-by-one (enables models larger than VRAM)

**General Offload Options:**
- `vram_estimation`: dryrun (accurate), formula (fast)
- `offload_cond_stage`: Offload LLM/CLIP after conditioning
- `offload_diffusion`: Offload diffusion model after sampling
- `reload_cond_stage/reload_diffusion`: Reload components after generation
- `target_free_vram_mb`: Target free VRAM before VAE decode (0 = always offload)
- `min_offload_size_mb`: Minimum component size to offload

**Layer Streaming Options (for `layer_streaming` mode):**
- `layer_streaming_enabled`: Enable layer-by-layer streaming execution
- `streaming_prefetch_layers`: Number of layers to prefetch ahead (default: 1)
- `streaming_keep_layers_behind`: Layers to keep after execution (for skip connections)
- `streaming_min_free_vram_mb`: Minimum VRAM to keep free during streaming

**Backend implementation:**
- `include/model_manager.hpp`: `ModelLoadParams` struct with all offload options
- `src/model_manager.cpp`: Parses options, converts to `sd_offload_config_t`
- `src/request_handlers.cpp`: Exposes `features.experimental_offload` in health endpoint

**WebUI implementation:**
- `webui/src/api/client.ts`: TypeScript types for offload options
- `webui/src/stores/app.ts`: `experimentalOffloadEnabled` computed property
- `webui/src/views/ModelLoad.vue`: VRAM Offloading accordion with layer streaming UI
- `webui/src/views/Dashboard.vue`: Displays active offload settings with labels

**Conditional compilation pattern:**
```cpp
#ifdef SDCPP_EXPERIMENTAL_OFFLOAD
    // Experimental code here
#endif
```

## Key Patterns

### OpenAPI Schema System (Auto-Generated)

The server generates an OpenAPI 3.1 specification at `GET /openapi.json` that is always in sync with the code. The schema is built from:

1. **Schema structs** in `include/api_schemas/*.hpp` using `SchemaBuilder` fluent API
2. **ApiRegistry** in `include/api_registry.hpp` that registers routes + schemas together

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fszontagh/stable-diffusion.cpp-restapi](https://github.com/fszontagh/stable-diffusion.cpp-restapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
