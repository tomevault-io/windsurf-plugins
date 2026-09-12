---
trigger: always_on
description: Copyright 2026 FlagOS Contributors
---

<!--
 Copyright 2026 FlagOS Contributors

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
 -->

<!--
 Copyright 2026 FlagOS Contributors

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
 -->

<!--
 Copyright 2026 FlagOS Contributors

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
 -->

<!--
 Copyright 2026 FlagOS Contributors

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
 -->

# AGENTS.md

Guidance for AI coding agents (Claude Code, Gemini, Copilot, Codex, Cursor, etc.)
working in this repository. Human contributors may also find it useful.

## Project overview

**FlagGems-vllm** is a high-performance operator library for vLLM, part of
[FlagOS](https://flagos.io/). Operators are written in the
[Triton](https://github.com/openai/triton) programming language (FlagTree in
production) and target multiple hardware backends (NVIDIA by default).

The Python package is `flaggems_vllm` and lives under `src/`. It exposes
vLLM-scenario fused kernels — e.g. `flaggems_vllm.grouped_topk`,
`flaggems_vllm.fused_experts_impl`, `flaggems_vllm.ops.moe_align_block_size`.

### Relationship with sibling repos

- **FlagGems** — general-purpose operator backend; registers ops into PyTorch
  dispatch via `flag_gems.enable()` / `flag_gems.use_gems()`.
- **FlagGems-vllm** (this repo) — vLLM-specific fused kernels plus aligned
  tests/benchmarks. Exposes ops through the `flaggems_vllm` package.
- **vllm-plugin-fl** — the vLLM plugin layer; calls `flag_gems.enable()` for
  general ops and imports `flaggems_vllm.<operator>()` for vLLM fused kernels.

Call flow: `vLLM -> vllm-plugin-fl -> flag_gems.enable() + flaggems_vllm.<op>()`.

## Repository layout

```
src/flaggems_vllm/
  __init__.py              # top-level exports, enable()/only_enable()/use_gems()
  config.py                # enable/exclude config resolution, C-extension detection
  ops/                     # operator implementations (one file per op, mostly)
    __init__.py            # re-exports every public op + __all__
    <op>.py                # host dispatch + Triton kernels for <op>
    DSA/  FLA/  mhc/        # grouped op families (subpackages)
  runtime/
    __init__.py            # device detection, get_tuned_config(), torch_device_fn
    register.py            # Register class (PyTorch dispatch registration)
    configloader.py        # loads tune_configs.yaml / heuristics configs
    backend/_<vendor>/     # per-vendor config: tune_configs.yaml, heuristics, arch dirs
  utils/                   # libentry, libtuner, triton helpers, codegen, dtype utils
  testing/
tests/                     # functional tests: tests/test_<op>.py (+ test_FLA/, test_DSA/)
  conftest.py              # pytest hooks, --quick / --ref / --record options
  accuracy_utils.py        # to_reference(), gems_assert_* comparison helpers
benchmark/                 # perf tests: benchmark/test_<op>.py (+ conftest, base, consts)
  core_shapes.yaml         # recommended shapes per op
tools/                     # setup.sh (env bootstrap), select_tests.py (CI selection), etc.
workflow.md                # MANDATORY operator-development protocol (read before coding ops)
```

Supported backend vendor dirs live under `src/flaggems_vllm/runtime/backend/`:
`_nvidia` (default), `_amd`, `_metax`, `_iluvatar`, `_cambricon`, `_ascend`,
`_hygon`, `_mthreads`, `_kunlunxin`, and others. NVIDIA additionally has

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flagos-ai/FlagGems-vllm](https://github.com/flagos-ai/FlagGems-vllm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
