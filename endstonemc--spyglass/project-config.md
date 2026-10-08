---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

## Project Overview

Spyglass reports what the Minecraft: Bedrock client failed to decode: which packet, how far into
it the decode got, and the reason the client gave. It runs inside the client — a DLL an injector
loads on Windows, an Android x86_64 shared object the Linux launcher loads as a mod — hooks
`Packet::readNoHeader`, and draws its findings in an ImGui overlay.

## Rules

These exist because generated code drifts towards all of them. They are not suggestions.

### No comments

Do not add a comment. Not to explain a change, not to restate the code, not to record why a
decision was made. Name things so the code says it by itself; if a comment still feels necessary,
the code is wrong — restructure or rename until it is not.

- No comment that narrates the development process ("now handles X", "kept for compatibility").
- No comment that repeats what the line below it does.
- No block of prose above a function explaining its design.
- Leave existing comments alone unless a change makes one wrong, in which case delete it.
- Write one only when explicitly asked for it, and then a single terse line.

Rationale for a non-obvious change belongs in the commit message, where it is attached to the diff
rather than to the source forever.

### No single-use helpers

A function called from one or two places is not an abstraction, it is indirection. Inline it. A
helper earns its name by having three or more callers, or by being the public surface of a header.
The same goes for a variable that holds a value used once, a struct that wraps one field, and a
`constexpr` naming a number that appears once.

`detail::fp_cast` is exempt. It is the mechanism that makes a member detour ABI compatible with the
member function it replaces, not indirection, and it stays however few callers it has.

### A detour keeps the signature form it replaces

A detour is declared the way the function it hooks is declared. A member function stays a member
function; it never becomes a free function taking `this` as a first parameter. With a large return
type MSVC passes a member its `this` in `rcx` and the return buffer in `rdx`, and a free function
the other way round, so the swap hands the detour its return buffer where it expects the object.

Nothing about this fails loudly. The forwarding call passes the registers straight through, so the
hooked function still decodes correctly and only the detour misreads its own arguments; the Itanium
ABI passes both alike, so the Android build stays right while Windows dies on the first packet.
Spyglass shipped that bug for a week and it read as a client update.

### One dependency source

`third_party/` holds a directory per dependency, each with a `CMakeLists.txt` that fetches it with
`FetchContent` and defines the target the rest of the build links. No Conan, no `ExternalProject`,
no vendored copies, no submodules, no system packages, no per-platform exception. A dependency that
ships no CMake of its own has its build written in its own directory, never inlined into the top
level.

Every dependency is pinned to a tag or a commit, and both platforms take the same revision. imgui
is pinned to the revision the launcher's own submodule sits on: the two copies share one
`ImGuiContext`, so a different revision is a different layout.

EnTT is the exception, and only because it has to be: cereal's reflection *is* EnTT's, so the pin
has to be whatever the client was compiled against, and that changed at 1.26.50. Its directory
fetches both revisions, exposes one target each, and `entt_for_client` maps a client version to the
right one. Nothing else picks between them, and no other dependency gets to grow a second copy.

### One build description

`CMakeLists.txt` describes all targets. Platform differences are the narrow parts that genuinely
differ — a source list, a system library — not two parallel worlds under an `if`. Prefer explicit
source lists over `file(GLOB)` plus `list(REMOVE_ITEM)`: a glob that is immediately filtered is a
glob that should not have been written.

### Toolchains

| target | toolchain |
| --- | --- |
| Windows DLL and injector | clang-cl, MSVC's Windows SDK and STL |
| Launcher mod | Android NDK (clang, libc++), x86_64 |

There is no Linux target. The Linux launcher runs the Android build of the client under its own
linker, so the payload it loads is an Android shared object built against bionic, not a Linux one.
Both platforms build through `CMakePresets.json`, as `debug-windows` and `debug-android` and the
same for `release` and `relwithdebinfo`, each configuring into `build/<preset>`. The Android presets
take the NDK from `ANDROID_NDK_ROOT` and hide themselves when it is unset. The Windows presets hide
themselves off a Windows host.

### CI covers every platform

Every platform that ships is built in CI. A target nobody builds is a target that is already
broken. Release artifacts are the injector and the DLL for Windows, and the Android mod the Linux
launcher loads.

### Read the client, do not reason about it

Every wrong answer costs an inject cycle; reading costs a minute. The declarations are in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EndstoneMC/spyglass](https://github.com/EndstoneMC/spyglass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
