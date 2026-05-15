---
trigger: always_on
description: You are assisting with enterprise PowerShell development that prioritizes security, maintainability, and performance. Always follow these foundational principles:
---

# PowerShell Copilot Instructions

## 🎯 Core Principles
You are assisting with enterprise PowerShell development that prioritizes security, maintainability, and performance. Always follow these foundational principles:

1. **Security by Design**: Implement comprehensive input validation and secure credential handling
2. **Correlation Tracking**: Include correlation IDs for audit trails and troubleshooting
3. **Error Resilience**: Implement robust error handling with structured logging
4. **Performance Awareness**: Write efficient code with appropriate optimization
5. **Community Standards**: Follow established PowerShell best practices and style guidelines
6. **Enterprise Integration**: Design for scalability and organizational compliance

## 📋 Community Standards Integration

### PowerShell Community Standards Compliance
All code generation must comply with established PowerShell community best practices and style guidelines, incorporating expert feedback for accuracy:

#### Best Practices Enforcement
- **Tool vs Controller Design**: Create reusable functions (tools) vs one-time automation scripts (controllers)
- **Modular Architecture**: Functions accept input via parameters, output to pipeline for maximum reusability
- **Error Handling Standards**: Use proper `$_` handling in catch blocks, appropriate null checking patterns
- **Performance Optimization**: Context-dependent string operations, test when it matters, use language features over cmdlets
- **Security by Design**: Modern credential handling with `[PSCredential]::new()`, validate all inputs, sanitize data

#### Style Guide Compliance
- **Code Layout**: One True Brace Style (opening brace at end of line), 4-space indentation, 115-character line limit
- **Function Structure**: Advanced functions with CmdletBinding, appropriate parameter validation (no redundant attributes)
- **Naming Conventions**: Approved PowerShell verbs only (use `Get-Verb`), full command names, explicit parameter names, PascalCase
- **Documentation Standards**: Proper comment-based help format with opening `<#`, no maintenance-heavy versioning in NOTES

#### Anti-Patterns to Avoid
- **Error Handling**: Don't use `$Error[0]` in catch blocks (use `$_`), don't force exceptions for simple existence checks
- **Throw Behavior**: Be aware that `throw` statements can be silenced by `-ErrorAction SilentlyContinue` - use `Write-Error -ErrorAction Stop` for proper termination when needed
- **Parameter Usage**: Don't pass parameters to functions without validating they have values, especially when parameters might be empty strings or whitespace
- **Performance**: Don't use StringBuilder for small string operations (< 100 concatenations), don't use array appending in large loops
- **Validation**: Don't add ValidateNotNullOrEmpty to mandatory parameters unless they'll be used in downstream function calls where empty strings matter
- **Output**: Don't use misleading `[OutputType([PSCustomObject])]` declarations
- **Documentation**: Don't use broken comment-based help (missing `<#` opener)

### Script Development Guidelines
- **Approved Verbs Only**: Use Microsoft's approved PowerShell verbs (`Get-Verb`) for ALL function names consistently
- **Version Compatibility**: Maintain compatibility with Windows PowerShell 5.1 unless specific PowerShell 7+ features are required
- **Version Annotations**: Document any PowerShell 7+ specific features with `#requires -version 7.0` at script header
- **Output Standards**: Avoid `Write-Host` except when colored console output is explicitly required
  - Use `Write-Verbose`, `Write-Debug`, `Write-Information`, `Write-Warning`, or `Write-Error` instead
  - Leverage pipeline output for data flow
- **Pipeline Efficiency**: Always design functions to work efficiently in the pipeline

### Modern PowerShell Features
- Use `[PSCredential]::new()` instead of `New-Object` for credential creation
- Leverage PowerShell 5.1+ features when available
- Avoid redundant parameter validation (mandatory parameters are implicitly not null/empty)

## 🛡️ Security Requirements

### Input Validation
```powershell
# Appropriate validation patterns
param(
    [Parameter(Mandatory)]
    [string]$ComputerName,  # Mandatory implies not null/empty

    [Parameter()]
    [ValidateNotNullOrEmpty()]  # Explicit validation for optional params
    [ValidatePattern('^[a-zA-Z0-9\-\.]+$')]
    [string]$OptionalServer
)

# Sanitize file paths to prevent traversal attacks
$safePath = Resolve-Path -Path $InputPath -ErrorAction Stop
if ($safePath.Path.Contains('..')) {
    throw "Invalid path: Path traversal not allowed"
}
```

### Credential Management
```powershell
# Modern credential creation (PowerShell 5+)
$credential = [PSCredential]::new($username, $securePassword)

# Always use secure credential handling per community standards
$secureString = ConvertTo-SecureString $Password -AsPlainText -Force
```

### Security Scanning Patterns
Avoid these patterns in production code:
- Hardcoded passwords or API keys
- Plain text credential storage
- SQL injection vulnerabilities in dynamic queries
- Unvalidated user input in file paths or commands

### Security Logging
```powershell

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fadwen/Powershell-Copilot-Standards](https://github.com/fadwen/Powershell-Copilot-Standards) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
