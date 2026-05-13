---
trigger: always_on
description: - **MANDATORY .NET 9.0 SDK**: All local development must use .NET 9.0.303 or later
---

# # GitHub Copilot Guidelines

## .NET 9.0 Local Development Environment Enforcement (MANDATORY)

### Rule 1: .NET 9.0 Environment Requirements (CRITICAL)
- **MANDATORY .NET 9.0 SDK**: All local development must use .NET 9.0.303 or later
- **Before submitting any GitHub workflow or PR**, developers MUST verify:
  - Local environment has .NET 9.0 SDK installed (`dotnet --version` returns 9.0.x)
  - Aspire workload is installed and functional
  - All solutions build successfully locally with .NET 9.0
  - LocalTesting workflow executes successfully locally
- **Local environment setup requirements**:
  - .NET 9.0 SDK installation using official Microsoft installer
  - Aspire workload installation (`dotnet workload install aspire`)
  - Docker Desktop or Podman running for Aspire orchestration
  - LocalTesting solution builds and runs without errors
- **GitHub workflow local validation**:
  - ALL GitHub workflows must pass locally before submission for review
  - No version compatibility issues between local and CI environments
  - LocalTesting workflow must execute successfully with Aspire dashboard accessible
  - Integration tests must pass locally with same results as CI
- **Environment consistency enforcement**:
  - Local development environment must match CI environment (.NET 9.0)
  - global.json version must be respected locally
  - No .NET version downgrades or workarounds permitted
  - Aspire orchestration must work locally before CI submission
- **Verification commands required before PR submission**:
  ```bash
  # Verify .NET version
  dotnet --version  # Must return 9.0.x
  
  # Install Aspire workload
  dotnet workload install aspire
  
  # Build all solutions
  dotnet build FlinkDotNet/FlinkDotNet.sln --configuration Release
  dotnet build Sample/Sample.sln --configuration Release  
  dotnet build LocalTesting/LocalTesting.sln --configuration Release
  
  # Test LocalTesting workflow
  ./test-aspire-localtesting.ps1 -MessageCount 1000
  ```
- **Installation verification for new developers**:
  ```bash
  # Check if .NET 9.0 is installed
  dotnet --list-sdks | grep "9.0"
  
  # If not installed, download and install .NET 9.0 SDK
  # Windows: Download from https://dotnet.microsoft.com/download/dotnet/9.0
  # Linux/macOS: Use the dotnet-install script
  curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --version latest --channel 9.0
  
  # Install Aspire workload
  dotnet workload install aspire
  
  # Verify installation
  dotnet --version  # Should show 9.0.x
  ```
- **Project file enforcement**:
  - All new .csproj files MUST target `net9.0` framework
  - Existing projects should be updated to .NET 9.0 when modified
  - global.json MUST specify .NET 9.0 SDK version
  - No mixed framework targeting (e.g., net8.0 and net9.0 in same solution)
- **Troubleshooting common issues**:
  - If `dotnet --version` shows 8.x, ensure .NET 9.0 is installed and PATH is updated
  - If Aspire workload fails to install, update to latest .NET 9.0 version first
  - If LocalTesting fails, verify Docker Desktop or Podman is running and has sufficient resources
  - If build errors occur, clean and rebuild: `dotnet clean && dotnet build`
- **Failure to verify .NET 9.0 environment is a MAJOR violation** requiring complete environment setup before work can proceed
- **Automated environment verification**:
  - Add .NET version check to all build scripts
  - Include environment validation in PR templates
  - Require .NET 9.0 confirmation in issue templates
  - Document environment setup in CONTRIBUTING.md

This document defines the coding standards and best practices that GitHub Copilot should enforce during code reviews for this .NET project. These guidelines ensure adherence to SOLID principles and .NET best practices, with specialized guidance for BizTalk to Inobiz migrations using .NET 9 and direct XSLT mapping.

## SOLID Principles Enforcement

### Single Responsibility Principle (SRP)
- **Rule**: Each class should have only one reason to change
- **Enforcement**:
  - Flag classes with more than 3 public methods doing unrelated tasks
  - Identify methods longer than 20 lines that handle multiple concerns
  - Suggest splitting classes that handle both business logic and infrastructure concerns (e.g., data access + business logic)
  - Recommend separating UI logic from business logic
  - Flag constructors that perform multiple initialization tasks

**Example Violations to Flag:**
```csharp
// BAD: Class doing too many things
public class UserManager
{
    public void ValidateUser() { }      // Validation
    public void SaveToDatabase() { }    // Data access
    public void SendEmail() { }         // Communication
    public void LogActivity() { }       // Logging
    public void GenerateReport() { }    // Reporting
}
```

**Suggested Fix:**
```csharp
// GOOD: Separated responsibilities
public class UserValidator { }
public class UserRepository { }
public class EmailService { }
public class ActivityLogger { }
public class ReportGenerator { }
```

### Open/Closed Principle (OCP)
- **Rule**: Classes should be open for extension but closed for modification
- **Enforcement**:
  - Flag switch statements or long if-else chains that would require modification to add new types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devstress/FlinkDotnet](https://github.com/devstress/FlinkDotnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
