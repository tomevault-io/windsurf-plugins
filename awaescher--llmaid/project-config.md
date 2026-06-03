---
trigger: always_on
description: llmaid is a C# .NET 10.0 command-line tool that automates AI-supported file changes using large language models. It reads source code files, sends them to Ollama, LM Studio, or OpenAI-compatible APIs, and writes back the model's responses.
---

# AGENTS.md - Coding Agent Guidelines for llmaid

## Project Overview

llmaid is a C# .NET 10.0 command-line tool that automates AI-supported file changes using large language models. It reads source code files, sends them to Ollama, LM Studio, or OpenAI-compatible APIs, and writes back the model's responses.

## Build, Run, Test, and Lint Commands

### Build
```bash
dotnet build                           # Build entire solution
dotnet build llmaid/llmaid.csproj      # Build main project only
```

### Run
```bash
dotnet run --project llmaid -- --profile ./profiles/code-documenter.yaml
```

### Testing the app (CLI) locally

Each profile in `./profiles/` has a dedicated folder under `./testfiles/` with matching demo files.
Use the commands below to run a specific profile against its testfiles. Successful runs may show Git changes in the testfiles directory.

The main developer is using LM Studio and the LLM `qwen3.5-35b-a3b` these days. Testing the code-documenter profile should look like this.

```bash
dotnet run --project llmaid -- --profile ./profiles/code-documenter.yaml --targetPath "./testfiles/code" --provider lmstudio --uri http://localhost:1234/v1 --model mlx-community/qwen3.5-35b-a3b
```

The code-documenter profile is changing files (`applyCodeblock=true`) so in addition to checking the console output, watch expected changes with git diff.

**Using LM Studio (preferred):**
```bash
dotnet run --project llmaid -- --profile TEST-PROFILE-HERE --targetPath TESTFILES-FOLDER-HERE --provider lmstudio --uri http://localhost:1234/v1 --model MODEL-HERE --verbose
```

If the model is not available, query the models endpoint to find an available model:
```bash
curl http://localhost:1234/api/v1/models
```

**Using Ollama (fallback):**
```bash
dotnet run --project llmaid -- --profile TEST-PROFILE-HERE --targetPath TESTFILES-FOLDER-HERE --provider ollama --uri http://localhost:11434 --model MODEL-HERE --verbose
```

Query available models via:
```bash
curl http://localhost:11434/api/tags
```

#### Profile smoke tests

The table below lists the expected behavior for each profile when run against the provided demo files.
Use this to verify a profile works correctly after changes. `applyCodeblock: true` profiles write changes back to files (visible as Git diffs); `applyCodeblock: false` profiles print structured output to the console.

| Profile | Testfiles folder | applyCodeblock | Expected behavior |
|---------|-----------------|:-:|---|
| `code-documenter.yaml` | `./testfiles/code` | ✅ | Adds/fixes XML/JSDoc/docstring documentation on public members in all code files |
| `code-changer.yaml` | `./testfiles/code` | ✅ | Replaces `new List<T> {}` / `new T[] {}` with C# 12 collection expressions `[...]` |
| `code-changer-vb.yaml` | `./testfiles/code` | ✅ | Replaces `And`→`AndAlso` and `Or`→`OrElse` in VB.NET files (only without method calls) |
| `unprofessional-content-finder.yaml` | `./testfiles/code` | 📋 | Returns JSON findings for `cache.php` ("I freaking hate PHP"), `contract.sol` ("f*ckface", "libido"), `FileReader.cs` ("Wurstblinker"), `JumpWidget.cpp` ("kurva"); all others return `OK` |
| `unprofessional-content-fixer.yaml` | `./testfiles/code` | ✅ | Removes/replaces profanity and cringe comments; leaves clean files unchanged |
| `sensitive-data-marker.yaml` | `./testfiles/code` | ✅ | Adds `// TODO security review: sensitive data` comment to `JumpWidget.cpp` ("pa55wood!123") |
| `sql-injection-changer.yaml` | `./testfiles/code` | ✅ | Wraps unsafe SQL string concatenations with `SqlTools.MakeSqlValue()` in `SqlInjectionTests.vb` |
| `age-rater.yaml` | `./testfiles/age-rater` | 📋 | Returns YAML age ratings; `story-fsk0.txt` → FSK 0, `story-fsk12.txt` → FSK 12; images rated by visible content |
| `wiki-proofreader.yaml` | `./testfiles/code` | ✅ | Fixes spelling/grammar in `.md` and `.txt` files; leaves code blocks unchanged |
| `nda-checker.yaml` | *(provide an NDA text file)* | 📋 | Returns ✅/❌/⚠️ for each of the 15 company NDA rules with quoted evidence |
| `invoice-checker.yaml` | `./testfiles/invoice-checker` | 📋 | `invoice-correct.txt` → all 12 §14 UStG fields pass + calculations correct; `invoice-problematic.txt` → multiple ❌ (missing VAT ID, address, delivery date, bank details, rounding error) |
| `brand-detector.yaml` | `./testfiles/brand-detector` | 📋 | Returns YAML listing all visible brand logos/wordmarks per image with confidence and location |
| `meme-analyzer.yaml` | `./testfiles/meme-analyzer` | 📋 | Returns YAML per image with tone, content flags, and corporate suitability ratings for internal/external/customer-facing use |
| `image-alt-text-generator.yaml` | `./testfiles/alt-text-generator` | 📋 | Returns YAML with three alt text variants (short ≤125 chars, medium, long) plus visible text transcription per image |

### Test
```bash
# Run all tests
dotnet test

# Run all tests with verbose output
dotnet test --verbosity normal

# Run a single test by method name (partial match)
dotnet test --filter "Name=Ignores_Text_Outside_Of_The_Code_Block"

# Run a single test by fully qualified name

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awaescher/llmaid](https://github.com/awaescher/llmaid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
