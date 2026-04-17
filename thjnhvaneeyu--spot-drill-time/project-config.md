---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains a comprehensive collection of optimization algorithms for **Arc Proton Therapy** treatment planning, focusing on spot drill time optimization and dose delivery efficiency. The codebase implements multiple mathematical norm approaches based on research by Sophie Wuyckens et al.

**Author**: Huỳnh Khôi Minh Uyên  
**Last Updated**: 2025-08-15  
**Status**: Complete analysis with 21 algorithms implemented and validated

## Common Development Commands

### Running Individual Algorithms
```bash
# FISTA algorithms  
python fista_proton_optimization_demo.py         # Quick demonstration
python fista_proton_optimization_improved.py     # Advanced features

# ADMM variants
python ADMM_L1.py                    # L1 regularization (high sparsity)
python ADMM_L2.py                    # L2 regularization (smooth weights)
python ADMM_SoftThreshold.py         # Adaptive thresholding
python ADMM_L1_SoftThreshold.py      # Combined L1 + soft thresholding
python ADMM_L2_SoftThreshold.py      # L2 smoothness + selective sparsity

# Advanced norm implementations
python FISTA_L2_L1_GroupLasso.py     # Group Lasso (convex)
python ELOSPAT_L2_L1half_NonConvex.py # Non-convex L2,1/2 norm
python LocalSearch_L0_DirectSparsity.py # Direct L0 control
python SPArc_L0_Heuristic.py         # L0 heuristic approximation  
python MIP_L0_Exact.py               # Exact L0 via Mixed-Integer Programming
python ELSA_L0_L2_Hybrid.py          # Two-phase L0+L2 hybrid

# Main optimization wrappers
python SPArc_optimization.py
python LocalSearch_optimization.py
python MIP_optimization.py
python ELOSPAT_optimization.py
python ELSA_optimization.py
```

### Comprehensive Analysis and Visualization
```bash
# Complete analysis pipeline (RECOMMENDED)
python complete_visualization_pipeline.py   # Process all 21 algorithms with extended analysis

# Standard analysis pipeline  
python integrated_visualization_pipeline.py  # Process 17 core algorithms

# File standardization and analysis
python standardize_analyze_files.py         # Analyze algorithm implementations
```

### Algorithm Comparison
```bash
python algorithm_comparison.py       # Compare all implemented algorithms
```

### Testing and Validation
```bash
python test.py                      # Basic functionality tests
```

## Core Architecture

### Mathematical Framework
The repository implements multiple mathematical norm approaches for the general optimization problem:
```
minimize: f(d) + Regularization(x) + γ·ELST(x)
```
Where:
- `f(d) = ||Ax - d₀||²` (dose fidelity term)
- `Regularization(x)` varies by algorithm (L1, L2, L0, Group Lasso, etc.)
- `ELST(x)` is Energy Layer Switching Time penalty
- `A` is the kernel fluence matrix (voxels × spots)
- `x` are the spot weights to optimize

### Algorithm Categories

1. **FISTA-based algorithms** (`fista_*.py`): Fast proximal gradient methods
2. **ADMM variants** (`ADMM_*.py`): Alternating Direction Method of Multipliers  
3. **Local Search methods** (`LocalSearch_*.py`): Heuristic optimization with neighborhood operators
4. **Mixed-Integer Programming** (`MIP_*.py`): Exact L0 optimization with binary variables
5. **SPArc methods** (`SPArc_*.py`): Beam splitting and layer selection heuristics
6. **ELSA variants** (`ELSA_*.py`): Hybrid L0+L2 two-phase optimization
7. **ELOSPAT methods** (`ELOSPAT_*.py`): Non-convex L2,1/2 norm optimization
8. **Norm-specific implementations**: Each mathematical norm has dedicated implementation
9. **Optimization wrappers**: High-level interfaces for complete workflows

### Complete Algorithm Portfolio (21 Algorithms)

#### Standard Implementations (17)
- **ADMM Family**: L1, L2, SoftThreshold, L1_SoftThreshold, L2_SoftThreshold
- **FISTA Family**: Demo, Improved, L2_L1_GroupLasso  
- **Local Search**: Standard, L0_DirectSparsity
- **MIP**: Standard, L0_Exact
- **SPArc**: Standard, L0_Heuristic
- **ELSA**: Standard, L0_L2_Hybrid
- **ELOSPAT**: L2_L1half_NonConvex

#### Algorithm Variants (4)
- **Alternative**: ELOSPAT_Alternative (alternative L2,1/2 implementation)
- **Original**: FISTA_Demo_Original, FISTA_Improved_Original (original versions)
- **Reduced**: ELSA_L2_Reduced (reduced-dimension model)

### Data Requirements
All algorithms require:
- `Kernel_targetPID9174251.mat`: MATLAB file containing sparse matrix `Target_fluence` (105 voxels × 4934 spots)
- Prescribed dose (typically 1.8 Gy)

### Algorithm Output Files
Each algorithm generates standardized `.npy` files:
- **Standard outputs**: `[algorithm]_optimal_weights.npy`
- **Special analysis**: `mip_l0_exact_binary.npy`, `elsa_l0_selection_mapping.npy`
- **Variants**: `[algorithm]_[variant]_weights.npy`

### Results and Analysis
The repository includes comprehensive analysis infrastructure:

#### Analysis Directories
- **`integrated_results/`**: Standard 17-algorithm analysis
- **`complete_results/`**: Extended 21-algorithm analysis with variants
- **Individual plots**: Per-algorithm detailed visualizations
- **Comparison dashboards**: Multi-algorithm performance analysis
- **CSV reports**: Quantitative comparison data

#### Key Analysis Files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thjnhvaneeyu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
