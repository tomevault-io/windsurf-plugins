---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

**MaxMind-DB-Reader-dotnet** is the .NET API for reading MaxMind DB files.
MaxMind DB is a binary file format that stores data indexed by IP address
subnets (IPv4 or IPv6). This is the lower-level library used by the
GeoIP2-dotnet client library.

**Key Technologies:**

- .NET 10.0, .NET 9.0, .NET 8.0, .NET Standard 2.1, and .NET Standard 2.0
- xUnit for testing
- BenchmarkDotNet for performance benchmarking
- Modern C# features (nullable reference types, unsafe code, Span<T>)

## Development Commands

### Building

```bash
# Build all projects
dotnet build MaxMind.Db
dotnet build MaxMind.Db.Test
dotnet build MaxMind.Db.Benchmark

# Build entire solution
dotnet build MaxMind.Db.sln
```

### Running Tests

```bash
# Run all tests
dotnet test MaxMind.Db.Test/MaxMind.Db.Test.csproj

# Run specific test class
dotnet test --filter "FullyQualifiedName~ReaderTest"
dotnet test --filter "FullyQualifiedName~DecoderTest"

# Run specific test method
dotnet test --filter "FullyQualifiedName~ReaderTest.TestMany"
```

### Running Benchmarks

```bash
# Run benchmarks (must specify target framework)
dotnet run -c Release -f net10.0 -p MaxMind.Db.Benchmark/MaxMind.Db.Benchmark.csproj
```

### Test Data Submodule

The test suite requires the MaxMind-DB test data git submodule:

```bash
# Initialize submodule (if not already done)
git submodule update --init --recursive

# Update submodule to latest
git submodule update --remote
```

Tests expect to find test databases in
`MaxMind.Db.Test/TestData/MaxMind-DB/test-data/`.

## Code Architecture

### Core Components

The library has three main architectural layers:

1. **Buffer Layer** - File access abstraction
   - `MemoryMapBuffer`: Memory-mapped file implementation for all file access
     modes (named mmap for cross-process sharing, anonymous mmap for
     `FileAccessMode.Memory` and stream construction)

2. **Reader Layer** - Database navigation and IP lookup
   - `Reader`: Main entry point for IP address lookups
   - Performs binary search tree traversal to locate data for an IP address
   - Supports three `FileAccessMode` options: `MemoryMapped`,
     `MemoryMappedGlobal`, `Memory`

3. **Decoder Layer** - Binary format deserialization
   - `Decoder`: Converts binary MaxMind DB format to .NET objects
   - `TypeActivatorCreator`: Compiles LINQ expression trees for fast object
     instantiation
   - `DictionaryActivatorCreator`, `ListActivatorCreator`: Specialized
     activators for collections

### MaxMind DB Binary Format

**File Structure:**

1. **Search Tree Section**: Binary search tree at beginning of file
2. **Data Section**: Deduplicated data records (multiple tree nodes can point to
   same data)
3. **Metadata Section**: Database metadata at end of file with magic marker
   (0xAB 0xCD 0xEF + "MaxMind.com")

**IP Lookup Process:**

1. Parse IP address (IPv4 = 32 bits, IPv6 = 128 bits)
2. Traverse binary search tree bit-by-bit from most significant to least
   significant
3. Each node contains two pointers (0 branch and 1 branch)
4. When data pointer found, resolve to data section and decode

**IPv4 Optimization:** For IPv4 lookups in IPv6 databases, the Reader
pre-calculates the IPv4 start node, skipping the first 96 nodes (::0/96 prefix).

### Deserialization System

The library uses an attribute-based deserialization system that maps MaxMind DB
data to .NET types:

#### Four Key Attributes

1. **`[Constructor]`**: Marks the constructor to use for deserialization (one
   per class)

2. **`[MapKey("db_field_name")]`**: Maps database field to a constructor
   parameter or property
   - Supports `AlwaysCreate = true` to instantiate nested objects even when
     database field is missing
   - If no attribute, uses parameter/property name as database key

3. **`[Inject("injectable_name")]`**: Injects runtime values not in database
   - Pass values via `InjectableValues` dictionary to `Find<T>()`
   - Example: Inject the queried IP address into the result object

4. **`[Network]`**: Injects the network CIDR (prefix length + network address)
   for the matched IP

#### Example Model Classes

**Constructor-based activation** (explicit `[Constructor]` attribute):

```csharp
using MaxMind.Db;

public class AsnResponse
{
    [Constructor]
    public AsnResponse(
        [MapKey("autonomous_system_number")] long? autonomousSystemNumber = null,
        [MapKey("autonomous_system_organization")] string? autonomousSystemOrganization = null,
        [Inject("ip_address")] IPAddress? ipAddress = null,
        [Network] Network? network = null
    )
    {
        AutonomousSystemNumber = autonomousSystemNumber;
        AutonomousSystemOrganization = autonomousSystemOrganization;
        IpAddress = ipAddress;
        Network = network;
    }

    public long? AutonomousSystemNumber { get; }
    public string? AutonomousSystemOrganization { get; }
    public IPAddress? IpAddress { get; }
    public Network? Network { get; }
}
```

**Property-based activation** (no `[Constructor]`, uses parameterless ctor +
`init` properties):

```csharp
using MaxMind.Db;

public class AsnResponse
{
    [MapKey("autonomous_system_number")]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxmind/MaxMind-DB-Reader-dotnet](https://github.com/maxmind/MaxMind-DB-Reader-dotnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
