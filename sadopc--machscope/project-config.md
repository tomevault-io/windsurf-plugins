---
trigger: always_on
description: Auto-generated from feature plans. Last updated: 2026-01-12
---

# MachScope Development Guidelines

Auto-generated from feature plans. Last updated: 2026-01-12

## Project Overview

MachScope is a native macOS binary analysis tool providing Mach-O parsing, ARM64 disassembly, and optional process debugging. Pure Swift implementation with no external dependencies.

## Active Technologies

- **Language**: Swift 6.2.3 with `SWIFT_STRICT_CONCURRENCY=complete`
- **Frameworks**: Darwin, Foundation, Security (system frameworks only)
- **Platform**: arm64-apple-macosx26.0
- **Build System**: Swift Package Manager

## Project Structure

```text
Package.swift                           # Swift Package manifest (4 targets)
.gitignore                              # Swift/Xcode ignore patterns

Sources/
├── MachOKit/                           # Core Mach-O parsing library
│   ├── MachOBinary.swift               # Main entry point for parsing
│   ├── Header/
│   │   ├── MachHeader.swift            # mach_header_64, FileType, MachHeaderFlags
│   │   ├── FatHeader.swift             # Fat/Universal binary header, FatArch
│   │   └── CPUType.swift               # CPUType, CPUSubtype enums
│   ├── LoadCommands/
│   │   ├── LoadCommand.swift           # Base LoadCommand, LoadCommandType enum
│   │   ├── SegmentCommand.swift        # LC_SEGMENT_64 parsing
│   │   ├── SymtabCommand.swift         # LC_SYMTAB parsing
│   │   ├── DyldCommand.swift           # LC_LOAD_DYLIB and related
│   │   └── CodeSignatureCommand.swift  # LC_CODE_SIGNATURE parsing
│   ├── Sections/
│   │   ├── Segment.swift               # Segment struct, VMProtection
│   │   └── Section.swift               # Section struct, SectionType enum
│   ├── Symbols/
│   │   ├── Symbol.swift                # Symbol struct, SymbolType enum
│   │   ├── SymbolTable.swift           # Lazy symbol table loading
│   │   ├── StringTable.swift           # String table for symbol names
│   │   └── StringExtractor.swift       # Extract strings from sections
│   ├── CodeSignature/
│   │   ├── SuperBlob.swift             # Code signature SuperBlob parser
│   │   ├── CodeDirectory.swift         # CodeDirectory, HashType enum
│   │   └── Entitlements.swift          # XML/DER entitlements parser
│   ├── IO/
│   │   ├── BinaryReader.swift          # Bounds-checked binary reading
│   │   └── MemoryMappedFile.swift      # mmap() wrapper for large files
│   └── Errors/
│       └── MachOParseError.swift       # All parsing error cases with context
│
├── Disassembler/                       # ARM64 instruction decoder
│   ├── ARM64Disassembler.swift         # Main disassembler entry point
│   ├── Instruction.swift               # Instruction model, InstructionCategory
│   ├── Decoder/
│   │   ├── InstructionDecoder.swift    # Base decoder with bit extraction
│   │   ├── DataProcessing.swift        # ADD, SUB, MOV, etc.
│   │   ├── Branch.swift                # B, BL, BR, RET
│   │   ├── LoadStore.swift             # LDR, STR, LDP, STP
│   │   └── System.swift                # SVC, NOP, PAC instructions
│   ├── Formatter/
│   │   ├── InstructionFormatter.swift  # Assembly notation output
│   │   └── OperandFormatter.swift      # Operand display formatting
│   ├── Analysis/
│   │   ├── SymbolResolver.swift        # SymbolResolving protocol
│   │   ├── PACAnnotator.swift          # PAC instruction highlighting
│   │   └── SwiftDemangler.swift        # Swift symbol demangling
│   └── Errors/
│       └── DisassemblyError.swift      # Disassembly error cases
│
├── DebuggerCore/                       # Process debugging (requires entitlements)
│   ├── Debugger.swift                  # Main debugger class
│   ├── Process/
│   │   ├── TaskPort.swift              # task_for_pid wrapper
│   │   ├── ProcessAttachment.swift     # Attach/detach management
│   │   ├── ThreadState.swift           # Thread management
│   │   └── ARM64Registers.swift        # x0-x30, sp, pc, cpsr
│   ├── Breakpoints/
│   │   ├── Breakpoint.swift            # Breakpoint model
│   │   └── BreakpointManager.swift     # Set/remove/hit management
│   ├── Memory/
│   │   ├── MemoryReader.swift          # vm_read wrapper
│   │   └── MemoryWriter.swift          # vm_write wrapper
│   ├── Exceptions/
│   │   ├── ExceptionHandler.swift      # Mach exception handler
│   │   └── MachExceptionServer.swift   # Exception port server
│   ├── Permissions/
│   │   ├── PermissionChecker.swift     # Tiered capability detection
│   │   ├── EntitlementValidator.swift  # Debugger entitlement check
│   │   └── SIPDetector.swift           # SIP status detection
│   └── Errors/
│       └── DebuggerError.swift         # Debugger error cases
│
└── MachScope/                          # CLI executable
    ├── main.swift                      # Entry point, command dispatch
    ├── Commands/
    │   ├── ParseCommand.swift          # machscope parse
    │   ├── DisasmCommand.swift         # machscope disasm
    │   ├── DebugCommand.swift          # machscope debug
    │   └── CheckPermissionsCommand.swift # machscope check-permissions
    ├── Output/
    │   ├── TextFormatter.swift         # Human-readable output
    │   └── JSONFormatter.swift         # Machine-readable JSON output
    └── Utilities/
        └── ArgumentParser.swift        # CLI argument parsing

Tests/
├── MachOKitTests/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sadopc/machscope](https://github.com/sadopc/machscope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
