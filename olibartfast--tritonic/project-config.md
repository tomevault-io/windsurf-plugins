---
trigger: always_on
description: TritonIC is a C++20 application for running ML inference (object detection, instance segmentation, classification, video classification, pose estimation, optical flow, depth estimation) via the Nvidia Triton Inference Server **or** any OpenAI-compatible chat endpoint. It communicates with Triton over HTTP or gRPC, and with chat servers via `/v1/chat/completions`.
---

# Tritonic — Agent Guidelines

TritonIC is a C++20 application for running ML inference (object detection, instance segmentation, classification, video classification, pose estimation, optical flow, depth estimation) via the Nvidia Triton Inference Server **or** any OpenAI-compatible chat endpoint. It communicates with Triton over HTTP or gRPC, and with chat servers via `/v1/chat/completions`.

## Build and Test

**Prerequisites:**
```bash
export TritonClientBuild_DIR=$(pwd)/triton_client_libs/install
./docker/scripts/extract_triton_libs.sh   # extract from Docker if not present
```

**Standard build:**
```bash
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build . -j$(nproc)
```

**CMake options:**
- `-DWITH_SHOW_FRAME=ON` — display frames in a window during inference
- `-DWITH_WRITE_FRAME=ON` — write output frames/video to disk (default ON)
- `-DWITH_CHAT_BACKEND=ON` — OpenAI-compatible chat backend (default ON)
- `-DBUILD_TESTING=ON` — build unit and integration tests

**Build and run tests:**
```bash
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTING=ON ..
cmake --build . -j$(nproc)
ctest --output-on-failure
# or: ./build/tritonic_unit_tests
```

**Code formatting and git hooks:**
```bash
# Install git hook for automatic formatting on commit (RECOMMENDED)
./scripts/setup/install_git_hooks.sh

# Or manually format all files before committing
./scripts/format_all.sh

# Or use the legacy pre-commit setup (requires Python)
./scripts/setup/pre_commit_setup.sh
```

> **Important:** Install the git hook to avoid CI format check failures. The hook automatically formats C++ files before each commit.

## Architecture

### Dual-backend design

| Backend | CLI flag | Use case |
|---------|----------|----------|
| **Triton** (default) | `--backend=triton` | Binary tensor inference — detection, seg, classification, optical flow, pose, depth |
| **Chat** | `--backend=chat` | OpenAI `/v1/chat/completions` — VLMs, LLMs, multimodal chat |

Both backends implement `tritonic::core::IInferenceBackend` (Strategy pattern), enabling dependency injection and mockability in tests.

### Code structure

```
src/
  main/client.cpp          Entry point: parses CLI, selects backend, runs
  main/App.cpp             Inference pipeline (image/video loop, rendering)
  main/Logger.cpp          Logger implementation (namespace tritonic::infra)
  main/ConfigManager.cpp   CLI parsing via cv::CommandLineParser (namespace tritonic::infra)
  triton/Triton.hpp/.cpp   Concrete ITriton (HTTP & gRPC, shared memory)
  triton/ITriton.hpp       Forwarding header → tritonic/triton/itriton.hpp
  triton/TritonModelInfo.hpp  Forwarding header → tritonic/triton/model_info.hpp
  triton/TritonBackend.hpp Forwarding header → tritonic/triton/triton_backend.hpp
  chat/ChatBackend.hpp/.cpp  OpenAI-compatible CURL facade (dual-inherits IChatBackend + IInferenceBackend)
  chat/ChatSession.hpp/.cpp  Stateful multi-turn manager (sliding window, pinned context)
  chat/IChatBackend.hpp    Forwarding header → tritonic/chat/ichat_backend.hpp
  common/IInferenceBackend.hpp  Forwarding header → tritonic/core/interfaces.hpp
include/
  tritonic/core/types.hpp       Tensor, Message, ChatRequest/Response, BackendRequest/Response
  tritonic/core/interfaces.hpp  IInferenceBackend
  tritonic/triton/model_info.hpp   ModelInfo (namespace tritonic::triton)
  tritonic/triton/itriton.hpp      ITriton
  tritonic/triton/triton_backend.hpp  TritonBackend adapter
  tritonic/chat/ichat_backend.hpp  IChatBackend
  tritonic/infra/logger.hpp        Logger, LoggerManager, LogLevel
  tritonic/infra/config.hpp        InferenceConfig
  tritonic/infra/config_manager.hpp  ConfigManager
  App.hpp / CommonTypes.hpp / Logger.hpp / Config.hpp / ConfigManager.hpp
    ↑ backward-compat forwarding headers (kept for existing code)
tests/
  unit/            GoogleTest unit tests
  mocks/           MockTriton.hpp, MockChatBackend.hpp
  test_main.cpp    GoogleTest main runner
```

### Namespace layout

| Namespace | Headers | Contents |
|-----------|---------|----------|
| `tritonic::core` | `include/tritonic/core/` | `Tensor`, `Message`, `ChatRequest/Response`, `IInferenceBackend` |
| `tritonic::triton` | `include/tritonic/triton/` | `ITriton`, `ModelInfo`, `TritonBackend` |
| `tritonic::chat` | `include/tritonic/chat/` | `IChatBackend` |
| `tritonic::infra` | `include/tritonic/infra/` | `InferenceConfig`, `ConfigManager`, `Logger` |

Old flat `include/*.hpp` and `src/*/` headers are forwarding headers with backward-compat `using` aliases — do not revert them to full definitions.

### Data flow (Triton backend)

1. `client.cpp`: `ConfigManager::LoadFromCommandLine()` → creates `Triton` → creates `App`
2. `App::run()`: connects, checks liveness, loads model, fetches `ModelInfo`, creates task via `TaskFactory::createTaskInstance()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [olibartfast/tritonic](https://github.com/olibartfast/tritonic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
