---
trigger: always_on
description: You are an agent setting this up on a fresh pair of machines. This file is the
---

# AGENTS.md — bring up DeepSeek-V4-Flash on the 2-box gfx1151 vLLM cluster

You are an agent setting this up on a fresh pair of machines. This file is the
runbook: build the pieces, wire them together, get the model serving, and verify
it. Read it top to bottom **before** running anything — several steps are
hard-to-reverse and order matters.

## 0. What you are building

DeepSeek-V4-Flash served by a **patched vLLM**, tensor-parallel across **two AMD
Strix Halo (gfx1151) boxes**, with the inter-GPU all-reduce carried over a
**Thunderbolt-4 / USB4 RoCE-RDMA** link.

```
        ┌────────────── box1 (ray HEAD, gfx1151) ──────────────┐
        │  distrobox "vllm"  ──►  vllm serve  TP rank 0         │
        │  ds4-vllm.service → ds4-cluster-restart.sh            │
        └───────────────┬───────────────────────────────────────┘
                        │  Thunderbolt-4 cable
                        │  RoCE-RDMA dev = usb4_rdma*   (tbv stack)
                        │  IP link thunderbolt0 = 192.168.100.1/.2
        ┌───────────────┴───────────────────────────────────────┐
        │  distrobox "vllm"  ──►  ray worker  TP rank 1          │
        └────────────── box2 (ray WORKER, gfx1151) ─────────────┘
```

Three independent layers, build/verify them in this order:

1. **tbv RDMA** (`tbv/`) — the Thunderbolt interconnect. Foundational and the
   riskiest; do it first. *(The cluster will also run without it on a slow TCP
   fallback — see §1.5 to de-risk by validating vLLM first, then adding RDMA.)*
2. **vLLM engine** (`container/`) — rebuild the patched image, one distrobox per box.
3. **Host orchestration** (`host/`) — the launch scripts, env, model weights.

## 0.1 Prerequisites (verify these exist; do NOT try to synthesize them)

- **2× AMD Strix Halo / gfx1151**, ~128 GB unified memory each, on the same LAN.
- A **Thunderbolt-4 / USB4 cable** physically connecting the two boxes.
- Linux with **kernel headers/devel** for the running kernel on each box, `podman`,
  `distrobox`, `rdma-core`/`libibverbs`, `git`, build toolchain.
- The model weights **`deepseek-ai/DeepSeek-V4-Flash-0731`** (~150 GB) downloaded
  on **both** boxes (`hf download deepseek-ai/DeepSeek-V4-Flash-0731`).
- Root/sudo on both boxes (kernel modules, systemd units).
- **Secure Boot disabled on both boxes** — the tbv modules are unsigned;
  a Secure Boot kernel will refuse every `insmod` in §1.

Pick roles now and keep them consistent everywhere: **box1 = ray head**, IP on
Thunderbolt `192.168.100.1`; **box2 = worker**, `192.168.100.2`. Site values
(IPs, container name, transport, HCA pin, disk KV) live in
`host/ds4-config.yaml`, deployed as `~/ds4-config.yaml` on box1 (see §3);
paths in the scripts are `$HOME`-relative.

---

## 0.2 Recall / context integrity — fixed, gated

Long-context recall on this stack is correct for the deployed profile. What
keeps it correct:

- `DS4_IDX_OFFICIAL=1` in `host/ds4-cluster-env.sh` -- the sparse indexer's
  official Hadamard128 + FP4 QAT scoring graph. Must be engaged on BOTH TP
  ranks (it is exported from the shared env, so keep the env identical).
- The `deepseek_v4_encoding.py` patch -- the chat encoder no longer strips
  prior assistant reasoning on tool conversations.

Any change to the indexer, MTP, kernels, or tuning knobs must re-pass
needle/recall probes at your target context depth before it ships (see §5).

---

## 1. tbv — Thunderbolt RDMA (do this first)

Full detail in [`tbv/README.md`](tbv/README.md); this is the ordered action list.
**The kernel modules are vermagic-locked to a specific kernel**,
so plan to build for your exact kernel. Run every module step on **both**
boxes.

### 1.1 Build the matched core+net (per box, per kernel)

The `thunderbolt` core, `thunderbolt_net`, and `thunderbolt_ibverbs` must be one
matched set or the box **panics on cable connect**.

```bash
tbv/build-modules.sh [KVER]          # all four modules, no sudo
sudo tbv/install-modules.sh [KVER]   # stage /var/lib/tbv + blacklist + boot units
```

`build-modules.sh` fetches the pinned upstream trees (westeri @`503c5ae`;
hellas-ai/thunderbolt-ibverbs @`76ba39b` + `tbv/ibverbs-local.patch`), applies
the kernel patch series the ibverbs repo carries, and builds against the
target kernel-devel. Gotchas it handles:
- Force **`CONFIG_USB4_CONFIGFS=y`** in the KDIR `auto.conf` (else
  `tb_configfs_init/exit` are undefined at link).
- Build `thunderbolt_net` with **`KBUILD_EXTRA_SYMBOLS=<core>/Module.symvers`**
  (it needs `tb_ring_throttling` from the patched core).
- MODVERSIONS is off; only **vermagic** must match `uname -r`.

### 1.2 Build the out-of-tree modules

`build-modules.sh` above already builds ibverbs + nhi_throttle (steps 3-4)
against the same patched KDIR; nothing separate to run.

### 1.3 Install the userspace provider (host AND container)

```bash
# The serving container image builds and ships the provider itself
# (container/Dockerfile provider-build stage) — nothing to install for serving.
# For host-side diagnostics (ibv_devices on the host), build the same way:
# rdma-core v57.0 + the provider patches from the upstream ibverbs repo.
```
The provider matches devices **by name**, which is why the device must be
renamed to `usb4_rdma*`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlexKGwyn/ds4-vllm](https://github.com/AlexKGwyn/ds4-vllm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-18 -->
