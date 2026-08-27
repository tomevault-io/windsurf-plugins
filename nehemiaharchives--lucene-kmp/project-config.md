---
trigger: always_on
description: I'm porting apache lucene from java to platform agnostic kotlin common code.
---

## Project Overview
I'm porting apache lucene from java to platform agnostic kotlin common code.
This project is a sub directory under the the root directory of the porting project and the directory name is lp.
Under this directory you find two sub directories:

1. lucene sub directory
   This is the source code of java lucene. the commit id is fixed to ec75fcad5a4208c7b9e35e870229d9b703cda8f3 until all java classes/unit tests ported. After all ported, the project will proceed to the next phase to port commit by commit from this commit id to catch up with the latest lucene code. The code will be ported from this read-only repository.

2. lucene-kmp (kmp stands for kotlin multiplatform) sub directory, which is THIS project

## Porting Guideline

- The GitHub copilot never make any change to lucene java source code but only read, copy from, analyze and answer question based on its content. If it's in agent mode, it can use git commands which does not change any code but only read the code and history.

- **Do not deviate from Java Lucene logic.** The Kotlin port must be an exact behavioral port. Only deviate when unavoidable for KMP (e.g., okio for IO, coroutines for threading). Any deviation must be explicitly justified and minimized.
- Exception for development speed: Java-Kotlin numeric-value discrepancies are allowed only when reducing test/runtime iteration counts to speed up local iteration or CI.
- Speed-up reductions must be order-of-magnitude changes, not tiny tweaks:
  - Target example: if a test takes ~10 minutes, reduce to ~3 seconds when possible.
  - Numeric example: if iteration/repeat is `1000`, reduce to `10`; if still >30 seconds, reduce to `3`.
  - Counter example: do not treat small edits like `19 -> 15` as sufficient speed-up by default.
- For every such discrepancy, add an inline comment immediately after the exact reduced line (not above it).
- Required comment format:
  - Starts with `// TODO`
  - States the reduction values explicitly (example: `reduced valueA = x1 to x2, valueB = y1 to y2`)
  - Ends with `for dev speed`
  - Example: `// TODO reduced valueA = 1025 to 5, valueB = 500 to 3 for dev speed`

- When porting, class name, interface name, method name, variable name should be the same as much as possible especially for APIs which is used by other classes.

- root package name of java lucene is org.apache.lucene. The root package name of the kotlin common code is org.gnit.lucene-kmp. The sub package structure under the root package should be the same as much as possible.

- When porting, when the certain java class included in JDK (e.g. java.util.List, java.util.Map, java.lang.String, etc) is used in lucene java code, you should use kotlin common code equivalent class (e.g. kotlin.collections.List, kotlin.collections.Map, kotlin.String, etc) instead of the JDK class. If those equivalent class is not found in kotlin common code of kotlin standard library, this project will copy the source code of the JDK class which is missing in kotlin std lib and port it into kotlin common code in a package called org.gnit.lucene-kmp.jdkport. These jdk ported classes/interface need to have annotation called @Ported with argument called from like this: @Ported(from="java.util.List") So when porting, if you encounter compilation error saying unresolved reference to certain JDK class/interface, you should first look into the package org.gnit.lucene-kmp.jdkport to see if the ported class/interface is already there. Only when if not found, it should be ported form JDK source code. Most of the missing JDK classes are already in the package.

- Do not port `getXxx()` `setXxx()` method from java when there is `val xxx` or `var xxx` in kotlin common. porting them will end up compilation error regarding platform declaration.   

- The ported project is targeting following platforms:
    1. jvm (jvm server, jvm desktop, Android)
    2. native (iOS, linux)

- The linux native target is not for projection but for development in linux desktop environment. The kotlin native uses LLVM as its compiler and gradle kotlin compile job emits almost exact same compilation error for native/linux and native/iOS. So even in the linux box which does not have build toolchain for iOS, you can quickly check if the ported code compiles for iOS or not by compiling for native/linux target.

- The porting project prioritizes kotlin common code over expect/actual mechanism to reduce the amount of platform specific code. So when porting, first try to use kotlin common code only. Only when if not possible, you should use expect/actual mechanism. When using expect/actual mechanism, the platform specific code should be created in following 2 source sets:
    1. jvmAndroidMain, jvmAndroidTest for jvm/android platform
    2. nativeMain, nativeTest for native platform (iOS, linux)

- If the target java class to port into kotlin common is too large for your context size, eg. more than 500 LOC, or more than 50 unit test functions, try porting again baby step strategy. For example, create a class with empty functions with // TODO comments, or when you port unit test class, first create empty test class with no-op tests with //TODO comments as skeleton. Then one by one implement the //TODO of each functions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nehemiaharchives/lucene-kmp](https://github.com/nehemiaharchives/lucene-kmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
