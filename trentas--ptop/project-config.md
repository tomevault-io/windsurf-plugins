---
trigger: always_on
description: Interactive TUI for deep inspection of processes.
---

# ptop — implementation guide

Interactive TUI for deep inspection of processes.
Linux is the rich target (eBPF + /proc); macOS is a Tier 1 port via
libproc + Mach with a reduced feature set (see the `*_darwin.go` files
under `pkg/collector/` and issue #22).

This file documents the implementation: tech stack, project layout, type
contracts, and the conventions every collector and view follows.

If something here drifts from reality, the code wins. Update this file.

---

## Stack

| Layer  | Technology | Reason |
|--------|-----------|--------|
| TUI    | [Bubbletea](https://github.com/charmbracelet/bubbletea) + [Lipgloss](https://github.com/charmbracelet/lipgloss) | Mature, composable, mouse support |
| eBPF   | [cilium/ebpf](https://github.com/cilium/ebpf) | Pure-Go, no libbpf.so needed at runtime |
| Build  | Go 1.25+, clang, libbpf-dev (build only) | Single static binary on Linux (`CGO_ENABLED=0`) |
| eBPF C | clang `-target bpf` → `.bpf.o` → `go:embed` | See `Makefile` |
| macOS  | libproc + Mach via cgo (darwin-only build tag) | The only public path for per-process info on macOS |

> Don't introduce a CLI framework — `flag` is sufficient.
> Don't add a logging library — `fmt.Fprintln(os.Stderr, ...)` is enough.
> CGo is gated to `//go:build darwin` for libproc/Mach. The Linux binary
> stays `CGO_ENABLED=0` and statically linked; do not pull cgo into any
> file that compiles on linux.

---

## Visual reference

`assets/mockup.jsx` contains the React prototype with all tabs implemented and
simulated data. **Each Go view must faithfully reproduce the layout of the
corresponding mockup.** Use it as the authoritative visual spec — if there's
any doubt about layout, the mockup wins.

`assets/screenshot-overview.txt` is a captured F1 dump used as a regression
fixture in `internal/tui/dump_test.go`.

Color palette (defined in `internal/tui/styles.go`):

```
bg:      #0e1014    bgPanel: #13161c    border:  #2a2d35
dim:     #3a3d45    muted:   #5a5f72    text:    #c8ccd8
bright:  #e8ecf5    green:   #4ade80    cyan:    #22d3ee
amber:   #fbbf24    red:     #f87171    blue:    #60a5fa
purple:  #a78bfa    pink:    #f472b6    orange:  #fb923c
teal:    #2dd4bf
```

---

## Project structure

```
ptop/
├── CLAUDE.md, README.md, CONTRIBUTING.md, SECURITY.md, LICENSE
├── go.mod, go.sum
├── Makefile, .goreleaser.yaml
├── buf.yaml, buf.gen.yaml         protobuf codegen config (`make proto`)
├── proto/                         event stream schema (pkg ptop.v1)
│   ├── event.proto                unified Event + payloads
│   └── service.proto              EventStream gRPC service
├── cmd/ptop/main.go               entrypoint: parse flags, start model
├── cmd/ebpfselftest/              root-only eBPF self-diagnostic
├── internal/
│   ├── bpf/                       eBPF programs + loader (build tag `ebpf`)
│   │   ├── programs/              .bpf.c sources, compiled by `make gen`
│   │   │   ├── target.bpf.h       shared pid-namespace target filter
│   │   │   ├── syscalls.bpf.c     raw_syscalls/sys_{enter,exit}
│   │   │   ├── cpu.bpf.c          perf_event @ 100Hz/CPU
│   │   │   ├── io.bpf.c           VFS read/write/fsync
│   │   │   ├── network.bpf.c      sock tracepoints + tcp kprobes
│   │   │   ├── threads.bpf.c      sched_switch
│   │   │   ├── memory.bpf.c       mmap/brk/page-fault
│   │   │   ├── heap.bpf.c         libc malloc/free uprobes → lifetime + leak
│   │   │   ├── futex.bpf.c        futex wait/wake → lock graph
│   │   │   ├── signal.bpf.c       signal_generate → signals with origin (#58)
│   │   │   ├── tls.bpf.c          libssl SSL_write/read uprobes → plaintext (#55)
│   │   │   ├── proc.bpf.c         sched fork/exec/exit → exec lineage subtree (#60)
│   │   │   └── security.bpf.c     PROT_EXEC mmap/mprotect + SELinux AVC (#59)
│   │   ├── available.go           runtime feature flag (build-tag based)
│   │   ├── target.go              pid-namespace target resolver (shared)
│   │   ├── caps.go                CAP_BPF / CAP_PERFMON detection
│   │   ├── caps_stub.go           non-Linux stub
│   │   ├── caps_test.go
│   │   ├── cpu.go                 perf_event tracer
│   │   ├── syscalls.go            raw_syscalls tracepoint loader
│   │   ├── network.go             sock tracepoints + connection seeding
│   │   ├── io.go                  VFS syscall tracker loader
│   │   ├── memory.go              memory counter loader
│   │   ├── heap.go                libc allocator uprobe loader (#53)
│   │   ├── tls.go                 libssl uprobe loader → TLS plaintext (#55)
│   │   ├── threads.go             sched_switch loader
│   │   ├── futex.go               futex wait/wake loader
│   │   ├── signal.go              signal_generate loader (#58)
│   │   ├── proc.go                sched fork/exec/exit loader → exec lineage (#60)
│   │   ├── security.go            PROT_EXEC + SELinux AVC loader + stack (#59)
│   │   └── *_stub.go              stubs for non-Linux / no-ebpf builds
│   ├── serve/                     headless gRPC server (ptop --serve)
│   │   ├── serve.go               addr parse + privilege boundary + Run
│   │   ├── hub.go                 fan-in collectors → fan-out to sinks
│   │   ├── sink.go                Sink iface: gRPC subscriber + JSONL writer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trentas/ptop](https://github.com/trentas/ptop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
