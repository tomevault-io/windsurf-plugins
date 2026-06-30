---
trigger: always_on
description: Ptera Software is a fast, easy-to-use, and open-source package for analyzing flapping-wing flight using unsteady and steady vortex lattice methods.
---

# Ptera Software Development Guidelines for Claude

## Project Overview

Ptera Software is a fast, easy-to-use, and open-source package for analyzing flapping-wing flight using unsteady and steady vortex lattice methods.

### Key Features

- **Multiple Simulation Methods**: Steady horseshoe VLM, steady ring VLM, and unsteady ring UVLM
- **Customizable Aircraft Geometry**: Multi-wing aircraft with arbitrary wing cross sections and airfoils
- **Time-Dependent Motion**: Custom prescribed flapping motions
- **Formation Flight**: Multi-airplane simulations supported since v2.0.0
- **High-Speed Computing**: JIT compilation via Numba for fast simulations
- **Trim Analysis**: Automatic search for trim operating points
- **Convergence Analysis**: Automatic search for converged parameters
- **Aeroelasticity**: First-order structural wing deformation coupled to the UVLM via a torsional spring-mass-damper model (beta)
- **Free Flight**: Six-degree-of-freedom flight dynamics from UVLM aerodynamics coupled to MuJoCo rigid-body dynamics (beta)
- **Visualization Tools**: 3D mesh visualization and 2D plotting of results
- **Save and Load**: JSON serialization of solved simulations without pickle security risks
- **Extensive Testing**: Comprehensive unit and integration tests for reliability

### Python Version Constraint

Requires Python 3.11, but active development is done in 3.13

### Key Runtime Dependencies

- **NumPy/SciPy**: Core numerical computations
- **Numba**: JIT compilation for performance-critical loops
- **PyVista**: 3D mesh processing and visualization
- **Matplotlib**: 2D plotting and analysis output

## Architecture Overview

### Relevant Directories, Packages, and Files

- `.github/`: Directory with GitHub configuration files
    - `ISSUE_TEMPLATE/`: Directory with issue templates
        - `bug_report.md`
        - `feature_request.md`
    - `workflows/`: Directory with GitHub Actions workflows
        - `ascii-only.yml`
        - `black.yml`
        - `codespell.yml`
        - `docformatter.yml`
        - `isort.yml`
        - `label-sync.yml`
        - `mypy.yml`
        - `pre-commit-hooks.yml`
        - `publish.yml`
        - `tests.yml`
    - `CODEOWNERS`
    - `dependabot.yml`
    - `FUNDING.yml`
    - `labels.yml`
    - `pull_request_template.md`
- `.venv/`: Directory for the Python virtual environment, configured for the host machine's OS (not included in version control)
- `.venv-wsl/`: Directory for the Python virtual environment configured for a WSL OS (not included in version control, may be missing if host machine doesn't use WSL for development)
- `experimental/`: Directory with experimental scripts and prototypes (not included in version control)
- `docs/`: Directory with documentation files
    - `examples_expected_output/`: Example output files for verification
    - `hero_graphics/`: Assets for the README hero graphic
    - `private/`: Directory with documentation not included in this repository's version control (may be missing if the private repo hasn't been cloned and linked to this local repo)
        - `katz_plotkin_12_2/`: A recreation of Chapter 12.2, which describes efficiently including the effects of symmetry and ground effect for vortex lattice methods, from the textbook "Low-Speed Aerodynamics" by Katz and Plotkin
        - `katz_plotkin_13_12/`: A recreation of Chapter 13.12, which describes the UVLM, from the textbook "Low-Speed Aerodynamics" by Katz and Plotkin
        - `katz_plotkin_d/`: A recreation of Appendix D, which includes example Fortran programs, from the textbook "Low-Speed Aerodynamics" by Katz and Plotkin
        - `lambert_2015_2_3__2_4/`: A recreation of Sections 2.3 and 2.4 from Thomas Lambert's thesis "Modeling of aerodynamic forces in flapping flight with the unsteady vortex lattice method"
    - `website/`: Directory with the source files for generating the documentation website
    - `ANGLE_VECTORS_AND_TRANSFORMATIONS.md`: Conventions and definitions for angle vectors and transformations **READ BEFORE CONTRIBUTING ANY CODE, PARTICIPATING IN DISCUSSIONS REGARDING, OR PLANNING RELATED TO VECTOR-VALUED VARIABLES**
    - `AXES_POINTS_AND_FRAMES.md`: Conventions and definitions for axis systems, points, and reference points: **READ BEFORE CONTRIBUTING ANY CODE, PARTICIPATING IN DISCUSSIONS REGARDING, OR PLANNING RELATED TO VECTOR-VALUED VARIABLES**
    - `CLASSES_AND_IMMUTABILITY.md`: Description of class structure and attribute immutability.
    - `CODE_STYLE.md`: Code style guidelines: **READ BEFORE CONTRIBUTING ANY CODE**
    - `MUJOCO_CONVENTIONS.md`: Definitive interpretation of MuJoCo state variables and their mapping to Ptera Software's axes, points, frames, and transformations
    - `RUNNING_TESTS_AND_TYPE_CHECKS.md`: Instructions for running tests and type checks **READ BEFORE RUNNING TESTS OR TYPE CHECKS LOCALLY**
    - `STRONG_COUPLING.md`: Mathematical framework for the strongly coupled free-flight UVLM-MuJoCo solver: the fixed-point sub-iteration, Aitken relaxation, the weighting matrix, and the convergence tolerances

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [camUrban/PteraSoftware](https://github.com/camUrban/PteraSoftware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
