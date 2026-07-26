---
trigger: always_on
description: > This document is maintained by LLMs working on the gfx module. See [Maintenance](#maintenance) section for update rules.
---

# GFX Module Architecture

> This document is maintained by LLMs working on the gfx module. See [Maintenance](#maintenance) section for update rules.

## Module Overview

`hitagi::gfx` is a graphics device abstraction layer providing a unified API over DX12 and Vulkan backends.
`hitagi::rg` is a frame-scoped render graph built on top of gfx.

**Modules**: `gfx.base` (abstract API) → `gfx.dx12` / `gfx.vulkan` / `gfx.mock` (backends) → `gfx.render_graph`

## File Map

| Directory | Module | Purpose |
|---|---|---|
| `base/` | `gfx.base` | Abstract GPU API — all types, virtual interfaces, shader compiler |
| `dx12/` | `gfx.dx12` | DX12 backend (Windows-only) using D3D12 Agility SDK + D3D12MA |
| `vulkan/` | `gfx.vulkan` | Vulkan backend (Windows + Linux) using Vulkan-Hpp RAII + VMA |
| `mock/` | `gfx.mock` | Null backend for unit tests — all methods are no-ops |
| `render_graph/` | `gfx.render_graph` | Frame-scoped render graph with typed handles and pass builders |
| `test/` | — | GTest-based tests: device_test, render_graph_test, shader_compiler_test, desc_hash_test |

### Key Files per Directory

**`base/`** — 8 module partitions composing `gfx.base`:

| File | Partition | Key Exports |
|---|---|---|
| `types.cppm` | `:types` | `Format`, `GPUBufferUsageFlags`, `TextureUsageFlags`, `ShaderType`, `BarrierAccess`, `PipelineStage`, `TextureLayout`, pipeline states |
| `gpu_resource.cppm` | `:gpu_resource` | `Resource`, `ResourceWithDesc<Desc>` (CRTP), `GPUBuffer`, `Texture`, `Sampler`, `SwapChain`, `Shader`, `RenderPipeline`, `ComputePipeline`, all `*Desc` structs |
| `sync.cppm` | `:sync` | `Fence`, `GlobalBarrier`, `GPUBufferBarrier`, `TextureBarrier`, `FenceSignalInfo`, `FenceWaitInfo` |
| `bindless.cppm` | `:bindless` | `BindlessHandle`, `BindlessMetaInfo`, `BindlessUtils` |
| `command_context.cppm` | `:command_context` | `CommandType`, `CommandContext`, `GraphicsCommandContext`, `ComputeCommandContext`, `CopyCommandContext` |
| `command_queue.cppm` | `:command_queue` | `CommandQueue` |
| `shader_compiler.cppm` | `:shader_compiler` | `ShaderCompiler` (DXC-based; DXIL and SPIR-V output) |
| `utils.cppm` | `:utils` | `format_as()` for fmt, `get_format_bit_size`, `split_semantic` |
| `device.cppm` | primary | `Device` (abstract, extends `core::RuntimeModule`) — factory for all GPU objects |

**`dx12/`** — 8 module partitions composing `gfx.dx12`:

| File | Partition | Contains |
|---|---|---|
| `dx12_device.cppm` | primary | `DX12Device` — IDXGIFactory2, IDXGIAdapter4, ID3D12Device, D3D12MA::Allocator |
| `dx12_resource.cppm` | `:resource` | `DX12GPUBuffer`, `DX12Texture`, `DX12Sampler`, `DX12Shader`, `DX12RenderPipeline`, `DX12ComputePipeline`, `DX12SwapChain` |
| `dx12_command_list.cppm` | `:command_list` | `DX12GraphicsCommandList`, `DX12ComputeCommandList`, `DX12CopyCommandList` |
| `dx12_command_queue.cppm` | `:command_queue` | `DX12CommandQueue` — wraps `ID3D12CommandQueue` + internal `DX12Fence` for WaitIdle |
| `dx12_descriptor_heap.cppm` | `:descriptor_heap` | `Descriptor`, `DescriptorHeap`, `DescriptorAllocator` — CPU-side RTV/DSV only |
| `dx12_sync.cppm` | `:sync` | `DX12Fence` — wraps `ID3D12Fence` + Win32 event handle |
| `dx12_bindless.cppm` | `:bindless` | `DX12BindlessUtils` — CBV_SRV_UAV heap + Sampler heap + shared root signature |
| `dx12_utils.cppm` | `:utils` | Format/barrier/state conversion functions (`to_dxgi_format`, `to_d3d_barrier_access`, etc.) |

**`vulkan/`** — 9 module partitions composing `gfx.vulkan`:

| File | Partition | Contains |
|---|---|---|
| `vk_device.cppm` | primary | `VulkanDevice` — vk::raii::Instance/PhysicalDevice/Device, VmaAllocator |
| `vk_resource.cppm` | `:resource` | `VulkanBuffer`, `VulkanImage`, `VulkanSampler`, `VulkanSwapChain`, `VulkanShader`, `VulkanRenderPipeline`, `VulkanComputePipeline` |
| `vk_command_buffer.cppm` | `:command_buffer` | `VulkanGraphicsCommandBuffer`, `VulkanComputeCommandBuffer`, `VulkanTransferCommandBuffer` |
| `vk_command_queue.cppm` | `:command_queue` | `VulkanCommandQueue` — wraps vk::raii::Queue with queue family index |
| `vk_sync.cppm` | `:sync` | `VulkanTimelineSemaphore` — wraps `vk::raii::Semaphore` (timeline type) |
| `vk_bindless.cppm` | `:bindless` | `VulkanBindlessUtils` — descriptor pool + set layouts + pipeline layout |
| `vk_configs.cppm` | `:configs` | Required instance/device extensions, descriptor pool sizes |
| `vk_utils.cppm` | `:utils` | Format/barrier/state conversion functions, physical device scoring, queue helpers |
| `vma_patch.cpp` | — | VMA translation unit (separate static lib `vma_patch`) |

**`render_graph/`** — 5 module partitions composing `gfx.render_graph`:

| File | Partition | Contains |
|---|---|---|
| `type.cppm` | `:type` | `RenderGraphNode`, `RenderGraphHandle<T>` template, all 9 typed handle aliases |
| `resource_edge.cppm` | `:resource_edge` | `GPUBufferEdge`, `TextureEdge`, `SamplerEdge` — carry access/stage/layout metadata |
| `resource_node.cppm` | `:resource_node` | `ResourceNode` hierarchy: `GPUBufferNode`, `TextureNode`, `SamplerNode`, `RenderPipelineNode`, `ComputePipelineNode` |
| `pass_node.cppm` | `:pass_node` | `PassNode` hierarchy: `RenderPassNode`, `ComputePassNode`, `CopyPassNode`, `PresentPassNode` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [L-Sun/HitagiEngine](https://github.com/L-Sun/HitagiEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
