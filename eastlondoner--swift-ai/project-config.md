---
trigger: always_on
description: Okay, here are detailed instructions to create, build, test, and publish a Swift package.
---

Okay, here are detailed instructions to create, build, test, and publish a Swift package.

I. Package and Module Setup

Package.swift contains the package's name, targets, dependencies etc.

If you're having compilation or module not found lint errors, particularly in test files, run `swift build` to ensure that the built interfaces are up to date.

In general in Swift you do not need to have explicit import statements for files in the same module. This means that during refactoring you can move code and files around without having to update import statements.

II. Adding Dependencies

Identify Required Dependencies:

Use llm (or other search tool) to figure out what is needed to add firebase as a dependency, for example using llm "how to add firebase as a swift package dependency?"

Extract the relevant information, including dependency name and version, from the llm output.

Modify Package.swift:

Use a text editor (e.g. echo 'text' > file, sed -i 's/old_text/new_text/g' file) to modify the Package.swift file.

Add the dependency to the dependencies section, for example like this:

dependencies: [
    .package(url: "https://github.com/firebase/firebase-ios-sdk.git", from: "10.0.0")
    ],
content_copy
download
Use code with caution.
Swift

Add the dependency to the relevant target in the targets section, for example like this:

targets: [
   .target(
       name: "<package_name>",
       dependencies: [
          .product(name: "FirebaseAnalytics", package: "firebase-ios-sdk"),
       ]),
  .testTarget(
      name: "<package_name>Tests",
      dependencies: ["<package_name>"]),
content_copy
download
Use code with caution.
Swift

]
```

Use cat Package.swift again to verify that the changes were made.

III. Writing Swift Code

Create Source Files:

Navigate to the Sources/<package_name> directory.

Create Swift files for your code using touch <filename>.swift.

Use echo 'code' > <filename>.swift to add content to the files.

Write modular, well-documented code.

Implement Functionality:

Write the desired functionality in the created Swift files.

Consider using common design patterns like singletons, strategies, etc.

Write Documentation Comments:

Add Swift documentation comments (/// for single-line and /** ... */ for multi-line) to all public methods, properties, and classes. This is crucial for the README and developer understanding.

IV. Compilation

Run swift build:

Execute swift build from the root directory of the package.

Redirect standard error to a file for analysis swift build 2> build_error.log

Analyze Compilation Output:

Check if there are any compilation failures by checking the return code of swift build.

If there were errors, use grep "error:" build_error.log to find them.

If errors are found, report the output of grep "error:" build_error.log using llm, and retry the compilation step after the appropriate changes to code are made to fix compilation errors.

V. Unit Tests


Write thorough tests that cover all important aspects of the package's functionality.

Write Test Cases:

Write test cases inside the test file, focusing on testing each public method.

Use XCTAssert methods to verify the correctness of the functionality.

Run Tests:

Execute swift test from the root directory of the package. Detailed instructions for testing and coverage are in TESTING.md The most common commands are:
- Run tests: `swift test`
- Generate coverage reports: `./generate_coverage.sh` (creates both text and HTML reports in coverage_report/)

Analyze Test Output:

Check if there are any test failures using the return code of swift test.

If there were errors, use grep "fail" test_error.log to find them.

If failures are found, report the output of grep "fail" test_error.log using llm and adjust the code and/or the tests and retry the test step.

If there are warnings, examine the output of grep "warning:" test_error.log, and report to llm to see if there is any action to be taken.

VI. Linting

Run SwiftLint after making code changes and fix any issues:

To write lint output to a file, run swiftlint --lint --reporter compact > lint_report.log in the root directory of the package.

VII. README.md

Keep the README.md file up to date with the latest information about the package. In particular, it should include:

Installation: Instructions for adding the package as a dependency in Xcode using Swift Package Manager. Example:

Dependencies: List the dependencies of your package.

Requirements: Specify the minimum versions of iOS or macOS supported.

Usage: Code examples demonstrating how to use the package's core functionalities. Include multiple examples and add comments to help developers understand what the code does.

Documentation: Include a relative link to generated documentation

Generate Documentation:

Use swift package generate-documentation to generate a documentation archive.

Check the output of this program and fix any failures.

VIII. Packaging and Publishing

Version Control:

Initialize a Git repository using git init

Add all files using git add .

Commit the changes git commit -m "Initial commit"

Tag the version using git tag -a v0.1.0 -m "Initial release".

Push to the remote repository git push origin main && git push origin --tags


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eastlondoner/swift-ai](https://github.com/eastlondoner/swift-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-07 -->
