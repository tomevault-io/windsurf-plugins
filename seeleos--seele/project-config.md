---
trigger: always_on
description: - `agent-tools/run-agent-vm.sh`: headless VM run with serial output; use this for runtime verification.
---

# Repository Guidelines

## Build, Test, and Development Commands

- `agent-tools/run-agent-vm.sh`: headless VM run with serial output; use this for runtime verification.
- `cargo fmt --all`: format Rust code before submitting changes.
- `rootfs_making/make_disk.sh`: build or refresh `disk.img` and the guest root filesystem contents.
- If a required tool is missing for this repository workflow, add it to the `flake.nix` dev shell instead of treating it as a one-off host prerequisite.
- When launching the VM during agent work, use a checked-in `.sh` wrapper script instead of invoking the VM command directly. Put any needed log redirection inside the wrapper rather than on the outer command line.
- When using the checked-in VM wrapper, run it directly (for example `agent-tools/run-agent-vm.sh`). Do not wrap it with `bash`, and do not override its default log file path unless explicitly requested.
- `agent-tools/run-agent-vm.sh` should not impose a default timeout. If a timeout is needed for a specific debugging pass, set it explicitly and shut the VM down yourself when finished.
- When polling a background VM terminal, prefer short polling intervals and frequent checks instead of waiting a long time in one shot.
- After finishing VM-based testing, shut the VM down and verify there is no leftover background runner or QEMU process before moving on.
- Do not assume `sysroot/` is mounted or synchronized with `disk.img`. Verify whether it is mounted before using it for runtime inspection, and prefer guest logs captured through the VM wrapper when in doubt.
- If you need `sysroot/` mounted, run `agent-tools/ensure-sysroot-mounted.sh` as a separate step first. Do not chain the mount step together with the real inspection command.
- After `agent-tools/ensure-sysroot-mounted.sh`, if you only need to read files from `sysroot/`, read them directly without `sudo` or a fresh privilege escalation unless it is actually necessary.
- If the sandbox, `no_new_privileges`, missing mounts, or network restrictions block a necessary command, ask the user for privilege escalation or the required access instead of silently giving up on that path.

After finishing a change, run `agent-tools/run-agent-vm.sh` to test the VM. If the VM test fails, keep fixing the issue before considering the work done. If you are validating a shell or userspace fix, prefer the `--agent` path so serial logs are captured automatically.

## Coding Style & Naming Conventions

- Prefer `enum` and `bitflags` over integer `const` groups when values are a closed set.
- When a Linux flag set is already modeled as a `bitflags` type, do not duplicate the same bits as separate local `const`s. Reuse the `bitflags` type directly and prefer Linux ABI names such as `MS_*`, `O_*`, or `MAP_*` on the flags themselves.
- Match Linux naming where the kernel exposes Linux ABI behavior.
- Do not write fully qualified type paths inline such as `alloc::string::String`. If a common type is used, import it at the top of the file and use the short name in code.
- When a handle or ID type needs behavior, prefer a dedicated newtype with inherent methods over a `type` alias plus scattered free helper functions.
- Do not accumulate large amounts of unrelated code in one file. Split code by subsystem or feature when a file starts carrying multiple responsibilities, for example moving select-like syscalls into their own `select.rs`.
- When a file grows to cover multiple distinct responsibilities, split it by behavior instead of keeping one catch-all module. Prefer small neighboring modules such as `state.rs`, `events.rs`, `ioctl_display.rs`, or `ioctl_buffer.rs` over monoliths or generic `abi.rs` buckets. DRM-style ABI constants and structs should live next to the subsystem they serve, not in one shared dump file. File size should preferably stay under 200 lines, but it is acceptable to exceed that when the alternative would make the structure worse.
- When there is a clearly better structural solution, prefer it over local patching. In particular, favor changes that remove repetitive boilerplate, unify error handling, and let call sites use direct propagation such as `?` instead of open-coded checks.
- When an existing library or crate feature can cleanly replace handwritten repetitive decoding or boilerplate, prefer using it over custom open-coded conversion logic.
- Do not take shortcuts just to get something running quickly. In particular, avoid adding stubs, temporary shortcuts, or ad-hoc special cases merely to make a feature appear to work.
- If a debug-only stub is temporarily unavoidable, mark it explicitly with `todo!()` or `unimplemented!()`. If it cannot use either, add a clear `TODO` comment stating that it is a temporary debug stub and not a real implementation.
- For syscall handlers, do not take a user pointer as `u64` and then immediately cast it to `*const T` or `*mut T` in the body. Make the syscall argument itself a properly typed pointer and add or reuse the `SyscallArg` conversion in `kernel/src/systemcall/arg_types.rs`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeeleOS/seele](https://github.com/SeeleOS/seele) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
