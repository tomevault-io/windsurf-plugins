---
trigger: always_on
description: Guidelines for automatically committing changes made by CursorAI using conventional commits format for Swift projects and all related assets
---


# Git Conventional Commits for Swift Projects

This rule establishes a framework for automatically generating Git commits in the conventional commits format for Swift projects, including all configuration files and asset types.

## Conventional Commits Format

All commits should follow the conventional commits format:

```
<type>(<scope>): <description>
```

Where:
- **type**: Indicates the kind of change being made
- **scope**: Optional field indicating the section of the codebase affected
- **description**: Brief description of the change in imperative mood

## Swift-Specific Types and Scopes

### Commit Types

- **feat**: A new feature or functionality
- **fix**: A bug fix
- **refactor**: Code change that neither fixes a bug nor adds a feature
- **docs**: Documentation only changes
- **style**: Changes that do not affect code logic (whitespace, formatting, etc.)
- **test**: Adding or correcting tests
- **perf**: Performance improvements
- **chore**: Changes to build process or auxiliary tools
- **ui**: UI/UX improvements (specific to Swift UI components)
- **config**: Changes to configuration files
- **build**: Changes to Xcode build settings, project files, or build scripts
- **i18n**: Internationalization and localization changes
- **assets**: Adding or updating media and other resource files

### Scope Conventions for Swift Projects

- **app**: App-level changes (e.g., AppDelegate, SceneDelegate)
- **view**: View layer changes (UIView, SwiftUI View components)
- **controller**: View controller changes
- **presenter**: Presenter layer (VIPER/MVP)
- **interactor**: Interactor layer (VIPER)
- **entity**: Model/entity changes
- **router**: Navigation/routing logic
- **networking**: API/networking code
- **persistence**: CoreData or other persistence mechanisms
- **utils**: Utility functions
- **extensions**: Swift extensions
- **config**: Configuration changes
- **project**: Xcode project settings
- **assets**: Images, colors, and other resources
- **i18n**: Localization files
- **deps**: Dependencies and packages
- **media**: Audio, video, and other media files
- **fonts**: Typography assets
- **docs**: Documentation files

## Automated Commit Rule

<rule>
name: swift_conventional_commits
description: Automatically commit changes made by CursorAI using conventional commits format for Swift projects and all related assets
filters:
  - type: event
    pattern: "build_success"
  - type: file_change
    pattern: "*"
actions:
  - type: execute
    command: |
      # Extract the change type and scope from the changes and description
      CHANGE_TYPE=""
      
      # Determine type based on what was changed
      case "$CHANGE_DESCRIPTION" in
        *"add"*|*"create"*|*"implement"*|*"new feature"*)
          CHANGE_TYPE="feat";;
        *"fix"*|*"correct"*|*"resolve"*|*"bug"*)
          CHANGE_TYPE="fix";;
        *"refactor"*|*"restructure"*|*"reorganize"*)
          CHANGE_TYPE="refactor";;
        *"test"*|*"unit test"*|*"ui test"*)
          CHANGE_TYPE="test";;
        *"doc"*|*"comment"*|*"documentation"*|*"readme"*)
          CHANGE_TYPE="docs";;
        *"style"*|*"format"*|*"whitespace"*|*"indent"*)
          CHANGE_TYPE="style";;
        *"perf"*|*"optimize"*|*"performance"*)
          CHANGE_TYPE="perf";;
        *"ui"*|*"interface"*|*"visual"*)
          CHANGE_TYPE="ui";;
        *"localize"*|*"translate"*|*"i18n"*)
          CHANGE_TYPE="i18n";;
        *"image"*|*"icon"*|*"graphic"*|*"asset"*)
          CHANGE_TYPE="assets";;
        *"audio"*|*"sound"*|*"music"*|*"video"*)
          CHANGE_TYPE="assets";;
        *)
          # Look at file extension to determine type for non-keyword cases
          if [[ "$FILE" == *".xcodeproj"* || "$FILE" == *".pbxproj"* || "$FILE" == *".xcworkspace"* ]]; then
            CHANGE_TYPE="build"
          elif [[ "$FILE" == *".plist"* || "$FILE" == *".xcconfig"* || "$FILE" == *".yml"* || "$FILE" == *".yaml"* || "$FILE" == *".json"* ]]; then
            CHANGE_TYPE="config"
          elif [[ "$FILE" == *".strings"* || "$FILE" == *".stringsdict"* ]]; then
            CHANGE_TYPE="i18n"
          elif [[ "$FILE" == *".png"* || "$FILE" == *".jpg"* || "$FILE" == *".jpeg"* || "$FILE" == *".gif"* || "$FILE" == *".svg"* || "$FILE" == *".pdf"* ]]; then
            CHANGE_TYPE="assets"
          elif [[ "$FILE" == *".mp3"* || "$FILE" == *".mp4"* || "$FILE" == *".mov"* || "$FILE" == *".wav"* ]]; then
            CHANGE_TYPE="assets"
          elif [[ "$FILE" == *".otf"* || "$FILE" == *".ttf"* || "$FILE" == *".woff"* ]]; then
            CHANGE_TYPE="assets"
          elif [[ "$FILE" == *".md"* || "$FILE" == *".txt"* || "$FILE" == *"README"* || "$FILE" == *"LICENSE"* ]]; then
            CHANGE_TYPE="docs"
          else
            CHANGE_TYPE="chore"
          fi
          ;;
      esac
      
      # Extract scope based on file type and location
      if [[ "$FILE" == *".swift" ]]; then
        # Swift files follow the standard Swift scoping
        if [[ "$FILE" == *"View.swift" || "$FILE" == *"/Views/"* || "$FILE" == *"/View/"* ]]; then
          SCOPE="view"
        elif [[ "$FILE" == *"ViewController.swift" || "$FILE" == *"/ViewControllers/"* ]]; then

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielraffel/SwiftCatalyst](https://github.com/danielraffel/SwiftCatalyst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
