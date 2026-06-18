---
trigger: always_on
description: We are using `claude-opus-4-6`!
---

# Engineering Notes

We are using `claude-opus-4-6`!

## Project Map

```
nnmetal/
├── build.zig                    152 lines   Build config
├── build.zig.zon                             Package manifest
├── examples/
│   ├── bonsai.zig               553 lines   Bonsai tree classifier
│   ├── bonsai_bench.zig         636 lines   Bonsai benchmarking
│   ├── inference_bench.zig      681 lines   Inference benchmarking
│   ├── mnist.zig              1,042 lines   MNIST training
│   └── mnist_1bit.zig          803 lines   1-bit MNIST variant
├── src/
│   ├── benchmark.zig            706 lines   Benchmarking infra
│   ├── layout.zig               636 lines   Comptime network layout
│   ├── metal.zig              1,568 lines   Metal GPU bindings
│   ├── mnist.zig                407 lines   MNIST data loading
│   ├── model.zig              1,276 lines   Model (safetensors/loading)
│   ├── network.zig            3,308 lines   Core network (forward/backward/train)
│   ├── root.zig                  49 lines   Public re-exports
│   ├── safetensors.zig          736 lines   Safetensors format parser
│   ├── shaders/
│   │   ├── compute.metal      4,675 lines   GPU compute kernels
│   │   ├── qmv_specialized.metal  902 lines   Quantized matmul kernels
│   │   └── transformer.metal  1,343 lines   Transformer-specific kernels
│   ├── specialized_qmv.zig     132 lines   Specialized quantized matmul
│   ├── tokenizer.zig          1,574 lines   Tokenizer
│   └── transformer.zig       5,982 lines   Transformer implementation
├── benchmarks/                               JSON benchmark results
└── data/mnist/                               MNIST raw dataset
                              ──────
                              27,161 lines total

labrat/
├── build.zig                                 Build config
├── build.zig.zon                             Package manifest
├── src/
│   ├── agent_core.zig        2,271 lines   Shared agent framework
│   ├── api_client.zig          786 lines   Anthropic HTTP client
│   ├── bonsai_agent.zig        468 lines   Bonsai agent profile
│   ├── bonsai_researcher.zig    93 lines   Bonsai researcher config
│   ├── mnist_agent.zig         529 lines   MNIST agent profile
│   ├── mnist_researcher.zig    885 lines   MNIST researcher config + custom tools
│   ├── toolbox.zig            2,399 lines   Generic toolbox (shared)
│   └── tools.zig               699 lines   Shared CLI/file utilities
├── programs/
│   ├── bonsai_system.md                      Bonsai system prompt
│   └── mnist_system.md                       MNIST system prompt
├── benchmarks/                               JSON benchmark results
└── data/mnist/                               MNIST raw dataset
```

Heavy hitters: `transformer.zig`, `network.zig`, `compute.metal` (~14k lines, half the nnmetal codebase).
Comptime spine: `layout.zig` resolves all buffer shapes at compile time.
Three shader files: `compute.metal` (general NN kernels), `transformer.metal` (attention-specific), `qmv_specialized.metal` (quantized matmul).
Agent spine: `agent_core.zig` provides the shared experiment loop; profiles (`mnist_agent.zig`, `bonsai_agent.zig`) configure it.
Toolbox spine: `toolbox.zig` provides all generic CLI tools; domain binaries (`bonsai_researcher.zig`, `mnist_researcher.zig`) are thin config wrappers.

### 0. **Simplicity and Elegance**

Simplicity is not a free pass or a first attempt — it is the hardest revision. The goal is to find the "super idea" that solves safety, performance, and developer experience simultaneously. An hour or day of design is worth weeks or months in production. Spend mental energy upfront, proactively rather than reactively, because when the thinking is done, what is spent on the design will be dwarfed by implementation, testing, and maintenance.

> "Simplicity and elegance are unpopular because they require hard work and discipline to achieve." — Edsger Dijkstra

It's easy to say "let's do something simple", but to do that in practice takes thought, multiple passes, many sketches, and still we may have to throw one away. For nnmetal, this means: don't settle on the first kernel dispatch strategy or buffer layout that works. Sketch alternatives. The elegant design will be the one where the Metal buffer layout, the comptime network description, and the kernel dispatch strategy all reinforce each other.

### 1. **Zero Technical Debt Policy**

_Solve problems correctly the first time_. When you encounter a potential latency spike or algorithmic issue, fix it now — don't defer. The second pass may never come.

### 2. **Static Memory Allocation**

This is huge for a GPU compute library:

- **No dynamic allocation after initialization**
- Pre-allocate all Metal shared buffers (params, grads, activations) at startup
- Use fixed-capacity arrays/pools instead of growing `ArrayList`s during training
- Use comptime-known sizes from `NetworkLayout` to determine buffer capacities at compile time

For nnmetal, this means: parameter buffers, gradient buffers, and activation buffers should have fixed upper bounds allocated at init time. This eliminates allocation jitter during forward/backward passes and prevents GPU stalls caused by buffer reallocation.

### 3. **Assertion Density**

**Minimum 2 assertions per function**. For nnmetal:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duanebester/nnzap](https://github.com/duanebester/nnzap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
