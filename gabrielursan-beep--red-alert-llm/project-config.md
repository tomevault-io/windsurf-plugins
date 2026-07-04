---
trigger: always_on
description: Portable offline AI assistant + knowledge base + encrypted personal vault on a Kingston 1TB Dual Portable SSD (USB-A + USB-C, USB 3.2 Gen 2, ExFAT). Cross-platform: Windows, macOS, Linux, Android, iOS.
---

# Red Alert LLM — Claude Code Instructions

## Project Purpose
Portable offline AI assistant + knowledge base + encrypted personal vault on a Kingston 1TB Dual Portable SSD (USB-A + USB-C, USB 3.2 Gen 2, ExFAT). Cross-platform: Windows, macOS, Linux, Android, iOS.

## Affiliate & Pricing Note
- SSD affiliate link (eMAG.ro): https://bit.ly/4uEu0nA
- Product: SSD Portabil Dual Kingston, 1TB, USB Type-A/USB Type-C, USB 3.2 Gen 2, rosu/negru
- Price: 743 RON (as of 24 March 2026)
- All eMAG links in docs use this affiliate link. Be transparent about it in all documentation.
- Do NOT estimate prices for alternative SSDs. Only state the verified Kingston price.

## Target Audience
Romanian users (and anyone) who want offline AI access for emergencies, travel, privacy, or areas with no internet. End product must be usable by complete beginners — scripts self-explanatory, bilingual (EN + RO).

## Tech Stack

### LLM Inference Engines
- **llamafile v0.10.0** (Mozilla) — primary engine, single executable, runs on Win/Mac/Linux, built-in web UI at :8080
  - GitHub: https://github.com/mozilla-ai/llamafile/releases/tag/0.10.0
  - Binary: https://github.com/mozilla-ai/llamafile/releases/download/0.10.0/llamafile-0.10.0
  - Does NOT run on Android or iOS
- **KoboldCpp v1.110** — secondary engine, richer UI (KoboldAI Lite), single executable
  - GitHub: https://github.com/LostRuins/koboldcpp/releases/tag/v1.110
  - Win (no CUDA): koboldcpp-nocuda.exe (92 MB)
  - macOS ARM64: koboldcpp-mac-arm64 (44 MB)
  - Linux: koboldcpp-linux-x64-nocuda (109 MB)
  - Has Termux auto-installer for Android (v1.90.2+)

### AI Models (GGUF format, Q4_K_M quantization)
| Model | Size | Min RAM | Languages | Romanian | Use Case |
|-------|------|---------|-----------|----------|----------|
| Qwen3-4B | 2.5 GB | 6 GB | 119 | Yes | Primary — phones + low RAM PCs |
| Qwen3-8B | 5.0 GB | 10 GB | 119 | Yes | Enhanced — 12GB+ RAM |
| Qwen3-14B | 9.0 GB | 14 GB | 119 | Yes | Premium — 16GB+ RAM |
| Gemma-3-4B-IT | 2.5 GB | 6 GB | 140+ | Yes | Alternative — multimodal |
| Gemma-3-12B-IT | 7.0 GB | 12 GB | 140+ | Yes | Alternative — larger |
| Whisper-large-v3 | 3.0 GB | 4 GB | 99 | Yes | Speech-to-text |

Model sources:
- Qwen3: https://huggingface.co/Qwen/Qwen3-4B-GGUF (and 8B, 14B)
- Gemma 3: https://huggingface.co/ggml-org/gemma-3-4b-it-GGUF
- Also check bartowski repos for Q4_K_M quants

### Knowledge Base
- **Kiwix Desktop** — offline reader for ZIM files (Win/Mac/Linux/Android/iOS)
  - Download: https://kiwix.org/en/applications/
  - ZIM catalog: https://download.kiwix.org/zim/
  - Android: use FULL APK from website, NOT Play Store (scoped storage issues)
  - iOS: App Store, but must copy ZIM to device storage
- **kiwix-serve** — serve ZIM over local network (any device with a browser)

### Encrypted Vault
- **VeraCrypt Portable** — 200 GB encrypted container (vault.hc)
  - AES-256, SHA-512, ExFAT internal filesystem
  - Works on Win (portable), Mac, Linux
  - Does NOT work on Android/iOS

## Key Constraints

### ExFAT Filesystem Rules
- NO symlinks (ExFAT doesn't support them)
- NO Unix permissions (ExFAT ignores chmod)
- NO journaling — always remind users to safely eject
- Max filename: 255 chars, max path: 32,767 chars
- Case-preserving but case-insensitive

### Script Rules
- Windows scripts: .bat format, use `%~dp0` and `%~d0` for paths
- macOS scripts: .command format (double-clickable), use `$(dirname "$0")` for paths
- All .command/.sh files need Unix line endings (LF, not CRLF)
- llamafile binary: `llamafile.exe` on Windows, `llamafile` (no extension) on Mac/Linux
- The `-ngl 999` flag is CRITICAL for macOS — enables Metal GPU acceleration
- The `.portable` file in kiwix app folder enables portable mode (empty file)
- All scripts must be bilingual (Romanian + English) in echo messages

### Windows-Specific
- llamafile on Windows cannot exceed 4GB as single .exe — use engine + external GGUF approach
- Windows SmartScreen will block unsigned executables — scripts must handle this
- Use `wmic computersystem get totalphysicalmemory` for RAM detection

### macOS-Specific
- Gatekeeper blocks unsigned binaries: `xattr -dr com.apple.quarantine` required
- Three methods to unblock: xattr command, System Preferences > Security, right-click > Open
- Kiwix macOS cannot run portably — must install from App Store, but reads ZIM from USB

### Android-Specific
- llamafile does NOT run on Android
- Best beginner option: PocketPal AI or ChatterUI (free apps, load GGUF from storage)
- KoboldCpp available via Termux auto-installer (advanced users)
- Kiwix Android (FULL APK, not Play Store) reads ZIM from USB-OTG (v3.8+)
- Include APKs on the SSD for sideloading

### iOS-Specific
- Most restricted platform — models cannot stream from external USB
- PocketPal AI, LLM Farm, LM Studio can load GGUF but user must copy to device first
- Kiwix iOS: copy ZIM to device via Files app
- Realistic workflow: connect SSD, copy 2.5GB model via Files, load in app
- Include mobile/ folder with smaller quants (Q3_K_M) for storage-limited devices

## Storage Budget (1TB = ~931 GiB usable)

| Tier | Content | Size |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabrielursan-beep/red-alert-llm](https://github.com/gabrielursan-beep/red-alert-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
