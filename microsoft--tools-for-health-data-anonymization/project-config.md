---
trigger: always_on
description: **Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**
---

# Tools for Health Data Anonymization

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

Tools for Health Data Anonymization is a .NET 8.0 project that provides anonymization capabilities for healthcare data, specifically FHIR and DICOM formats. The project includes command-line tools, core libraries, unit tests, functional tests, and Azure Data Factory pipeline components.

## Working Effectively

### Prerequisites and Environment Setup
- Install .NET 8.0 SDK (confirmed working version: .NET 8.0.118)
- The project uses NuGet packages exclusively from nuget.org
- Code analysis is enforced using StyleCop with custom rules (stylecop.json and CustomAnalysisRules.ruleset)

### Build and Test Process

#### FHIR Components
Navigate to the FHIR directory for all FHIR-related operations:
```bash
cd FHIR
```

**Bootstrap and build (NEVER CANCEL - Build takes ~3 seconds, set timeout to 120+ seconds):**
```bash
dotnet restore    # Takes ~2 seconds
dotnet build --configuration Release    # Takes ~3 seconds, NEVER CANCEL
```

**Run tests (NEVER CANCEL - Tests take ~32 seconds, set timeout to 300+ seconds):**
```bash
dotnet test --configuration Release --collect "Code coverage"    # Takes ~32 seconds, NEVER CANCEL
```

#### DICOM Components
Navigate to the DICOM directory for all DICOM-related operations:
```bash
cd DICOM
```

**Bootstrap and build (NEVER CANCEL - Build takes ~2 seconds, set timeout to 120+ seconds):**
```bash
dotnet restore    # Takes ~1 second
dotnet build --configuration Release    # Takes ~2 seconds, NEVER CANCEL
```

**Run tests (NEVER CANCEL - Tests take ~10 seconds, set timeout to 180+ seconds):**
```bash
dotnet test --configuration Release --collect "Code coverage"    # Takes ~10 seconds, NEVER CANCEL
```

### Command-Line Tools Usage

#### FHIR R4 Anonymizer
**Location:** `FHIR/src/Microsoft.Health.Fhir.Anonymizer.R4.CommandLineTool/bin/Release/net8.0/`
```bash
./Microsoft.Health.Fhir.Anonymizer.R4.CommandLineTool -i input_folder -o output_folder
```

#### FHIR STU3 Anonymizer
**Location:** `FHIR/src/Microsoft.Health.Fhir.Anonymizer.Stu3.CommandLineTool/bin/Release/net8.0/`
```bash
./Microsoft.Health.Fhir.Anonymizer.Stu3.CommandLineTool -i input_folder -o output_folder
```

#### DICOM Anonymizer
**Location:** `DICOM/src/Microsoft.Health.Dicom.Anonymizer.CommandLineTool/bin/Release/net8.0/`
```bash
./Microsoft.Health.Dicom.Anonymizer.CommandLineTool -I input_folder -O output_folder
```

## Validation and Testing

### Manual Validation Requirements
**ALWAYS manually validate changes by running the appropriate anonymizer on sample data:**

**For FHIR changes:**
```bash
cd FHIR/src/Microsoft.Health.Fhir.Anonymizer.R4.CommandLineTool/bin/Release/net8.0/
mkdir -p /tmp/fhir-test-output
./Microsoft.Health.Fhir.Anonymizer.R4.CommandLineTool -i ../../../../../samples/fhir-r4-files -o /tmp/fhir-test-output -v
# Verify output files are created and contain anonymized data
ls -la /tmp/fhir-test-output/
```

**For DICOM changes:**
```bash
cd DICOM/src/Microsoft.Health.Dicom.Anonymizer.CommandLineTool/bin/Release/net8.0/
mkdir -p /tmp/dicom-test-output
./Microsoft.Health.Dicom.Anonymizer.CommandLineTool -I ../../../../../samples -O /tmp/dicom-test-output
# Verify output files are created
ls -la /tmp/dicom-test-output/
```

### Known Test Issues
- Some unit tests fail due to missing test configuration files (expected behavior)
- Some functional tests fail due to missing configuration files (expected behavior)
- Azure Data Factory pipeline tests are skipped (expected behavior)
- Code coverage collection only works on Windows (expected on Linux)
- **DO NOT** attempt to fix these test failures unless specifically requested

### Pre-commit Validation
Always run these commands before committing changes:
```bash
cd FHIR && dotnet build --configuration Release
cd ../DICOM && dotnet build --configuration Release
```

## Project Structure and Key Locations

### FHIR Components
- **Core Libraries:** `FHIR/src/Microsoft.Health.Fhir.Anonymizer.{R4|Stu3}.Core/`
- **Command-Line Tools:** `FHIR/src/Microsoft.Health.Fhir.Anonymizer.{R4|Stu3}.CommandLineTool/`
- **Unit Tests:** `FHIR/src/Microsoft.Health.Fhir.Anonymizer.{R4|Stu3}.Core.UnitTests/`
- **Functional Tests:** `FHIR/src/Microsoft.Health.Fhir.Anonymizer.{R4|Stu3}.FunctionalTests/`
- **Azure Data Factory:** `FHIR/src/Microsoft.Health.Fhir.Anonymizer.{R4|Stu3}.AzureDataFactoryPipeline/`
- **Shared Code:** `FHIR/src/Microsoft.Health.Fhir.Anonymizer.Shared.*/`
- **Sample Files:** `FHIR/samples/fhir-r4-files/` and `FHIR/samples/fhir-stu3-files/`
- **Configuration:** `configuration-sample.json` (copied to build outputs)

### DICOM Components
- **Core Library:** `DICOM/src/Microsoft.Health.Dicom.Anonymizer.Core/`
- **Command-Line Tool:** `DICOM/src/Microsoft.Health.Dicom.Anonymizer.CommandLineTool/`
- **Common Library:** `DICOM/src/Microsoft.Health.Anonymizer.Common/`
- **Unit Tests:** `DICOM/src/Microsoft.Health.Dicom.Anonymizer.*.UnitTests/`
- **Sample Files:** `DICOM/samples/` (I290.dcm, I341.dcm, lung.dcm)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/Tools-for-Health-Data-Anonymization](https://github.com/microsoft/Tools-for-Health-Data-Anonymization) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
