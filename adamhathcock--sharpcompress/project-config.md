---
trigger: always_on
description: Guidelines for building SharpCompress - A C# compression library
---


# SharpCompress Development

## About SharpCompress
SharpCompress is a pure C# compression library supporting multiple archive formats (Zip, Tar, GZip, BZip2, 7Zip, Rar, LZip, XZ, ZStandard, Arc, Arj, Ace, LZW). The project currently targets .NET Framework 4.8, .NET Standard 2.0/2.1, .NET 6.0, .NET 8.0, and .NET 10.0. The library provides both seekable Archive APIs and forward-only Reader/Writer APIs for streaming scenarios.

## C# Instructions
- Use language features supported by the current project toolchain (`LangVersion=latest`) and existing codebase patterns.
- Add comments for non-obvious logic and important design decisions; avoid redundant comments.
- Follow the existing code style and patterns in the codebase.

## General Instructions
- **Do not commit or stage changes unless the user explicitly asks for it.**
- Make only high confidence suggestions when reviewing code changes.
- Write code with good maintainability practices, including comments on why certain design decisions were made.
- Handle edge cases and write clear exception handling.
- For libraries or external dependencies, mention their usage and purpose in comments.
- Preserve backward compatibility when making changes to public APIs.

### Workspace Hygiene
- Do not edit generated or machine-local files unless required for the task (for example: `bin/`, `obj/`, `*.csproj.user`).
- Avoid broad formatting-only diffs in unrelated files.

## Naming Conventions

- Follow PascalCase for component names, method names, and public members.
- Use camelCase for private fields and local variables.
- Prefix interface names with "I" (e.g., IUserService).

## Code Formatting

**Copilot agents: You MUST run the `format` task after making code changes to ensure consistency.**

- Use CSharpier for code formatting to ensure consistent style across the project
- CSharpier is configured as a local tool in `.config/dotnet-tools.json`

### Commands

1. **Restore tools** (first time only):
   ```bash
   dotnet tool restore
   ```

2. **Check if files are formatted correctly** (doesn't modify files):
   ```bash
   dotnet csharpier check .
   ```
   - Exit code 0: All files are properly formatted
   - Exit code 1: Some files need formatting (will show which files and differences)

3. **Format files** (modifies files):
   ```bash
   dotnet csharpier format .
   ```
   - Formats all files in the project to match CSharpier style
   - Run from project root directory

4. **Configure your IDE** to format on save using CSharpier for the best experience

### Additional Notes
- The project also uses `.editorconfig` for editor settings (indentation, encoding, etc.)
- Let CSharpier handle code style while `.editorconfig` handles editor behavior
- Always run `dotnet csharpier check .` before committing to verify formatting

## Project Setup and Structure

- The project targets multiple frameworks: .NET Framework 4.8, .NET Standard 2.0/2.1, .NET 6.0, .NET 8.0, and .NET 10.0
- Main library is in `src/SharpCompress/`
- Tests are in `tests/SharpCompress.Test/`
- Performance tests are in `tests/SharpCompress.Performance/`
- Test archives are in `tests/TestArchives/`
- Build project is in `build/`
- Use `dotnet build` to build the solution
- Use `dotnet test` to run tests
- Solution file: `SharpCompress.slnx`

### Directory Structure
```
src/SharpCompress/
  ├── Archives/        # IArchive implementations (Zip, Tar, Rar, 7Zip, GZip)
  ├── Readers/         # IReader implementations (forward-only)
  ├── Writers/         # IWriter implementations (forward-only)
  ├── Compressors/     # Low-level compression streams (BZip2, Deflate, LZMA, etc.)
  ├── Factories/       # Format detection and factory pattern
  ├── Common/          # Shared types (ArchiveType, Entry, Options)
  ├── Crypto/          # Encryption implementations
  └── IO/              # Stream utilities and wrappers

tests/SharpCompress.Test/
  ├── Zip/, Tar/, Rar/, SevenZip/, GZip/, BZip2/  # Format-specific tests
  ├── TestBase.cs      # Base test class with helper methods

tests/
  ├── SharpCompress.Test/         # Unit/integration tests
  ├── SharpCompress.Performance/  # Benchmark tests
  └── TestArchives/               # Test data archives
```

### Factory Pattern
Factory implementations can implement one or more interfaces (`IArchiveFactory`, `IReaderFactory`, `IWriterFactory`) depending on format capabilities:
- `ArchiveFactory.OpenArchive()` - Opens archive API objects from seekable streams/files
- `ArchiveFactory.OpenAsyncArchive()` - Opens async archive API objects for async archive use cases
- `ReaderFactory.OpenReader()` - Auto-detects and opens forward-only readers
- `ReaderFactory.OpenAsyncReader()` - Auto-detects and opens forward-only async readers
- `WriterFactory.OpenWriter()` - Creates a writer for a specified `ArchiveType`
- `WriterFactory.OpenAsyncWriter()` - Creates an async writer for async write scenarios
- Factories located in: `src/SharpCompress/Factories/`

## Nullable Reference Types

- Declare variables non-nullable, and check for `null` at entry points.
- Always use `is null` or `is not null` instead of `== null` or `!= null`.
- Trust the C# null annotations and don't add null checks when the type system says a value cannot be null.

## SharpCompress-Specific Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamhathcock/sharpcompress](https://github.com/adamhathcock/sharpcompress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
