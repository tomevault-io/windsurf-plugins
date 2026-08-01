---
trigger: always_on
description: This document provides guidelines for AI-assisted development of this NES (Nintendo Entertainment System) emulator written in PHP.
---

# Junie Development Guidelines for NES Emulator

This document provides guidelines for AI-assisted development of this NES (Nintendo Entertainment System) emulator written in PHP.

## Agent Document Source of Truth

- `AGENTS.md` is the master agentic development document for this repository.
- If this document disagrees with `AGENTS.md`, follow `AGENTS.md`.
- Keep this document and all other agentic development documents in sync with `AGENTS.md`, including `.cursorrules`, `CLAUDE.md`, and `.github/copilot-instructions.md`.
- When changing agent guidance, update `AGENTS.md` first, then update the corresponding Cursor, Claude, Copilot, Junie, and contributor-facing documentation in the same change.

## Project Overview

This is a cycle-accurate NES emulator implementation in PHP 8.5+ using:
- **Graphics**: PHP-GLFW (OpenGL) via the VISU framework
- **Architecture**: 6502 CPU emulation, PPU (Picture Processing Unit), APU (Audio Processing Unit - planned)
- **Testing**: PHPUnit with comprehensive unit and integration tests
- **Quality**: PHPStan for static analysis, Laravel Pint for code formatting

## Code Style & Standards

### Documentation

**Properties and Methods:**
- All properties must have docblocks describing their purpose
- All methods must have docblocks describing what they do
- Only include `@param` or `@return` when generics need to be declared (e.g., `list<int>`, `array<int, string>`)
- Use `@throws`, `@see`, or `@inheritDoc` when appropriate
- Do NOT add class-level docblocks

**Example:**
```php
/**
 * The current PPU cycle within a scanline.
 */
private int $cycle = 0;

/**
 * Executes a single CPU instruction and returns the number of cycles consumed.
 *
 * @throws \RuntimeException
 */
public function run(): int
{
    // ...
}
```

### Comments

**General Rules:**
- Remove superfluous comments that merely restate the code
- All comments must end with a period (.)
- Multi-line comments use C-style `/* */` syntax
- Single-line comments use `//` syntax
- Keep comments that explain complex logic or non-obvious behavior

**Example:**
```php
/* Set up reset vector to point to 0x8000 (start of ROM).
 * Reset vector is at 0xFFFC-0xFFFD. */
$romData[0x7FFC] = 0x00;
$romData[0x7FFD] = 0x80;

// 6502 stack is from 0x0100-0x01FF, initialized to 0x01FD.
$this::assertSame(0x01FD, $registers->sp);
```

### Code Organization

**File Structure:**
- One class per file
- Namespace must match directory structure
- Use strict types: `declare(strict_types=1);`
- Order: properties first, constructor, public methods, protected methods, private methods

**Naming Conventions:**
- Classes: PascalCase
- Methods: camelCase
- Properties: camelCase
- Constants: SCREAMING_SNAKE_CASE
- Test methods: snake_case with `it_` prefix (e.g., `it_handles_8bit_values`)

## Architecture Patterns

### Emulator Components

**CPU (Central Processing Unit):**
- Located in `src/Cpu/`
- Implements 6502 instruction set (official and unofficial opcodes)
- Handles interrupts (NMI, IRQ)
- Cycle-accurate execution
- See `src/Cpu/Cpu.php` for reference

**PPU (Picture Processing Unit):**
- Located in `src/Graphics/`
- Handles rendering (background tiles, sprites)
- Manages VRAM, sprite RAM, palettes
- Runs at 3x CPU speed
- Generates VBlank interrupts
- See `src/Graphics/Ppu.php` for reference

**Memory Bus Architecture:**
- `CpuBus`: CPU address space ($0000-$FFFF)
- `PpuBus`: PPU address space ($0000-$3FFF)
- RAM mirroring implemented correctly
- Memory-mapped I/O for PPU, gamepad, DMA

**Key Classes:**
- `Cpu`: 6502 CPU emulation
- `Ppu`: Picture Processing Unit
- `Dma`: Direct Memory Access for sprite data
- `CpuBus`, `PpuBus`: Memory buses
- `Ram`, `Rom`: Memory components
- `Cartridge`, `Loader`: ROM file handling
- `Gamepad`: Input handling
- `Renderer`: Converts PPU output to framebuffer

### Value Objects

Use readonly classes for immutable data:
```php
readonly class Registers
{
    public function __construct(
        public int $a,
        public int $x,
        public int $y,
        public Status $p,
        public int $sp,
        public int $pc,
    ) {}
}
```

### Dependency Injection

Constructor injection is preferred:
```php
public function __construct(
    private readonly CpuBus $bus,
    private readonly Interrupts $interrupts,
) {}
```

## Testing Guidelines

### Unit Tests

**Structure:**
- Located in `tests/Unit/`
- Use `#[Test]` attribute instead of `test` prefix
- Use `#[CoversClass(ClassName::class)]` attribute
- Test method names: `it_does_something_specific()`

**Example:**
```php
#[CoversClass(Ram::class)]
final class RamTest extends TestCase
{
    #[Test]
    public function it_writes_and_reads_data(): void
    {
        $ram = new Ram(256);
        $ram->write(100, 0x42);
        
        $this::assertSame(0x42, $ram->read(100));
    }
}
```

**Helper Methods:**
- Add docblocks to helper methods
- setUp/tearDown methods use `@inheritDoc`
- Extract common setup to helper methods

### Integration Tests

**Structure:**
- Located in `tests/Integration/`
- Extend `IntegrationTestCase` for common utilities
- Test complete system interactions
- Use `requireTestRom()` for ROM file tests

**Example:**
```php

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oliverearl/nes-php-glfw](https://github.com/oliverearl/nes-php-glfw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
