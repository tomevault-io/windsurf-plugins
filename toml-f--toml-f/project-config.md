---
trigger: always_on
description: This document provides guidance for AI coding assistants (GitHub Copilot, Claude, etc.) working with the TOML-Fortran (toml-f) codebase.
---

# AGENTS.md - Guidelines for AI Coding Assistants

This document provides guidance for AI coding assistants (GitHub Copilot, Claude, etc.) working with the TOML-Fortran (toml-f) codebase.

## Project Overview

TOML-Fortran is a TOML parser implementation for data serialization and deserialization in Fortran. It provides a complete implementation of the [TOML v1.0.0 specification](https://toml.io/en/v1.0.0).

- **Language**: Fortran 2008
- **License**: Apache-2.0 OR MIT (dual-licensed)
- **Repository**: https://github.com/toml-f/toml-f

## Code Structure

```
toml-f/
├── src/                   # Main source code
│   ├── tomlf.f90          # Public API module
│   └── tomlf/             # Implementation modules
│       ├── build/         # Builder utilities (get_value, set_value)
│       ├── de/            # Deserializer (parser, lexer)
│       ├── ser.f90        # Serializer
│       ├── structure/     # Internal generic data structures (list, map, ordered_map, etc.)
│       ├── type/          # Type definitions (TOML types: toml_table, toml_array, toml_keyval)
│       ├── utils/         # Utility functions
│       └── ...
├── test/                  # Test suite
│   ├── unit/              # Unit tests (using test-drive framework)
│   ├── compliance/        # TOML compliance tests (toml2json, json2toml)
│   ├── example-1/         # Example programs
│   └── version/           # Version tests
├── doc/                   # Sphinx documentation source
├── config/                # Build configuration files
└── subprojects/           # Meson subprojects (test-drive)
```

## Build Systems

This project supports three build systems:

### Meson (Preferred)
```bash
meson setup _build
meson compile -C _build
meson test -C _build --print-errorlogs
```

### CMake
```bash
cmake -B _build -G Ninja
cmake --build _build
ctest --test-dir _build
```

### Fortran Package Manager (fpm)
```bash
fpm build
fpm test
```

## Coding Conventions

### File Headers

All source files must include the SPDX license header:

```fortran
! This file is part of toml-f.
! SPDX-Identifier: Apache-2.0 OR MIT
!
! Licensed under either of Apache License, Version 2.0 or MIT license
! at your option; you may not use this file except in compliance with
! the License.
!
! Unless required by applicable law or agreed to in writing, software
! distributed under the License is distributed on an "AS IS" BASIS,
! WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
! See the License for the specific language governing permissions and
! limitations under the License.
```

### Fortran Style

- Use **free-form** Fortran source format (`.f90` extension)
- Use `implicit none` in all program units
- Prefer `intent(in)`, `intent(out)`, or `intent(inout)` for all dummy arguments
- Use descriptive variable and procedure names
- Document public procedures with comment blocks describing parameters
- Use `allocatable` for dynamic data rather than pointers where possible

### Naming Conventions

- Module names: `tomlf_<component>` (e.g., `tomlf_de`, `tomlf_ser`)
- Public API uses `toml_` prefix (e.g., `toml_load`, `toml_dumps`, `toml_table`)
- Type names: `toml_<name>` (e.g., `toml_table`, `toml_array`)
- Internal/private procedures use descriptive names without prefix
- Test modules: `test_<component>` (e.g., `test_lexer`)

### Module Structure

```fortran
module tomlf_example
   use dependency_module, only : specific_imports
   implicit none
   private

   public :: public_type, public_procedure

   ! Type definitions

   ! Interface definitions

contains

   ! Procedure implementations

end module tomlf_example
```

### Documentation

- Use FORD-compatible docstrings (marked with `!>`)
- User documentation is written in reStructuredText (Sphinx)

## Public API

The main public API is exposed through the `tomlf` module (`src/tomlf.f90`):

- **Parsing**: `toml_parse`, `toml_load`, `toml_loads`
- **Serialization**: `toml_serialize`, `toml_dump`, `toml_dumps`
- **Data access**: `get_value`, `set_value`
- **Types**: `toml_table`, `toml_array`, `toml_keyval`, `toml_value`
- **Utilities**: `toml_datetime`, `toml_error`, `toml_stat`

Internal modules under `src/tomlf/` are not part of the public API and may change between versions.

## Testing

Unit tests are located in `test/unit/`, using the [test-drive](https://github.com/fortran-lang/test-drive) framework.

- Each test module should have a `collect_<name>` subroutine that returns a testsuite
- Run tests with: `meson test -C _build` or `fpm test` or `ctest`

Key test files:
- `test/unit/lexer.f90` - Lexer tests
- `test/unit/parser.f90` - Parser tests
- `test/unit/build.f90` - Builder tests
- `test/unit/utils.f90` - Utility tests
- `test/unit/sort.f90` - Sorting tests

### Adding New Tests

1. Create a new test file `test/unit/test_<name>.f90`
2. Add the test to the build system files (`meson.build`, `CMakeLists.txt`)
3. Register the test collection in `test/unit/main.f90`

### Compliance Tests

The project uses the [BurntSushi/toml-test](https://github.com/BurntSushi/toml-test) validator suite for TOML standard compliance testing. The `toml2json` and `json2toml` binaries in `test/compliance/` are used for this.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [toml-f/toml-f](https://github.com/toml-f/toml-f) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
