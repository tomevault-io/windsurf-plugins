---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

bVisor is an in-process Linux sandbox SDK and runtime written in Zig. It intercepts and virtualizes Linux syscalls from userspace using seccomp user notifier, providing isolation without VM overhead. Unlike gVisor (which runs as a separate service), bVisor runs directly in your application for millisecond-level sandbox lifecycle.

The goal of bVisor is to be a lightweight sandbox for untrusted user or LLM-generated code run on the server. Its most minimal implementation creates a virtualized filesystem and runs a bash command inside of it, but the goal is to increase sandboxing over time. This is intended as alternative to docker, gvisor, or other vm-based sandboxes.

**Status**: Early proof-of-concept. Core seccomp interception works; syscall virtualization is incomplete.

**Greenfield project**: No users, no backward compatibility concerns. Delete dead code freely.

## Build Commands

```bash
zig build                    # Build all targets (exe, tests, node .node binaries)
zig build test               # Run unit tests in Docker container
zig build run                # Run executable in Docker container
zig build run-node           # Run E2E node SDK tests with current zig core build
```


**Requires**: Zig 0.16.0-dev or later, Docker


## Architecture

**Supervisor-child process model with syscall interception:**

```
src/
  core/                 # Zig sandbox runtime
    root.zig            # Public exports for SDK usage
    main.zig            # Entry point, runs smoke test
    setup.zig           # Fork into child/supervisor, seccomp BPF installation
    Supervisor.zig      # Main loop: recv notif → handle → send response
    types.zig           # LinuxResult, Logger
    LogBuffer.zig       # Thread-safe buffered output capture (stdout/stderr)
    smoke_test.zig      # TDD-style smoke test scorecard

    seccomp/
      filter.zig        # BPF filter installation, returns notify FD
      notif.zig         # Helper to construct test notifications

    utils/              # Linux utility modules
      memory_bridge.zig # process_vm_readv/writev (test mode: local pointer access)
      pidfd.zig         # pidfd_open/pidfd_getfd (test mode: passthrough)
      proc_info.zig     # TID/TGID info, clone flags via /proc (test mode: mock maps)

    virtual/            # Virtualization layer
      OverlayRoot.zig   # Root overlay filesystem management
      path.zig          # Path routing: prefix-tree rules (block/passthrough/cow/tmp/proc)
      proc/             # Thread/process virtualization
        Threads.zig     # Manages all threads, kernel TID → Thread mapping
        Thread.zig      # Single thread: tid, thread_group, namespace, fd_table, parent/children
        ThreadGroup.zig # Thread group (process): tgid, threads map
        ThreadStatus.zig # Thread status information
        Namespace.zig   # TID namespace with refcounting, NsTid allocation
      fs/               # File descriptor virtualization
        FdTable.zig     # Per-process fd table, refcounted (shared on CLONE_FILES)
        FdEntry.zig     # Entry type in fd table, containing pointer to File and CLOEXEC flag
        File.zig        # Virtual file with Backend union and refcounting
        backend/        # File backend implementations
          passthrough.zig # Kernel FD passthrough (/dev/null, /dev/zero, /dev/random, /dev/urandom)
          cow.zig       # Copy-on-write files (default for most paths)
          tmp.zig       # Overlay-redirected /tmp files
          procfile.zig  # Virtualized /proc files
      syscall/          # Syscall handlers
        syscalls.zig    # Switch statement over syscalls, parsing notifications
        e2e_test.zig    # End-to-end syscall tests
        handlers/       # Individual syscall handlers
          openat.zig    # openat handler with path rules (block/allow/virtualize)
          close.zig     # close handler
          read.zig, write.zig, readv.zig, writev.zig  # I/O handlers
          lseek.zig     # File seek
          dup.zig, dup3.zig  # FD duplication
          fstat.zig, fstatat64.zig  # File metadata
          getpid.zig, getppid.zig, gettid.zig         # ID handlers
          exit.zig, exit_group.zig                    # Exit handlers
          kill.zig, tkill.zig                         # Signal handlers
          uname.zig     # System info (virtualizes hostname/domainname)
          sysinfo.zig   # System stats (virtualizes uptime/procs)
          lseek.zig     # repositioning file offsets
          faccessat.zig # checking user permissions to directory
          fcntl.zig     # file descriptor control
          pipe2.zig     # pipe creation
          getdents64.zig # directory listing
          mkdirat.zig   # directory creation
          unlinkat.zig  # file/directory removal
          execve.zig    # program execution
          socket.zig, socketpair.zig, connect.zig, shutdown.zig  # Socket lifecycle
          ioctl.zig     # device I/O control
          sendto.zig, recvfrom.zig, sendmsg.zig, recvmsg.zig    # Socket I/O

  sdks/
    node/               # Node.js SDK (see src/sdks/node/CLAUDE.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [butter-dot-dev/bVisor](https://github.com/butter-dot-dev/bVisor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
