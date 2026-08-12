---
trigger: always_on
description: UEFI application that boots directly into AI inference with tool use. No kernel, no OS — runs entirely in UEFI boot services mode.
---

# BootAI

UEFI application that boots directly into AI inference with tool use. No kernel, no OS — runs entirely in UEFI boot services mode.

## Relationship to Echad

BootAI is the **static core** of the Echad distributed cluster system (`../Echad/`). It is not a standalone product — it is the foundation every Echad node runs on. The cluster features (Raft consensus, distributed storage, state store) are built on top of BootAI.

BootAI's code compiles for multiple targets via a platform HAL:

| Target | HAL | Binary | Boot |
|--------|-----|--------|------|
| Tier 1 (UEFI node) | `efi/` | `all_bootai.efi` | UEFI direct |
| GPU node | `linux/` (to be built) | `echad-daemon` | Minimal Linux kernel (EFISTUB) |
| Tier 2 (Desktop) | `app/` (to be built) | Desktop app | User's OS |

Platform-independent code (`model/`, `tokenizer/`, `tools/`, `foundry/`) compiles for all targets. Platform-specific code (`efi/`, `linux/`, `app/`) implements the same HAL interface with different backends.

See `../Echad/Echad.md` for the full architecture.

## Project Structure

```
bootai/
├── CLAUDE.md
├── README.md
├── Makefile
├── efi/
│   ├── main.c                   # EFI entry point → boot menu → inference REPL
│   ├── efi.h                    # UEFI protocol types and helpers
│   ├── console.c                # Text output via ConOut + log tee to bootai.log
│   ├── input.c                  # Keyboard via Simple Text Input protocol
│   ├── fs.c                     # File ops via Simple File System protocol
│   ├── net.c                    # Networking via TCP4/HTTP protocols
│   └── mdns.c                   # mDNS/Bonjour responder (bootai.local)
├── model/
│   ├── model.h                  # Common model interface (forward, load, sample)
│   ├── qwen2.h                  # Qwen2.5 config + forward pass
│   ├── qwen2.c                  # Transformer: RoPE, GQA, SwiGLU (Q4 weights)
│   ├── rwkvx.h                  # RWKV-X config + forward pass
│   ├── rwkvx.c                  # Hybrid: RWKV-7 blocks + sparse attention (Q8)
│   ├── wkv7.c                   # WKV-7 kernel
│   ├── rope.c                   # Rotary position embeddings (Qwen2.5)
│   ├── kv_cache.c               # KV cache for Qwen2.5 + sparse attn in RWKV-X
│   ├── loader.c                 # Load quantized weights, auto-detect architecture
│   └── sampling.c               # Temperature, top-k, top-p sampling
├── tools/
│   ├── tool_use.h               # Tool call parser + dispatcher
│   ├── tool_use.c               # Parse <tool_call> JSON, dispatch to backends
│   ├── tools_fs.c               # read_file, write_file, list_dir (UEFI FS)
│   ├── tools_net.c              # web_search, web_fetch (UEFI TCP4/HTTP)
│   ├── tools_system.c           # system_info, memory stats (UEFI)
│   └── export_weights.py        # Convert safetensors → flat quantized binary
├── tokenizer/
│   ├── bpe.c                    # BPE tokenizer (Qwen2.5)
│   ├── rwkv_tokenizer.c         # RWKV World tokenizer (trie-based)
│   └── vocab.bin                # Vocabulary data (per-model)
├── foundry/                     # Submodule: pure C tensor runtime
├── JACLibc/                     # Submodule: header-only libc
└── docs/
```

## Tech Stack

- **Language**: C (C99, `-ffreestanding`)
- **Tensor Runtime**: Foundry (`foundry/tensor/tensor.c`) — pure C, 9300 lines, 141 tests
- **Libc**: JACLibc (header-only, bare-metal compatible)
- **Target**: x86_64 UEFI
- **Models**: Qwen2.5-Coder-7B Q4, RWKV-X 3.6B Q8
- **Cross-compiler**: `x86_64-elf-gcc` (from macOS)
- **Emulator**: QEMU + OVMF firmware

## Build Commands

```bash
make efi              # Build all_bootai.efi (full build — boots into menu)
make PROD=1 efi       # Build production EFI (boots straight into chat)
make release          # Build PROD + GPT disk image (flashable with balenaEtcher/dd)
make run              # Run in QEMU with OVMF (interactive — do NOT run from Claude)
make run-prod         # Clean build PROD + run in QEMU (straight to chat — do NOT run from Claude)
make clean            # Clean build artifacts

# Model export
python3 tools/export_weights.py --model Qwen/Qwen2.5-Coder-7B-Instruct --quant q4_k_m --output model.bin
```

### Build Flags

| Flag | What | When |
|------|------|------|
| `PROD=1` | Skip boot menu and REPL, boot straight into chat inference | Production/demo builds |

## Nodes

| Name | Machine | IP | Notes |
|------|---------|-----|-------|
| Dell | Dell E6510 | 192.168.1.85 | Primary dev node, Westmere SSE4.2, 8 GB |

## OTA Firmware Update (Preferred)

When a node is running and serving (after `serve 8080`), use OTA instead of USB flashing:

```bash
# 1. Build
make clean && make efi

# 2. Compute local CRC32 BEFORE pushing
python3 -c "import binascii; print(format(binascii.crc32(open('all_bootai.efi','rb').read()) & 0xFFFFFFFF, '08x'))"

# 3. Push firmware to Dell
curl -H "Authorization: Bearer bootai" \
     --data-binary @all_bootai.efi \
     http://<DELL_IP>:8080/api/update
# Response includes "crc32":"XXXXXXXX" — VERIFY it matches step 2 before proceeding!

# 4. Only after CRC32 match confirmed, reboot Dell
curl -H "Authorization: Bearer bootai" \
     -d '{"cmd":"reboot"}' \
     http://<DELL_IP>:8080/api/cmd


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dcherrera/bootai](https://github.com/dcherrera/bootai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
