---
trigger: always_on
description: **cuacd** — GPU convex approximate decomposition. Single CPython extension (abi3, cp310+) via CUDA driver API. Sub-algorithms: D&C convex hull, plane cut, lookahead tree search decomposition. All run on GPU with a custom heap allocator.
---

# CLAUDE.md

## Project Overview

**cuacd** — GPU convex approximate decomposition. Single CPython extension (abi3, cp310+) via CUDA driver API. Sub-algorithms: D&C convex hull, plane cut, lookahead tree search decomposition. All run on GPU with a custom heap allocator.

`CoACD/` — Reference C++ CoACD implementation (SIGGRAPH 2022), not part of the GPU extension.

## Repository Layout

```
setup.py              # Build config: setuptools builds _gpu extension
pyproject.toml        # PEP 621 metadata
cuda/                 # CUDA device code (compiled to single fatbin, shipped as cuacd/kernels.fatbin)
  allocator.cuh       #   DevicePool (bump alloc + embedded DeviceHeap×2) + pool_alloc + heap_alloc/free
  common.cuh          #   Constants, atomics, CheckedBuf<T> (includes allocator.cuh)
  reduce.cuh          #   Block-level parallel reductions (sum, bbox)
  geometry.cuh        #   signed_tet_volume (divergence theorem kernel helper)
  warp_common.cuh     #   WarpPool allocator + warp reductions
  hull_dandc.cuh      #   Preparata-Hong D&C hull (Bullet port)
  warp_sort.cuh       #   Generic warp-cooperative quicksort template + BtPoint32 legacy API
  plane_cut.cuh       #   plane_cut_block device function; returns PartPair via DeviceHeap
  kdop_hull.cuh       #   kdop_hull_block: single-warp exact hull via extreme-point prefilter + D&C
  mesh_volume.cuh     #   mesh_volume_warp: per-warp divergence theorem volume
  hausdorff.cuh       #   hausdorff_block: block-level bidirectional Hausdorff via sampling + linear BVH
  postprocess.cuh     #   decompose_components_block: block-level connected-components via union-find
  merge.cuh           #   Merge-hulls helpers: flat upper-tri pair index, AABB gap, warp concat, shared-plane finder
  structs.cuh         #   Device-side: Mesh, Part, PartPair, LaWorkItem, LaDecompState, LaEvalResult, ConcaveEdgePlane
  error_codes.cuh     #   Centralized GPU kernel error flags (KERR_*)
  la_common.cuh       #   Shared lookahead constants + la_part_cost/la_part_cost_rv inline functions
  mm.cu               #   heap_init_kernel
  kdop_const.cu       #   __constant__ KDOP_AXES[40][3] (icosphere axes)
  la_expand.cu        #   la_expand, la_hull, la_compute_best_ub, la_seed_tree, la_cleanup_tree, la_cleanup_tree3
  la_refine.cu        #   la_expand_quick, la_hausdorff_parts, la_evaluate, la_sort_and_record
  la_lifecycle.cu     #   la_initialize, la_sort_and_count_cutting, la_apply_cuts, la_hull_decomp, la_decompose_components, la_free_decomp
  la_concave.cu       #   la_find_concave_edges: concave edge detection + plane generation
  postprocess_merge.cu #   Merge-hulls pass: la_merge_cost_matrix, la_merge_hausdorff, la_merge_match, la_merge_apply, la_merge_free_unused, la_merge_compact
  mesh_audit.cuh      #   mesh_audit_kernel: watertight/manifold/orientation/degenerate audit → MAV_* bitmask
  pdmc_kernels.cu     #   PDMC Dual Marching Cubes device kernels (SarahWeiii/pdmc port, verbatim tables)
  c2s_kernels.cu      #   cumesh2sdf SDF kernels (eliphatfs/cumesh2sdf port: rasterize layers, sign fill, sdf shift)
  test_*.cu           #   Test kernels
csrc/                 # C host code
  structs.h           #   Host-side structs: DevicePool, HeapArena, DeviceHeap, gpu_ctx
  heap.h / heap.c     #   GPU context lifecycle + pool management
  test.h / test.c     #   Kernel host launchers (incl. gpu_mesh_audit)
  preprocess.h/c      #   gpu_preprocess host orchestration (c2s SDF + pdmc DMC), driver API only
  postprocess.h/c     #   Post-processing host launchers
  error_codes.h       #   Host-side error decoding (kerr_decode)
  lookahead.h/c       #   lookahead_decompose host implementation
  module.c            #   CPython extension (Py_LIMITED_API cp310)
cuacd/                # Python package
  __init__.py         #   Context class
  cli.py              #   cuacd console entry point
tests/                # All tests + benchmarks
bench/                # Standalone perf experiments (not built by setup.py)
  warp_sort_bench.cu  #   std::sort vs warp_sort vs cub BlockMergeSort/BlockRadixSort (+4-pass int4 radix)
  Makefile            #   warp_sort bench build (ARCH=89 default); `make verbose` for ptxas -v
  hull_bench.cu       #   btConvexHull (CPU) vs hull_dandc vs kdop_hull (batch=256 × {512,2048,16384,65536} pts × gaussian/uniform_cube)
  Makefile.hull       #   hull bench build (rdc=true; links mm.cu, kdop_const.cu, CoACD btConvexHull .cpp)
  malloc_test_bench.cu #  mstress-style alloc/free stress: coacd heap vs CUDA __device__ malloc; sweeps grid size for arena-contention curve
  Makefile.malloc     #   malloc bench build (rdc=true; links mm.cu)
  plot_malloc_and_hausdorff.py  #   Two-panel figure: malloc throughput + Hausdorff CPU vs GPU bars
  prepare_hausdorff_data.py     #   Load meshes, normalize, convex hull → hausdorff_data.h
  hausdorff_bench.cu            #   GPU hausdorff_block benchmark (matches warp_sort_bench timing style)
  bench_cpu_hausdorff.cpp       #   CPU CoACD face_hausdorff_distance wrapper (nanoflann, no openvdb)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliphatfs/cuacd](https://github.com/eliphatfs/cuacd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
