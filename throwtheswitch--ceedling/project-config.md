---
trigger: always_on
description: **How to get things done and understand what's happening during builds**
---

# Conventions & Behaviors

**How to get things done and understand what's happening during builds**

## Directory structure & filenames

Much of Ceedling's functionality is driven by collecting files
matching certain patterns inside the paths it's configured
to search. See the documentation for the `:extension` section
of your configuration file (found in the [configuration reference](../configuration/index.md)) to
configure the file extensions Ceedling uses to match and collect
files. Test file naming is covered later in this section.

Test files and source files must be segregated by directories.
Any directory structure will do. Tests can be held in subdirectories
within source directories, or tests and source directories
can be wholly separated at the top of your project's directory
tree.

## Test build search paths

Test builds in C are fairly complex. Each test file becomes a test
executable. Each test executable needs generated runner code and 
optionally generated mocks. Slicing and dicing what files are 
compiled and linked and how search paths are assembled is tricky
business. That's why Ceedling exists in the first place. Because
of these issues, search paths, in particular, require quite a bit
of special handling.

Unless your project is relying exclusively on `extern` statements and
uses no mocks for testing, Ceedling _**must**_ be told where to find 
header files. Without search path knowledge, mocks cannot be generated, 
and test file compilation will fail for lack of symbol definitions
and function declarations.

Ceedling provides two mechanisms for configuring search paths:

1. The [`:paths` ↳ `:include`](../configuration/reference/paths.md) section within your 
   project file (or mixin files).
1. The [`TEST_INCLUDE_PATH(...)`](build-directives.md#test_include_path) build directive 
   macro. This is only available within test files.

In testing contexts, you have three options for assembling the core of 
the search path list used by Ceedling for test builds:

1. List all search paths within the `:paths` ↳ `:include` subsection 
   of your project file. This is the simplest and most common approach.
1. Create the search paths for each test file using calls to the 
  `TEST_INCLUDE_PATH(...)` build directive macro within each test file.
1. Blending the preceding options. In this approach the subsection
   within your project file acts as a common, base list of search 
   paths while the build directive macro allows the list to be 
   expanded upon for each test file. This method is especially helpful 
   for large and/or complex projects in trimming down 
   problematically long compiler command lines.

As for the complete search path list for test builds created by Ceedling,
it is assembled from a variety of sources. In order:

1. Mock generation build path (if mocking is enabled)
1. Paths provided via `TEST_INCLUDE_PATH(...)` build directive macro
1. Any paths within `:paths` ↳ `:test` list containing header files
1. `:paths` ↳ `:support` list from your project configuration
1. `:paths` ↳ `:include` list from your project configuration
1. `:paths` ↳ `:libraries` list from your project configuration
1. Internal path for Unity's unit test framework C code
1. Internal paths for CMock's C code (if respective feature enabled)
1. `:paths` ↳ `:test_toolchain_include` list from your project 
   configuration

The paths lists above are documented in detail in the discussion of 
project configuration.

_**Notes:**_

* The order of your `:paths` entries directly translates to the ordering
  of search paths.
* The logic of the ordering above is essentially that:
    * Everything above (5) should have precedence to allow test-specific 
     symbols, function signatures, etc. to be found before that of your 
     source code under test. This is the necessary pattern for effective 
     testing and test builds.
    * Everything below (5) is supporting symbols and function signatures
     for your source code. Your source code should be processed before
     these for effective builds generally.
* (3) is a balancing act. It is entirely possible that test developers
  will choose to create common files of symbols and supporting code 
  necessary for unit tests and choose to organize it alongside their 
  test files. A test build must be able to find these references. At the
  same time it is highly unlikely every test directory path in a project
  is necessary for a test build — particularly in large and sophisticated
  projects. To reduce overall search path length and problematic command
  lines, this convention tailors the search path. This is low risk
  tailoring but could cause gotchas in edge cases or when Ceedling is 
  combined with other tools. Any other such tailoring is avoided as it
  could too easily cause maddening build problems.
* Remember that the ordering of search paths is impacted by the merge 
  order of any Mixins. Paths specified with Mixins will be added to 
  path lists in your project configuration in the order of merging.

## Release build search paths

Unlike test builds, release builds are relatively straightforward. Each
source file is compiled into an object file. All object files are linked.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ThrowTheSwitch/Ceedling](https://github.com/ThrowTheSwitch/Ceedling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
