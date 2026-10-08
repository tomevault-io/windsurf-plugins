---
trigger: always_on
description: A cross-platform audio coding platform combining three integrated C++23 sub-projects:
---

# TZPL -- Audio Coding Platform

A cross-platform audio coding platform combining three integrated C++23 sub-projects:

- **`lang/`** -- Tzopilotl, a statically typed, real-time safe interpreted language with a direct-threaded VM, TLSF allocator, and incremental GC.
- **`synthdef-compiler/`** -- Compiles audio signal flow graphs (from S-expressions or C++ DSL) into optimized `.dylib` plugins via a 14-pass analysis pipeline.
- **`engine/`** -- Real-time audio engine with dynamic graph patching, lock-free threading, multi-silo parallelism, crossfading, and sample-accurate scheduling.
- **`bridge/`** -- FFI bridges connecting Tzopilotl to the engine and synthdef-compiler, plus optional OSC and NATS support.
- **`app/`** -- GUI application integrating all components: multi-tab code editor, output panel, REPL, notebooks. The JUCE version (`app/juce/`, target `tzpl_app_juce`) is preferred; the original Dear ImGui version (`tzpl_app`) is still available.
- **`shared/`** -- Common headers: plugin ABI (`tzpl_plugin_abi.h`), SIMD, RNG, matrix ops.

## Building

Build from the project root (NOT from sub-project directories):

    ./build.sh

    cmake -B build -DCMAKE_BUILD_TYPE=Release -DTZPL_BUILD_APP=ON
    cmake --build build -j$(nproc 2>/dev/null || sysctl -n hw.ncpu)

AddressSanitizer build:

    cmake -B build-asan -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS="-fsanitize=address"
    cmake --build build-asan -j$(nproc 2>/dev/null || sysctl -n hw.ncpu)

The build directory is `build/` at the project root. Key targets: `tzpl`, `tzpl_lib`, `engine`, `audio_engine_lib`, `synthdef-compiler`, `synthdef_compiler_lib`, `tzpl_app`.

**Requirements**: Clang 19+ (Clang required: `ext_vector_type` swizzles and `[[clang::musttail]]`), CMake 3.21+, C++23. macOS (CoreAudio) or Linux (ALSA/JACK/Pulse; see `docs/LINUX.md`).

## Testing

    cd lang/tests && bash run_tests.sh

Flags: `-v` (verbose), `-f "pattern"` (filter), `-u` (update golden files), `-x` (stop on failure).

Synthdef compiler tests:

    ./build/synthdef-compiler/synthdef-compiler --test

Always use the official test runner for lang tests -- do not use ad-hoc test loops.

## Tzopilotl Language Quick Reference

Syntax essentials for reading and writing `.x` files:

    -- Line comment
    /* Block comment (nestable) */

    -- Variable declarations
    let x = 42;                    -- immutable
    var y = 0;                     -- mutable local

    -- Functions: no colon between name and type, no -> before return type
    fn add(a Int, b Int) Int { a + b }        -- trailing expr: no semicolon
    fn add2(a Int, b Int) Int = a + b;        -- expression-body form
    fn greet(name String) Void { print(name); }

    -- Statements end with semicolons
    -- A function's return value is its trailing expression, which has NO
    -- semicolon; `{ a + b; }` is a statement, so the body returns Void
    -- Semicolons after struct/enum closing braces are optional

    -- Types: Bool, Int, Float, Symbol, String, Fraction, Complex,
    --        [T], List<T>, Range<T>, Set<T>, Map<K,V>,
    --        Tuple(T1, T2, ...), Struct, Enum, Ref<T>, Function, Coroutine<T>

    -- Mutable containers (Array, Map, Set): heap objects passed by reference.
    -- Mutating builtins idiomatically end in `!`; non-mutating ones return new
    -- values. `!` is part of the identifier -- `push` and `push!` are distinct.
    var a = [1, 2, 3];
    a[0] = 100;                    -- indexed assignment (cyclic, like reads)
    a push!(4);                    -- mutating builtin
    let b = a copy;                -- shallow copy for independent alias

    var m = ["x": 1];
    m["y"] = 2;                    -- insert-or-update

    var s = Set(1, 2);
    s insert!(3);
    let popped = s pop!;           -- mutating; returns one element
    a clear!;                      -- empty any Array/Map/Set in place
    a append!(b);                  -- bulk-append an array/list in place ($ analogue)
    a isEmpty;                     -- true when any collection has no elements

    -- Auto-mapping: functions automatically apply over arrays/lists
    let xs = [1, 2, 3];
    let ys = xs add(10);           -- [11, 12, 13]

    -- @ operator for explicit mapping and Cartesian products
    [[1,2], [3,4]] @ reverse;      -- [[2,1], [4,3]]

    -- Pattern matching
    match value {
        case .Some(x) => x;
        case .None => 0;
    }

    -- Modules
    import math.*;
    import utils.foo, utils.bar;

    -- Structs and enums
    struct Point { x Float; y Float; }
    enum Option<T> { case Some(T); case None; }

    -- Lambdas
    let f = fn(x Int) Int { x * 2 };

    -- Template functions
    fn identity<T>(x T) T { x }

## C++ Coding Conventions

- **C++23** standard throughout.
- **East const**: `Record const&` not `const Record&`.
- **Formatting**: Prefer `std::print`/`std::format` over iostream/string concatenation.
- **Namespaces**: `lang` (interpreter), `engine` (audio engine), `synthdef` (compiler), `sexpr` (S-expression parser).
- **File naming**: Sub-project prefixes (`tzpl_` for engine, `synthdef_` for compiler). Snake_case filenames.
- **Memory**: TLSF allocator in lang VM (never call system allocator during execution). Arena allocator in synthdef-compiler.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lfnoise/tzpl](https://github.com/lfnoise/tzpl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
