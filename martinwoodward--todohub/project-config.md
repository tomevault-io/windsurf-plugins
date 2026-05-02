---
trigger: always_on
description: TodoHub is a native iOS application that uses GitHub Issues as a todo list backend. Users authenticate via GitHub OAuth, select/create a repository, and manage todos that are stored as GitHub Issues with GitHub Projects v2 for ordering and custom fields.
---

# GitHub Copilot Custom Instructions for TodoHub

## Project Overview

TodoHub is a native iOS application that uses GitHub Issues as a todo list backend. Users authenticate via GitHub OAuth, select/create a repository, and manage todos that are stored as GitHub Issues with GitHub Projects v2 for ordering and custom fields.

## Tech Stack

- **Language:** Swift 5.9
- **UI Framework:** SwiftUI (iOS 17+)
- **Architecture:** MVVM (Model-View-ViewModel)
- **Project Generation:** XcodeGen (project.yml → TodoHub.xcodeproj)
- **Authentication:** OAuth 2.0 with PKCE via AppAuth-iOS
- **API:** GitHub GraphQL API (NOT REST)
- **Storage:** GitHub Issues + GitHub Projects v2
- **CI/CD:** GitHub Actions + Fastlane (deployment only)

## Code Review Priorities

When reviewing code, prioritize:

1. **Security** - OAuth credentials, token storage, API security
2. **App Store Compliance** - Follow Apple App Store Review Guidelines (see `instructions/apple-appstore-reviewer.instructions.md`)
3. **Swift Concurrency** - Proper use of async/await, actor isolation, Sendable
4. **MVVM Pattern** - Clear separation between Views, ViewModels, and Services
5. **GraphQL Usage** - Efficient queries, proper error handling
6. **iOS 17+ Features** - Modern SwiftUI patterns, @Observable macro

## App Store Compliance

This app targets the Apple App Store. When reviewing or generating code, always consider:

### Privacy & Permissions
- All permission usage strings (`NS*UsageDescription`) must be clear and specific
- Never request permissions at launch without justification
- Privacy manifest (`PrivacyInfo.xcprivacy`) must accurately reflect data collection

### Authentication
- GitHub OAuth is the only authentication method (no Apple Sign-In requirement)
- Account deletion must be accessible if account creation exists
- Clear explanation of why account is required

### Technical Quality
- Handle network errors gracefully with user-friendly messages
- Provide meaningful empty states (no blank screens)
- Support offline scenarios where appropriate
- No crashes or dead-end states

### Reviewer Experience
- First-run experience should clearly show app purpose
- Core features accessible without complex setup
- Onboarding should explain key features

For detailed App Store review guidance, see `instructions/apple-appstore-reviewer.instructions.md`

## Critical Conventions

### Swift/SwiftUI
- ✅ Use `@Observable` macro for ViewModels (iOS 17+)
- ✅ Use `@MainActor` for UI-bound classes
- ✅ Prefer `async/await` over completion handlers
- ✅ Use `Sendable` for thread-safe types
- ❌ Avoid force unwrapping (`!`) unless absolutely safe
- ❌ Don't use Combine or older state management patterns

### Architecture
- **ViewModels**: Business logic, state management, marked with `@Observable` and `@MainActor`
- **Views**: UI only, lightweight, use `@State` for local state and `@Bindable` for ViewModel binding
- **Services**: Stateless, thread-safe, handle external APIs
- **Models**: Value types (structs), conform to `Identifiable`, `Codable`, `Sendable`, `Hashable`

### Naming Conventions
- ViewModels: `*ViewModel` (e.g., `TodoListViewModel`, `AuthViewModel`)
- Views: Descriptive names (e.g., `TodoRowView`, `QuickAddView`, `LoginView`)
- Services: `*Service` (e.g., `GitHubAPIService`, `KeychainService`)
- Row/cell views: `*RowView` suffix

### Error Handling
- Use `APIError` enum for all API errors
- Display user-friendly error messages via alerts
- Log errors for debugging with clear context
- Never expose raw error messages to users

### GitHub API
- ✅ Use GraphQL for ALL GitHub API calls (not REST)
- ✅ Queries defined in `GraphQL/` directory
- ✅ Token stored securely in Keychain via `KeychainService`
- ✅ Handle rate limiting and network errors gracefully

## Project Structure

```
TodoHub/
├── App/                    # App entry point (TodoHubApp.swift)
├── Config/                 # OAuth credentials (Config.swift - GITIGNORED)
├── Models/                 # Data models (Todo, User, Repository, Priority)
├── Views/                  # SwiftUI views organized by feature
│   ├── Login/             # GitHub OAuth login
│   ├── Setup/             # Repository selection
│   ├── TodoList/          # Main todo list interface
│   ├── TodoDetail/        # Todo editing screen
│   ├── AllIssues/         # Cross-repository issues view
│   ├── Settings/          # App settings
│   └── Components/        # Reusable UI components
├── ViewModels/            # Business logic and state
├── Services/              # API, Auth, Keychain services
├── Extensions/            # Swift extensions
├── GraphQL/               # Query definitions
└── Resources/             # Assets
```

## Key Files and Their Purpose

| File | Purpose | Edit Carefully |
|------|---------|----------------|
| `project.yml` | XcodeGen config - edit this, NOT .xcodeproj | ⚠️ Yes |
| `Config.swift` | OAuth credentials (gitignored) | 🔒 Never commit |
| `Config.swift.template` | Template for OAuth credentials | ✅ Safe |
| `GitHubAuthService.swift` | OAuth flow implementation | 🔒 Security critical |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martinwoodward/todohub](https://github.com/martinwoodward/todohub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
