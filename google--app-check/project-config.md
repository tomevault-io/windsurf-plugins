---
trigger: always_on
description: The goal of this document is to ensure high-quality, reproducible, and
---

# App Check - Agent Workflow Instructions

The goal of this document is to ensure high-quality, reproducible, and
verifiable contributions in a fully autonomous loop for the App Check
repository.

---

## 📥 Input Requirements

Before starting any work, the agent must require or acquire:
1.  **Feature Specification**: A detailed description of the feature, bug, or
    task.
2.  **Project Configuration**: Access to necessary credentials or configurations
    if applicable.
3.  **External Scripts**: Access to the `firebase-ios-sdk` scripts. If not using
    the cloned scripts via `./setup-scripts.sh`, ensure they are available in a
    local clone of `firebase-ios-sdk` (commonly at
    `<path_to_firebase_ios_sdk>/scripts` but path may vary). If the path is not
    found, ask the human for it.

## 📤 Output Requirements

A successful task completion MUST produce:
1.  **Code Changes**: The implemented feature or fix and corresponding tests.
2.  **Unit & Integration Tests**: Demonstrating success and handling edge cases.
3.  **Implementation Plan** (For complex tasks only): A scannable proposal
    before starting work.
4.  **Walkthrough Artifact**: A summary containing verification results and
    reproduction snippets.

---

## 💬 Communication Guidelines

When reporting back to the user, prioritize scannability and clarity:
1.  **Use Categorized Bullet Points**: Group findings and results into clear
    categories (e.g., "Build & Test Results", "Code Changes").
2.  **Use Indicators**: Prefix status updates with checkmarks (✅) or caution
    symbols (⚠️) for immediate visual parsing.
3.  **Be Concise**: Avoid conversational filler. Get straight to the results
    and next steps.
4.  **Final Report**: Conclude the task with a concise summary of work and a
    recommended conventional commit message.

---

## 🔄 The Agentic Loop: Step-by-Step

### Step 0: Workflow Selection & Planning (Hybrid Approach)
- **Prerequisite**: Verify that external scripts are accessible or that
  `./setup-scripts.sh` has been run to link them. If you cannot find them, ask
  the human for the path to the `firebase-ios-sdk` repository.
- **Action**: Assess the complexity of the task.
    - **Simple Task**: Proceed directly to **Step 1: TDD**.
    - **Complex Task**: Create a highly scannable **Implementation Plan** and
      get human approval.
- **Plan Requirements (Highly Scannable)**:
    - Keep it brief and hit key points.
    - Use bullet points for readability.
    - Focus on *what* changes and *why*, avoiding detailed *how*.
    - Highlight any open questions or design decisions requiring human input.

### Step 1: Test-Driven Development (TDD)
- **Constraint**: You MUST write tests before writing implementation code.
- **Action**:
    1. Create or identify the correct test target in `Package.swift`. Keep
       Swift and Obj-C test targets separate.
    2. Write a failing unit or integration test asserting the new behavior.
    3. Verify it fails by running the appropriate test command (e.g., `swift
       test --filter <TargetName>`).

### Step 2: Implementation
- Implement the feature or fix.
- Follow project conventions and guidelines if available.

### Step 3: Verification
- **Action**: Run tests using the cloned scripts or by referencing the external
  ones (e.g., in `<path_to_firebase_ios_sdk>`).
- **Iteration Workflow**: To get into a faster iterative loop, use the external
  scripts directly if possible. Set an environment variable like
  `FIREBASE_IOS_SDK_PATH` if your path differs from the default
  `<path_to_firebase_ios_sdk>`.
  - To bypass the CI secret check in `check_secrets.sh` when running external
    scripts in a trusted environment, export `FIREBASECI_IS_TRUSTED_ENV="true"`.
- **Commands**:
    - **Primary (Fast Iteration)**: For SPM testing (which uses `xcodebuild`
      under the hood):
      `${FIREBASE_IOS_SDK_PATH:-<path_to_firebase_ios_sdk>}/scripts/build.sh AppCheck <platform> spm`
      (where `<platform>` is `iOS`, `tvOS`, `macOS`, or `catalyst`).
    - For CocoaPods linting:
      `${FIREBASE_IOS_SDK_PATH:-<path_to_firebase_ios_sdk>}/scripts/pod_lib_lint.rb AppCheckCore.podspec --platforms=ios`
      (or other platforms: `tvos`, `macos --skip-tests`, `watchos`).
    - Alternatively, run `./setup-scripts.sh` to clone scripts locally and use
      `scripts/pod_lib_lint.rb`.
    - For Catalyst testing:
      `${FIREBASE_IOS_SDK_PATH:-<path_to_firebase_ios_sdk>}/scripts/test_catalyst.sh AppCheckCore test`.
- **xcodebuild Iteration**: For direct `xcodebuild` invocations, follow the
  order: `build`, `build-for-testing`, then `test`. This allows for faster
  iteration.

### Step 4: Public API Visibility
- **Requirement**: Identify and report any new public APIs created.
- **Method**: Check for changes in public headers or symbols.

### Step 5: Style Application
- **Action**: You MUST run `<path_to_firebase_ios_sdk>/scripts/style.sh` to
  maintain consistency.
- **Constraint**: Since style changes are non-functional, you do NOT need to
  re-run tests after applying style fixes.

### Step 6: Documentation Formatting
- **Requirement**: Wrap all documentation files (like `agents.md`) to be 80
  characters or less (excluding code blocks). Remove all trailing whitespace.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [google/app-check](https://github.com/google/app-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
