---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SyslogLogging is a C# class library for logging to syslog, console, and file systems. The project targets multiple .NET versions including .NET Standard 2.0/2.1, .NET Framework 4.6.2/4.8, and .NET 8.0/10.0.

## Architecture

### Core Components

- **LoggingModule.cs**: Main class implementing `IDisposable` and `IAsyncDisposable`. Provides sync/async API for logging with background processing, structured logging, and persistent queuing.
- **LogEntry.cs**: Structured log entry with properties, correlation IDs, and metadata support.
- **PersistentLogQueue.cs**: File-backed queue system for reliable log message delivery that survives application restarts.
- **LogProcessingService.cs**: Background service handling batched processing of log entries with per-target queues.
- **SyslogLoggerProvider.cs**: Microsoft.Extensions.Logging integration provider.
- **SyslogExtensions.cs**: Extension methods for DI integration and fluent structured logging.
- **SyslogServer.cs**: Represents a syslog server configuration (hostname, port).
- **LoggingSettings.cs**: Configuration class containing formatting options, minimum severity levels, file logging modes, and color schemes.
- **Severity.cs**: Enumeration defining log levels (Debug, Info, Warn, Error, Alert, etc.).
- **ColorScheme.cs**: Defines console color configuration for different severity levels.
- **FileLoggingMode.cs**: Enumeration for file logging options (None, SingleLogFile, FileWithDate).

### Project Structure

- **LoggingModule/**: Main library project containing the logging implementation
- **Test.Shared/**: Shared Touchstone test descriptors and helpers
- **Test.Automated/**: Console runner for the shared Touchstone suites
- **Test.Xunit/**: xUnit adapter project for the shared Touchstone suites
- **Test.Nunit/**: NUnit adapter project for the shared Touchstone suites
- **assets/**: Contains logo files and branding assets

### Multi-targeting Support

The LoggingModule project targets:
- .NET Standard 2.0 and 2.1 (for broad compatibility)
- .NET Framework 4.6.2 and 4.8 (for legacy applications)
- .NET 8.0 and 10.0 (for modern applications)

## Key Features (v2.1.0)

### Async Support
- All logging methods have async variants with `CancellationToken` support
- Background processing with persistent queuing for high-performance logging
- Non-blocking message ingestion with reliable delivery

### Structured Logging
- `LogEntry` class for structured log data with properties and metadata
- JSON serialization support for structured data
- Correlation ID and source context tracking
- Fluent builder pattern for structured logging

### Microsoft.Extensions.Logging Integration
- `SyslogLoggerProvider` for seamless integration
- Extension methods for DI container registration
- Support for structured logging patterns from Microsoft's logging framework

### Persistent Queuing
- File-backed queues that survive application restarts
- Per-target queues (console, file, each syslog server)
- Configurable memory limits and file rotation
- Batch processing for I/O optimization

### Thread Safety & Performance
- Thread-safe console color handling with proper color reset
- Optimized UDP client management with connection reuse
- Background processing to minimize caller blocking
- Configurable batch sizes for optimal throughput

## Development Commands

### Building
```bash
# Build entire solution
dotnet build SyslogLogging.sln

# Build specific configuration
dotnet build SyslogLogging.sln -c Release

# Build for specific framework
dotnet build LoggingModule/LoggingModule.csproj -f netstandard2.0
```

### Testing
```bash
# Run the Touchstone CLI runner
dotnet run --project Test.Automated/Test.Automated.csproj -f net10.0

# Run xUnit and NUnit adapters
dotnet test Test.Xunit/Test.Xunit.csproj
dotnet test Test.Nunit/Test.Nunit.csproj
```

### Packaging
```bash
# Create NuGet package (already configured for automatic generation)
dotnet pack LoggingModule/LoggingModule.csproj

# Pack with specific configuration
dotnet pack LoggingModule/LoggingModule.csproj -c Release
```

## Key Usage Patterns

The library supports multiple initialization patterns:

1. **Default constructor**: Logs to localhost:514
2. **Single server**: Specify hostname and port
3. **Multiple servers**: Pass List<SyslogServer>
4. **File-only logging**: Provide filename in constructor
5. **Combined logging**: Configure syslog servers + console + file

### Configuration

LoggingModule.Settings provides extensive configuration:
- `MinimumSeverity`: Filter logs by severity level
- `FileLogging`: Set file logging mode (None/SingleLogFile/FileWithDate)
- `LogFilename`: Specify log file path
- `EnableColors`: Enable/disable console colors
- `HeaderFormat`: Customize log message format using variables like {ts}, {host}, {thread}, {sev}

## NuGet Package Configuration

The project is configured for automatic NuGet package generation with:
- Package ID: SyslogLogging
- Version: 2.1.0
- Multi-framework targeting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jchristn/LoggingModule](https://github.com/jchristn/LoggingModule) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
