---
trigger: always_on
description: - **Never use em dashes** (the "—" character). Use parentheses, commas, colons, semicolons, or split into separate sentences instead.
---

# FlashInfer AI Kernel Generation Contest - GDN Track

## Writing Style
- **Never use em dashes** (the "—" character). Use parentheses, commas, colons, semicolons, or split into separate sentences instead.

## Code Style
- **Don't duplicate logic across local and Modal scripts.** Extract shared logic into a function in the local script, then have the Modal script import and call it (see `bench_nvbench.py` / `bench_nvbench_modal.py` pattern).

## Team
- Team name: **lmu-css**
- Track: **gated_delta_net** (Gated Delta Net)

## Project Structure
- `config.toml` — Solution metadata and build config. `definition` must match the exact definition name (e.g. `gdn_decode_qk4_v8_d128_k_last`), not the track name.
- `solution/triton/kernel.py` — Triton/Python kernel implementation. Entry point is a regular Python function (not necessarily `@triton.jit`).
- `solution/cuda/kernel.cu` — CUDA C++ kernel with TVM FFI export (`TVM_FFI_DLL_EXPORT_TYPED_FUNC`).
- `solution/cuda/binding.py` — Local Python binding via `tvm_ffi.cpp.build()` + `@register_global_func`.
- `scripts/` — Python package (has `__init__.py`). All scripts run as modules: `python -m scripts.X`.
  - `shared.py` — Shared constants: `ALGO_ENTRY_POINTS`, `ALGO_LANGUAGES`, `PROJECT_ROOT`, `parse_args()`.
  - `pack_solution.py` — Packs solution into `solution.json`.
  - `run_local.py` — Local benchmark runner.
  - `run_modal.py` — Cloud benchmark (B200 GPUs via Modal).
  - `modal_config.py` — Shared Modal infrastructure (image, volume, `TRACE_SET_PATH`).
  - `profile_proton.py` — Proton intra-kernel profiling. Also exports `load_workload_tensors()`.
  - `profile_ncu.py` — NCU profiling (launched by `ncu`, not run directly).
  - `bench_nvbench.py` — NVBench timing validation.
  - `bench_nvbench_modal.py` — NVBench on Modal B200.
  - `bench_fi_timing.py` — FlashInfer `bench_gpu_time` with CUPTI (pure GPU kernel time). Also used by Modal script.
  - `bench_fi_timing_modal.py` — FlashInfer CUPTI timing on Modal B200.
  - `log_speedups.py` — Parse bench logs into `findings/speedups.csv`.

### Import conventions
- `scripts/` and `solution/` are both Python packages (have `__init__.py`).
- All scripts are invoked as modules: `python -m scripts.run_local`, `modal run -m scripts.run_modal`.
- Within `scripts/`, use **relative imports**: `from .shared import ALGO_ENTRY_POINTS`.
- For kernel imports: `from solution.triton.kernel import ...` (works because `-m` adds CWD to sys.path).
- No `sys.path` manipulation, except inside Modal remote functions (container mounts at `/root/`).
- To add a new algo: add one entry to `ALGO_ENTRY_POINTS` in `shared.py`, add one wrapper function in `kernel.py`. For non-Triton algos, also add an entry to `ALGO_LANGUAGES`.

## Environment
- Venv: `.venv/` in project root
- Packages: `flashinfer-bench`, `modal`, `torch`, `triton`
- Dataset: `~/code/mlsys26-contest` (env var `FIB_DATASET_PATH`, set in `~/.bashrc`)

## Config Notes
- `entry_point` format: `kernel.py::kernel_fla_recurrent` (Triton) or `kernel.cu::kernel_cuda` (CUDA TVM FFI)
- `definition` must be the exact definition name from the dataset (e.g. `gdn_decode_qk4_v8_d128_k_last`), not the track name (`gated_delta_net`)
- DPS (Destination Passing Style) is default: kernel receives pre-allocated output tensors as extra args
- `language="triton"` uses PythonBuilder (imports .py, calls function). `language="cuda"` uses TVMFFIBuilder (compiles .cu, exports via TVM FFI). The `ALGO_LANGUAGES` dict in `shared.py` overrides the default from config.toml.

## What is GDN?

GDN (Gated Delta Net) is an **alternative to softmax attention** for LLMs. It replaces the O(n) per-token attention decode with an O(1) recurrent state update. Used in production: Qwen3-Next-80B (75% GDN layers), Kimi Linear-48B.

### GDN vs Attention at decode time
|                      | Compute per token                         | Memory                             |
| -------------------- | ----------------------------------------- | ---------------------------------- |
| **Causal Attention** | O(n) — dot product with all n cached keys | O(n) — KV cache grows with context |
| **GDN**              | O(1) — fixed 128x128 state matrix ops     | O(1) — constant state size         |

### Where GDN sits in the transformer
```
x = x + gdn_layer(norm(x))    # replaces attention sublayer
x = x + mlp(norm(x))          # FFN sublayer unchanged
```

### Full GDN layer (our kernel is the middle part)
```python
# --- Input projections (outside our kernel) ---
q = x @ W_q                    # [B, 1, num_q_heads, K]
k = l2_normalize(x @ W_k)     # [B, 1, num_k_heads, K]  (L2-normalized!)
v = x @ W_v                    # [B, 1, num_v_heads, V]
a = x @ W_a                    # [B, 1, num_v_heads]
b = x @ W_b                    # [B, 1, num_v_heads]

# --- Our kernel (the competition part) ---
g     = exp(-exp(A_log) * softplus(a + dt_bias))   # global decay ∈ (0,1)
beta  = sigmoid(b)                                  # update gate ∈ (0,1)
S_new = g * S - k^T @ (k @ S) + k^T @ (beta * v + (1-beta) * (k @ S))
out   = scale * q @ S_new

# --- Output projection (outside our kernel) ---
out = reshape(out) @ W_o       # back to hidden dim
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomasruizt/flashinfer-competition-codebase](https://github.com/tomasruizt/flashinfer-competition-codebase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
