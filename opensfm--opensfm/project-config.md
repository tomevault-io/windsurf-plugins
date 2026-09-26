---
trigger: always_on
description: OpenSfM is a Structure-from-Motion library with a hybrid architecture:
---

# OpenSfM AI Coding Instructions

## 1. Architecture & "Big Picture"
OpenSfM is a Structure-from-Motion library with a hybrid architecture:
- **Core Logic (C++)**: Heavy computational tasks (geometry, bundle adjustment, features) are implemented in C++ under `opensfm/src/`. These are exposed to Python as compiled modules (e.g., `pygeometry`, `pymap`, `pfm`) using **PyBind11**.
- **Pipeline Layout (Python)**: The application logic, pipeline orchestration, and user interface are in Python (`opensfm/` package).
- **Data Abstraction**: The `DataSet` class (`opensfm/dataset.py`) is the central definition for filesystem interactions. All pipeline stages read/write to a hardcoded directory structure (`images/`, `config.yaml`, `reconstruction.json`) managed by this class.
- **State Management**: The `Reconstruction` class (`opensfm/types.py`) wraps the C++ `pymap.Map` object. This pattern (Python class holding a C++ handle) is pervasive.

## 2. Key Developer Workflows

### Building
The project uses `scikit-build-core` to compile C++ extensions.
- **Full Build**: `pip install -e .[test]` (Builds C++ extensions and tests, and installs the package in editable mode).
- **Rebuild C++**: Re-running `pip install -e .` is usually required after changing `.cc` files.
- **Dependencies**: Managed via `pyproject.toml` for managing Python dependencies. Outer envinvironment (C++ toolchain, libraries) is set up through `conda` and the `conda.yml` file. Any change to `conda.yml` requires recreating the conda environment with `conda env create --file conda.yml --yes`, then activating it with `conda activate opensfm`. Note that before running the build commands, the conda environment must be activated once.

### Testing
- **Framework**: 
    - Python : `pytest` is used, and tests are in files `test_YYY.py` under `opensfm/test`.
    - C++ : `gtest` is used. For a given library XXX, tests are put under `opensfm/src/XXX/test/` as `YYY_test.cc`.
- **Style**: we favor many tests with few asserts and ideally one function to be tested. Only a few (1-2) larger integration tests per file. We also strive to test using toy examples to check for function correctness and not just check return semantics.
- **Location**: `opensfm/test/` for Python tests. C++ tests are in `./cmake_build`.
- **Synthetic Data**: Python tests heavily rely on synthetic scenes generated in `opensfm/test/conftest.py`. Look for fixtures like `scene_synthetic` or `scene_synthetic_cube`.
- **Run Tests**: `pytest opensfm/test/` for Python tests. C++ tests can be run via `ctest` with `cd cmake_build && ctest --output-on-failure && cd ..`. Don't forget to activate the conda environment before running tests, and run `export LD_PRELOAD=$CONDA_PREFIX/lib/libtcmalloc.so` before running.

### Running the Pipeline
- **Entry Point**: `bin/opensfm` (shell script) -> `bin/opensfm_main.py`.
- **Commands**: Each CLI command (e.g., `detect_features`, `reconstruct`) is implemented as a module in `opensfm/commands/`.
- **Example**: `bin/opensfm reconstruct path/to/dataset` invokes the `run()` method in `opensfm/commands/reconstruct.py`.

## 3. Coding Conventions & Patterns

### Python/C++ Interop
- **Wrappers**: Do not use C++ objects directly if a Python wrapper exists (e.g., use `opensfm.types.Reconstruction` instead of `opensfm.pymap.Map` where possible). 
- **Extensions**: C++ extensions are imported from the root package (e.g., `from opensfm import pygeometry`).


### Code Philosophy
- **Readability**: Prioritize clear, simple and maintainable code. Avoid complicated constructs : you're an seasoned engineer that knows about real-world trade-offs, not a rookie trying to expose its design patterns knowledge. You'll use frameworks and design pattern when they're strictly justified at the present moment, not for an hypothetical future. Use descriptive variable and function names, and break down complex functions into smaller, focused ones.
- **Documentation**: Document all public functions and classes with docstrings. For complex algorithms, include comments explaining the rationale and key steps.
- **Testing**: Write tests for new features and bug fixes. Use the existing test suite as a reference for style and structure.
- **C++ Style**: Follow the existing C++ style in the codebase. Use `clang-format` for consistent formatting. Key points:
    * Stick to use STL and Eigen for data structures. Avoid using OpenCV datastructures in C++, except for image resizing.
    * Use SoA structuring for performance critical data like point clouds.
    * Strive to minimize heap allocation and fragmentation by using stack-allocated buffer. In case of heap-allocated, re-use buffers, and avoid tiny buffers in long for loops.
- **Design**: For the data, try to re-use core lib's data structures. In addition, it is of upmost importance to be able to manipulate large reconstructions efficiently. We strive to handle large amounts of data, so minimize copy operations, use Data-Oriented Design principles, and leverage GPU acceleration where possible (e.g., for map rendering).

### Type Hinting
- **Strictness**: The codebase uses `pyre-strict`. Ensure all new code has complete type annotations. A comment `# pyre-strict` is often present at the top of files.

### Configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenSfM/OpenSfM](https://github.com/OpenSfM/OpenSfM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
