---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Standard build
mkdir -p build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)

# Debug build
cmake .. -DCMAKE_BUILD_TYPE=Debug
make -j$(nproc)

# Configure without tests/examples
cmake .. -DCMAKE_BUILD_TYPE=Release -DCHRONON_BUILD_TESTS=OFF
make -j$(nproc)
```

## Testing

```bash
# Run all tests
cd build && ctest --output-on-failure

# Run tests in parallel
ctest --output-on-failure --parallel $(nproc)

# Run a specific test by name
ctest -R test_tree_node --output-on-failure

# Run test executable directly
./build/test_tree_node
./build/test/sender/test_sender
```

## Architecture Overview

Chronon is a high-performance tick-based simulation framework for CPU microarchitecture modeling written in C++20, using stdexec for parallel execution.

### Key Components

| Component | Location | Purpose |
|-----------|----------|---------|
| Sender Framework | `src/sender/` | Core simulation framework (Unit, TickSimulation, Port) |
| - Core | `src/sender/core/` | Unit, TickableUnit, TickSimulation, CrashHandler |
| - Port | `src/sender/port/` | OutPort, InPort, Connection, MessageQueue with auto-registration |
| - Schedule | `src/sender/schedule/` | DependencyGraph, CycleAnalyzer, WeightedPartitioner, SimulatedAnnealingPartitioner, TickCostProfiler, SchedulerTimelineTrace |
| - Utilities | `src/sender/util/` | Graph, StageReg, SingleStageReg, PipelinePhase, PriorityArbiter, VersionedRegister |
| Observability | `src/observe/` | Unified counters, traces, logs with macro-free API |
| Tree | `src/tree/` | TreeNode hierarchy for unit organization |
| Parameters | `src/params/` | Self-registering parameter system |
| Factory | `src/sender/factory/` | Unit factory for YAML-driven instantiation |
| Configuration | `src/sender/config/` | SenderConfigLoader, SenderSimulationBuilder for YAML parsing |
| App | `src/sender/app/` | SimulationApp unified entry point with CLI support |

**Execution paths** (selected automatically; both are cycle-accurate):
- **Sequential** — selected when parallelism is disabled, not beneficial, or fails the epoch-free safety gate
- **Epoch-free lookahead** — persistent parallel workers advance tight clusters against dependency-progress atomics, with no per-cycle or per-epoch barrier

There is no separate scheduler class; both paths live in `TickSimulation` (`TickSimulation.cpp`, `TickSimulationParallel.cpp`).

### Detailed Documentation

For comprehensive design documentation, see the Docusaurus site under `website/docs/guides/`:

- **[Architecture Overview](website/docs/guides/architecture.md)**: High-level component relationships
- **[Scheduling and Parallelization](website/docs/guides/scheduling.md)**: Dependency graph, lookahead, weighted partitioning
- **[Port System](website/docs/guides/port-system.md)**: OutPort/InPort, connection delays, queue types, backpressure
- **[Observability](website/docs/guides/observability.md)**: Counters, traces, logs with macro-free API
- **[Configuration](website/docs/guides/configuration.md)**: YAML-driven unit instantiation, SimulationApp

### Quick Reference

**Port delays and queue modes:**
- `delay=0` means same-cycle delivery eligibility.
- `delay>0` means delivery is delayed by N cycles.
- Queue implementation is selected during simulation initialization from thread topology and producer count: same-thread, SPSC, or MPSC.

**Port registration (automatic):**
```cpp
// Ports auto-register on construction. Signature: (owner, name, capacity?).
// Delay is a property of the Connection, not the port.
OutPort<Data> out{this, "out", 256};  // per-cycle capacity 256
InPort<Data> in{this, "in"};          // unlimited capacity by default
```

**Modern observability API:**
```cpp
// Define categories (auto-assigned bit positions)
inline const auto MY_CATEGORY = Category<"my_category", "Description">{};

// Declare counters as unit members
EventCounter ops_{this, "ops", "Operations executed"};

// Use in ObservableUnit (no macros needed)
++ops_;                                      // Increment counter
event<"event">(MY_CATEGORY, arg<"value">(value)); // Emit structured timeline event
debug<"Debug info: {}">(value);              // Debug log
info<"Info message">();                      // Info log
```

**Simplified headers:**
```cpp
// Single include for all Chronon functionality
#include "chronon/Chronon.hpp"
using namespace chronon;

// All types available directly in chronon:: namespace:
// - TickableUnit, TickSimulation, TickSimulationConfig
// - OutPort<T>, InPort<T>, Connection<T>
// - EventCounter, Category<>, ObservableUnit
// - Param<T>, ParameterSet, AutoRegisteredUnit
// - StageReg<T, N>, SingleStageReg<T>
// - TerminationReason, TerminationRequest, TerminationController
```

**Unit constructor pattern (with YAML support):**
```cpp
// Define ParameterSet for your unit
struct MyUnitParams : public ParameterSet {
    Param<uint32_t> width{this, "width", 4, "Processing width"};
    Param<uint32_t> depth{this, "depth", 8, "Queue depth"};
};

// Use CHRONON_UNIT_CONSTRUCTOR macro to generate dual constructors

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chronon-sim/chronon](https://github.com/chronon-sim/chronon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
