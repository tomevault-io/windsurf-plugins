---
trigger: always_on
description: A Windows desktop application built with .NET 8.0 Windows Forms for managing Microsoft 365 user onboarding and offboarding operations. The application integrates with Microsoft Graph API to provide comprehensive user management capabilities.
---

# MS Cloud Ninja User Management Tool

A Windows desktop application built with .NET 8.0 Windows Forms for managing Microsoft 365 user onboarding and offboarding operations. The application integrates with Microsoft Graph API to provide comprehensive user management capabilities.

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Prerequisites and Environment Setup
- **CRITICAL**: This application **ONLY builds and runs on Windows**. The project uses `net8.0-windows` target framework with Windows Forms.
- Download and install .NET 8.0 SDK from: https://dotnet.microsoft.com/download/dotnet/8.0
- Windows 10 or later is required for optimal dark mode support
- Visual Studio 2022 or Visual Studio Code with C# extension recommended for development

### Build Commands
- `dotnet restore MSCloudNinjaGraphAPI.sln` -- restores NuGet packages. Takes ~8 seconds. Expected security warnings about Azure.Identity package vulnerabilities.
- `dotnet build MSCloudNinjaGraphAPI.sln --configuration Debug` -- builds in Debug mode. Takes ~10 seconds. NEVER CANCEL.
- `dotnet build MSCloudNinjaGraphAPI.sln --configuration Release` -- builds in Release mode. Takes ~10 seconds. NEVER CANCEL.
- Set timeout to 60+ seconds for build commands to account for package restoration and compilation.

### Running the Application
- **Debug**: `dotnet run --project MSCloudNinjaGraphAPI --configuration Debug`
- **Release**: Build first, then run the executable: `MSCloudNinjaGraphAPI\bin\Release\net8.0-windows\User Management Tool by MSCloudNinja.exe`
- The application requires Microsoft 365 admin credentials and appropriate Graph API permissions to function properly

### Testing and Validation
- **No automated tests exist** - this repository has no test projects or test infrastructure
- **Manual testing required**: Always test functionality manually after making changes
- **Critical validation scenarios**:
  1. **Authentication Flow**: Test sign-in with Microsoft 365 admin account
  2. **User Offboarding**: Select a test user and perform disable operations (without executing)
  3. **User Onboarding**: Navigate to onboarding tab and verify license loading
  4. **UI Responsiveness**: Verify dark theme loads correctly and navigation works

### Security Considerations
- **KNOWN VULNERABILITIES**: Azure.Identity package 1.10.4 has moderate severity vulnerabilities
- Always use test/development tenants when developing - never use production Microsoft 365 environments
- The application requires sensitive Microsoft Graph permissions (User.ReadWrite.All, Group.ReadWrite.All)

## Repository Structure

### Key Projects and Files
- **MSCloudNinjaGraphAPI.sln** - Main solution file (single project)
- **MSCloudNinjaGraphAPI/MSCloudNinjaGraphAPI.csproj** - Main project file targeting `net8.0-windows`
- **MSCloudNinjaGraphAPI/Program.cs** - Application entry point
- **MSCloudNinjaGraphAPI/Form1.cs** - Main form with authentication and navigation

### Important Directories
```
MSCloudNinjaGraphAPI/
├── Services/              # Business logic layer
│   ├── UserManagementService.cs  # Microsoft Graph API operations
│   └── LogService.cs             # Application logging
├── Controls/              # UI components
│   ├── UserOffboardingControl.cs # User offboarding interface
│   ├── UserOnboardingControl.cs  # User onboarding interface
│   └── GridControls.cs           # Data grid customizations
├── Models/                # Data models
│   └── License.cs               # License model for Graph API
└── Utils/                 # Utility classes
    └── ThemeColors.cs            # Dark theme color definitions
```

### Configuration Files
- **No external configuration files** - application uses hardcoded Graph API scopes and settings
- Logs are written to: `%LocalAppData%\MSCloudNinja\Logs\app_YYYYMMDD.log`

## Development Guidelines

### Build Environment Limitations
- **Linux/macOS**: Cannot build or run - will fail with "Microsoft.NET.Sdk.WindowsDesktop.targets not found"
- **Windows Containers**: May work but UI cannot be interacted with
- **GitHub Actions**: Must use `windows-latest` runners for any CI/CD workflows

### Code Quality
- No linters or code quality tools are configured - add these before implementing CI/CD
- No code formatting tools - recommend adding EditorConfig or running `dotnet format`
- Always maintain the existing dark theme color scheme when making UI changes

### Microsoft Graph Integration
- Application uses `InteractiveBrowserCredential` for authentication
- Required permissions: `User.ReadWrite.All`, `Group.ReadWrite.All`
- Always test Graph API calls with proper error handling
- The application queries both standard and beta Graph endpoints

### Common Development Tasks

#### Adding New Features
1. Add business logic to appropriate service class
2. Create or modify UI controls
3. Update the main form navigation if needed
4. Test manually with Microsoft 365 admin account

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OfirGavish/MSCloudNinjaUserManagementTool](https://github.com/OfirGavish/MSCloudNinjaUserManagementTool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
