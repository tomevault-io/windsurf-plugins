---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build System

This project uses [go-task](https://taskfile.dev) (`Taskfile.yml`) as the build orchestrator.

```sh
task build          # fmt + compile (CGO_ENABLED=0, PGO, stripped, version vars injected)
task build-debug    # fmt + compile with race detector (CGO_ENABLED=1)
task lint           # fmt + golangci-lint v2 (timeout 5m)
task lint-nil       # fmt + nilaway
task fmt            # gci + gofumpt + betteralign (Go sources)
task fmt-bpf        # clang-format on bpf/*.c / bpf/*.h (requires clang-format v15+)
task generate       # go generate (recompile eBPF C → Go; requires clang)
task modernize      # gopls modernize -fix -test ./...
task update         # go get -u && go mod tidy
task release        # goreleaser release --clean -p 4
```

Plain `go build` also works for the Go userland (pre-compiled eBPF objects are committed). The default `task` runs `update` then `build`.

## Testing

```sh
GOTMPDIR=/root/tmp go test ./...                       # run all Go unit tests
GOTMPDIR=/root/tmp go test -run TestOSPFTypeName ./... # run a single test
```

Tests live in three files:
- `helpers_test.go` — protocol decoder helpers (`ospfTypeName`, `arpOpName`, `greInnerName`).
- `output_test.go` — `outputPlain` / `MarshalJSON` formatting for protocols that pack metadata into port fields (ESP/AH SPI, ARP opcode, OSPF type/version, GRE inner protocol/flags). Has an `init()` that pre-points `useKProbes` and `useCGroup` to false/empty so `outputPlain` doesn't deref nil flag pointers.
- `sniff_test.go` — `sniffAppProtoGo`, a byte-for-byte Go mirror of the BPF `sniff_app_proto` C function. `TestSniffAppProtoPositives` / `TestSniffAppProtoNegatives` validate L7 signature matching. Any change to the BPF sniffer must be reflected here.

The eBPF C code has no Go-side tests — it is validated only by the kernel verifier at load time.

## Architecture Overview

pktstat-bpf is a Linux eBPF packet statistics tool with two layers:

**eBPF layer (C, in `bpf/`):**
- `bpf/counter.h` — protocol constants: `ETH_P_*`, `IPPROTO_*`, `PROTO_ARP_FAKE` (254), `PROTO_TCP_RETX` (253), `APP_PROTO_*` L7 identifiers (0–9), `L7_PEEK_LEN` (12), `MAX_ENTRIES` (131072). **The `APP_PROTO_*` values must stay in lockstep with the `appProto*` Go constants in `helpers.go`.**
- `bpf/counter_common.h` — shared types and helpers used by all counter programs: `statkey` / `statvalue` / `flowkey` structs; the `pkt_count` map (`BPF_MAP_TYPE_LRU_PERCPU_HASH`) and `flow_app_proto` map (`BPF_MAP_TYPE_LRU_HASH`, 5-tuple → `uint8` app-proto); IPv4-mapped-IPv6 encoding; ARP/ESP/AH/GRE/OSPF parsers; `sniff_app_proto` (pure-logic L7 byte-pattern matcher); `detect_and_cache_l7` (TC/XDP direct packet access path); `detect_and_cache_l7_skb` / `extract_tcp_flowkey_skb` / `sniff_tcp_skb` (KProbe skb path); runtime-patchable globals (`cgrpfs_magic`, `arp_enabled`).
- `bpf/tc.bpf.c` — TC (TCX ingress+egress) counter program.
- `bpf/xdp.bpf.c` — XDP (ingress only) counter program.
- `bpf/kprobe.bpf.c` — per-process TCP/UDP/ICMP/IP-layer KProbes with PID + CGroup tracking; uses `process_l4_skb` and `sniff_tcp_skb` from `counter_common.h`.
- `bpf/cgroup_skb.bpf.c` — CGroup SKB ingress/egress + `inet_sock_create` / `inet_sock_release` for PID attribution.
- `bpf/cgroup.bpf.c` — raw tracepoint on `cgroup_mkdir` that pushes new cgroup path events to userspace via a perf buffer; shared by KProbes and CGroup modes.

**Userland layer (Go, root package `main`):**
- `gen.go` — `//go:generate` directives invoke `bpf2go` for each `.bpf.c` × {amd64, arm64}, producing `tc_`, `xdp_`, `kprobe_`, `cgroupSkb`, and `cgroup` prefixed `*_{x86,arm64}_bpfel.{go,o}` files. Note: `bpf2go -target amd64` writes files with the `_x86_` suffix.
- `main.go` — entry point; selects capture mode, loads only the eBPF object(s) needed for that mode, wires up links, runs TUI or CLI loop. `loadAndPatchSpec` applies `applyCgrpfsMagic`, `applyMaxEntries`, `applyArpEnabled` uniformly.
- `probe.go` — `startTC`, `startXDP`, `startKProbes`, `startCgroup`, `startCGroupTrace`: attach eBPF programs and gate on `features.HaveProgramType`.
- `map.go` — reads `pkt_count` (prefers `BatchLookup` ≥5.6, falls back to iterator) and `flow_app_proto` (`readFlowAppProto` dispatches to `readFlowAppProtoBatch` / `readFlowAppProtoIter`). `statkeyToFlowkey` builds the 5-tuple `tcFlowkey` from a `tcStatkey` (remapping proto 253→6 for retransmits). `addStats` performs a primary lookup then a reverse-direction fallback for TCP rows when the flow was cached from the opposite kprobe direction.
- `cgroup.go` — `cgroup-id → path` cache populated by walking `/sys/fs/cgroup` and consuming `cgroup_mkdir` perf events.
- `output.go` — `processMap`, sort functions, `outputPlain`/`outputJSON`, bitrate formatting, and the `MarshalJSON` override that decodes protocol-packed port fields into named JSON keys.
- `tui.go` — rivo/tview TUI refreshed on `--refresh` interval. Column `l7` (index 4) shows the `AppProto` field.
- `flags.go` — flag parsing via `peterbourgon/ff/v4`; exposes package-level pointers (`useXDP`, `useKProbes`, `useCGroup`, `maxEntries`, `noARP`, …).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dkorunic/pktstat-bpf](https://github.com/dkorunic/pktstat-bpf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
