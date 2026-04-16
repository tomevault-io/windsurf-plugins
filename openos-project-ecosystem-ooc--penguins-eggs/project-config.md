---
trigger: always_on
description: `penguins-eggs` is a command-line tool for remastering AlmaLinux, AlpineLinux, Arch, Debian, Devuan, Fedora, Manjaro, Openmamba, openSuSE, RockyLinux, Ubuntu, and derivative systems.
---

# Gemini Project Context: penguins-eggs

## Project Overview
`penguins-eggs` is a command-line tool for remastering AlmaLinux, AlpineLinux, Arch, Debian, Devuan, Fedora, Manjaro, Openmamba, openSuSE, RockyLinux, Ubuntu, and derivative systems.

**The "Magic":** Unlike simple backup tools (like Clonezilla), `eggs` creates a live, bootable ISO image that is **hardware independent**. It removes specific user data and system identifiers (UUIDs, SSH keys), allowing the generated ISO to be installed on different hardware as a fresh distribution.

## Tech Stack
- **Language:** TypeScript
- **Framework:** [oclif](https://oclif.io/)
- **Package Manager:** pnpm
- **Testing:** Mocha & Chai
- **Key Libraries:** `execa` (for shell commands), `chalk` (for TUI colors), `inquirer` (for prompts).
- **Compression:** SquashFS (supports zstd, xz, gzip).

## Upgrade Tracking

### typescript-go (TS 7 native compiler)
`microsoft/typescript-go` is a Go port of the TypeScript compiler targeting a ~10× build speed improvement.
It is available as `@typescript/native-preview` on npm (preview channel).
Currently pinned to `typescript: ^5.9.3`. When `typescript-go` reaches stable/GA:
1. Replace `typescript` devDependency with `@typescript/native-preview` (or the stable package name once announced).
2. Verify `tsconfig.json` compatibility — `module: Node16` and `moduleResolution: node16` are supported.
3. Remove `ts-node` if it becomes redundant (tsgo includes a native runner).
Track progress at: https://github.com/microsoft/typescript-go

## Installers
- **krill:** A TUI system installer developed within eggs. Lightweight, fast, always available.
- **calamares:** Integration with the popular GUI installer. Eggs creates the configuration files needed for Calamares to install the custom ISO.

## Command Palette (The Family Metaphor)
The CLI organizes commands using a family metaphor:
- **`eggs produce`**: The core process. Remasters the system and creates the ISO.
- **`eggs dad`**: Configuration manager. Helps set compression levels, paths, and reset defaults.
- **`eggs kill`**: Cleanup utility. Removes temporary data (`/home/eggs`) and old ISOs to free space.
- **`eggs cuckoo`**: PXE Boot utility. Configures a local PXE server to boot the ISO over the network.
- **`eggs wardrobe`**: Interface to fetch and apply configurations ("costumes") from `penguins-wardrobe`.

## User Key Commands
- **`eggs love`**: The "magic button". Automatically runs: `eggs dad -d` (reset), `eggs tools clean` (cleanup), and `eggs produce` (create ISO).
- **`eggs produce --fast`**: Creates an ISO using lighter compression (faster build, larger file).
- **`eggs produce --max`**: Creates an ISO using maximum compression (slower build, smallest file).
- **`eggs produce --clone`**: Creates a backup of the current system *including* user data in clear text.
- **`eggs produce --homecrypt`**: Creates a backup of the current system *including* user data (encrypted via LUKS).
- **`eggs produce --fullcrypt`**: Creates a backup where *the entire system* is encrypted via LUKS.

---

## Project Timeline & Evolution
In short, the story of **eggs** is a journey from a Debian remastering tool to a universal ISO creation system capable of handling almost any Linux distribution, adapting to various architectures, and offering powerful backup and customization tools.

### 🐣 The Primordial Era (v7.x) - Consolidation and Basic Tools
In this phase, the focus was primarily on the Debian/Ubuntu/Devuan family.
* **Installers:** Intensive work to make `krill` (the CLI installer) reliable, along with standard integration with Calamares for graphical installation.
* **Yolk:** Introduction of the concept of *yolk*, a small local repository that allowed for installation even without an internet connection.
* **Compatibility:** Transition and support across different Node versions (8, 14, 16).
* **Themes:** Initial experiments with custom themes for the ISOs.

### 🛠️ The Era of Technical Expansion (v8.x) - Architectures and Backup
The project began to expand beyond simple x86 remastering.
* **ARM Support:** Introduction of support for arm64 and armel, paving the way for Raspberry Pi and other devices.
* **Backup & Encryption:** Implementation of the `--backup` feature, allowing user data and server configurations to be saved in an encrypted LUKS volume within the ISO, which could then be restored via `krill`.
* **UEFI:** Significant improvements for UEFI boot support, including Secure Boot management in later stages.
* **Refactoring:** Rewriting of core classes like `pacman` and dependency management.

### 👗 The Wardrobe Era (v9.x) - Customization and New Families
A phase of great creativity and openness to new distributions.
* **Wardrobe & Costumes:** Introduction of *Wardrobe*, a system to "dress" a "naked" system with specific configurations, graphical environments, and packages ("costumes").
* **Beyond Debian:** The beginning of concrete support for other distribution families like Arch Linux and Manjaro.
* **Pods:** First experiments with *eggs pods* to create minimal live images starting from containers (Docker/Podman).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OpenOS-Project-Ecosystem-OOC) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
