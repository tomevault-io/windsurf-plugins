---
trigger: always_on
description: nBayes is a C# .NET 8.0 library implementing Paul Graham's naive Bayesian spam filter for statistical decision making and text classification. The library provides simple APIs for training indexes and categorizing text entries.
---

# nBayes - Naive Bayesian Classifier

nBayes is a C# .NET 8.0 library implementing Paul Graham's naive Bayesian spam filter for statistical decision making and text classification. The library provides simple APIs for training indexes and categorizing text entries.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Quick Start - Build, Test, and Validate
- **Bootstrap the repository:**
  - `dotnet restore` -- takes ~2-10 seconds (first time slower). Set timeout to 30+ seconds.
  - `dotnet build` -- takes ~5-10 seconds. Set timeout to 30+ seconds.
  - `dotnet test` -- takes ~4 seconds. Set timeout to 15+ seconds.

### Core Development Commands
- **Restore dependencies:** `dotnet restore`
- **Build the solution:** `dotnet build` 
- **Run all tests:** `dotnet test`
- **Build specific project:** `dotnet build nBayes/nBayes.csproj`
- **Run sample (has limitations - see below):** `dotnet run --project nBayes.Sample`

### Key Projects Structure
- **nBayes/** - Core library with Bayesian classifier implementation
- **nBayes.Sample/** - Sample console application (has Twitter API dependency issues)
- **nBayes.test/** - NUnit test suite (2 tests, both pass reliably)

## Validation

### Always Run These Tests After Changes
1. **Build validation:** `dotnet build` - must complete without errors
2. **Unit tests:** `dotnet test` - both tests must pass
3. **Core functionality validation:** Create a simple test program to verify classification works:

```csharp
// Create simple test to validate core functionality
using System;
using nBayes;

class Program 
{
    static void Main()
    {
        var spam = nBayes.Index.CreateMemoryIndex();
        var notSpam = nBayes.Index.CreateMemoryIndex();
        
        spam.Add(Entry.FromString("want some viagra?"));
        notSpam.Add(Entry.FromString("Hello, how are you?"));
        
        var analyzer = new Analyzer();
        var result = analyzer.Categorize(Entry.FromString("viagra"), spam, notSpam);
        
        Console.WriteLine($"Classification result: {result}");
        // Should output: Classification result: First
    }
}
```

### CRITICAL Testing Scenarios
- **ALWAYS test classification accuracy** by training with known spam/not-spam examples and verifying correct categorization
- **Test with MemoryIndex and FileIndex** to ensure both storage types work
- **Verify Entry.FromString()** correctly tokenizes text input
- **Test Analyzer.Categorize()** with various input combinations

#### Complete Functionality Test
```csharp
// Test both MemoryIndex and FileIndex
var memIndex = nBayes.Index.CreateMemoryIndex();
var fileIndex = new FileIndex("/tmp/test.xml");
fileIndex.Open();

// Both should support Add/GetTokenCount operations
memIndex.Add(Entry.FromString("memory test"));
fileIndex.Add(Entry.FromString("file test"));

Console.WriteLine($"Memory index entries: {memIndex.EntryCount}");
Console.WriteLine($"File index entries: {fileIndex.EntryCount}");

fileIndex.Save(); // FileIndex requires explicit save

// Test Entry tokenization (requires System.Linq)
using System.Linq;
var entry = Entry.FromString("hello world test");
var tokens = entry.ToList(); // Entry implements IEnumerable<string>
Console.WriteLine($"Tokens: {string.Join(", ", tokens)}");
```

## Common Issues and Limitations

### Sample Application Issues
- **Sample app now uses Reddit RSS API** - The sample app has been updated to use `https://www.reddit.com/search.rss?q=QUERY&sort=new` instead of the deprecated Twitter API
- **Internet access required** - The sample app requires internet access to fetch Reddit RSS feeds for training data
- **Graceful fallback** - When internet access is unavailable (like in sandboxed environments), the app creates sample data for testing
- Use manual testing or unit tests instead of the sample app to validate core library changes

### Build Warnings
- **No obsolete WebRequest warnings** - The sample app has been updated to use modern HttpClient instead of obsolete WebRequest
- All builds should complete without warnings

### Namespace Conflicts
- When using `Index` class, specify `nBayes.Index` to avoid conflict with `System.Index` in .NET
- When testing `Entry` tokenization, add `using System.Linq;` to use `.ToList()` on Entry objects

## Development Workflow

### For Library Changes
1. Make changes to files in `nBayes/` directory
2. Run `dotnet build` to verify compilation
3. Run `dotnet test` to ensure tests pass  
4. Create manual test program to validate new functionality
5. Update unit tests in `nBayes.test/` if adding new features

### For Adding Tests
- Tests use NUnit framework
- Add new test methods to `nBayes.test/TestnBayes.cs`
- Follow existing test patterns with `[Test]` attribute and `Assert.That()` assertions

## Timing Expectations
- **NEVER CANCEL** any build operation - they complete quickly
- **dotnet restore:** ~2-10 seconds (first time slower, subsequent runs ~2s)
- **dotnet build:** ~5-10 seconds (clean builds take longer)  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joelmartinez/nBayes](https://github.com/joelmartinez/nBayes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
