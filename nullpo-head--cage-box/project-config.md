---
trigger: always_on
description: Cage Box is a sandbox tool to enable scripts and binaries to declare their allowed permissions,
---

# AGENTS.md for Cage Box project

## What is Cage box?

Cage Box is a sandbox tool to enable scripts and binaries to declare their allowed permissions,
ensuring they can only interact with allowed file paths. Even when running with
`sudo`, these programs are prevented from accessing any unauthorized files or
directories. Cage Box is a rootless tool; it allows you to restrict target
processes without root privilege.

Cage Box is like an AppArmor, but a standalone tool that can be used with any script or binary, not just those that are designed to work with it. It uses Linux's Landlock to restrict file access, and seccomp_unotify together to dynamically allow or deny file path access in an interactive mode.

With Cage Box,
- a developer can declare file paths their script or binary is allowed to access
- an end user can check what files are accessed by looking at the beginning of the file or `$ Cage Box -l app`, without inspecting the whole contents
- you can also run unverified applications that don't natively support Cage Box in an interactive sandbox that asks you whether each path accessed should be allowed at runtime
- you can also make a quick and secure sandbox for your AI agent to run commands, without worrying about it accessing unauthorized files or directories.

## Sandbox mechanism overview

Cage Box restricts file access by Landlock rules, which is converted from a Cage Box rule file. Since Landlock does not restrict some critical system calls such as truncate or setxattr, Cage Box combines Landlock with seccomp-bpf and seccomp notify; Cage Box spawns a monitor process to restrict such system calls by the target process, and emulates those system calls on behalf of the target process.

These sandbox are implemented mainly by src/monitor.rs

Cage Box also supports an interactive monitoring mode. Cage Box asks users to accept or deny file paths when it detects file accesses which are not allowed by the rules. This is achieved by a novel mechanism; Cage Box denies all file access by an empty Landlock rule first, then emulates file path system calls such as `openat` or `mkdir`. The recent seccomp notify mechanism enables this method, since it enables Cage Box to inject result file descriptors to the target process.

See src/interactive.rs

The novel points of Cage Box are
- Handy Cage Box script UX, which is convenient to distribute binaries and sandbox rules together
- Cage Box allows you to interactively allow or deny file access when you run it in the interactive sandbox mode, which is powerful and handy, but not a common feature among existing sandbox tools.
- It is a fully rootless sandbox solution. Vulnerability of Cage Box does not give more privileges than you originally have.
- It does not use user namespaces either; it runs on Debian and Ubuntu without additional configuration, where user namespaces are restricted

## Implementation design


Cage Box restricts the file access of scripts and binaries with Landlock LSM (Linux Security Module).

The restriction implementation is based on the following three components:

### 1. Landlock LSM:

Cage Box runs the binary to restrict the file access, applying the Landlock LSM rules defined in the Cage Box script.

This is the core mechanism of Cage Box. Cage Box converts the sandbox rules into Landlock rulesets.
Landlock does not require root privilege to apply restriction, which allows Cage Box to be a rootless sandbox solution.

### 2. seccomp_unotify (seccomp-bpf + unotify):

While Landlock is the core mechanism of Cage Box, it has two problems as-is;

* Some of critical syscalls are not restricted by Landlock, including `chmod`, `chown`, `truncate`, `ioctl`, and so on.
   You can find an exhaustive list in `man landlock`. This is a known limitation of Landlock.
   So, we need an additional mechanism to restrict these syscalls.

* Landlock does not allow us to hook file access. Thus, we cannot achieve interactive file path restriction by Landlock.

To solve these two problems, Cage Box utilizes seccomp_unotify.
seccomp_unotify is relatively a new feature of Linux. It is similar to regular seccomp-BPF + ptrace,
but it allows a monitor process to inject a FD to the target process,
by which the monitor process can emulate file related syscalls.

For the first issue, Cage Box uses seccomp-bpf + unotify to hook those syscalls,
check if the requested paths are allowed by applied Cage Box ruleset,
and call syscalls against the given paths on behalf of the target process and injects the result to the target process.

For the second issue, Cage Box first restricts all file access by Landlock.
Then, Cage Box hooks `open` syscall family by seccomp_unotify.
When file accesses happen, Cage Box asks the user interactively whether they want to allow that access or not.
If users allow them, Cage Box emulates `open` syscalls and injects result FDs to the target process.

### 3. regular seccomp-BPF + ptrace (to replace execve with execveat so that we can hook it via notify):

A remaining issue of the approach of Landlock + seccomp_unotify for the interactive mode is a fact that a monitor process cannot emulate execve for the target process.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nullpo-head/cage-box](https://github.com/nullpo-head/cage-box) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
