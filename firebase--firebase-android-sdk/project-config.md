---
trigger: always_on
description: This guide provides essential information for working within the `firebase-android-sdk` repository.
---

# Agents

This guide provides essential information for working within the `firebase-android-sdk` repository.

## Project Overview

This repository contains the source code for the Firebase Android SDKs. It is a large, multi-module
Gradle project. The project is written in a mix of Java and Kotlin.

The project is structured as a collection of libraries, each representing a Firebase product or a
shared component. These libraries are published as Maven artifacts to Google's Maven Repository.

## Project Structure

The `subprojects.cfg` file lists all the subprojects in this repository. Each line in this file
follows the format `<project-path> # <project-type>`, where `project-type` can be one of the
following:

- `sdk`: A public-facing SDK that is published.
- `test`: A test application or a test-only module.
- `util`: A utility module that is not part of the public API.
- `directory`: A directory containing other subprojects.

This file is useful for understanding the role of each subproject in the repository.

## Environment Setup

To work with this repository, the Android SDK must be installed. Use the `sdkmanager` command-line
tool for this purpose.

1. **Install Java 17**
   - All SDKs require Java 17 to build and run. Earlier or later versions will not suffice.
     If multiple versions of java are installed, the `JAVA_HOME` environment variable can be set without modifying system configuration.
2. **Install Android SDK Command-Line Tools**:
   - If not already installed, download the command-line tools from the
     [Android Studio page](https://developer.android.com/studio#command-line-tools-only).
   - Create a directory for the Android SDK, e.g., `android_sdk`.
   - Unzip the downloaded package. This will create a `cmdline-tools` directory. Move this
     directory to `android_sdk/cmdline-tools/latest`.
   - The final structure should be `android_sdk/cmdline-tools/latest/`.
3. **Install required SDK packages**:
   - Use `sdkmanager` to install the necessary platforms, build tools, and other packages. For
     example:

     ```bash
     # List all available packages
     sdkmanager --list

     # Install platform tools and the SDK for API level 33
     sdkmanager "platform-tools" "platforms;android-33"

     # Accept all licenses
     yes | sdkmanager --licenses
     ```
   - Refer to the specific requirements of the project to determine which packages to install.
4. **Configure for integration tests**:
   - To run integration tests, a `google-services.json` file is required.
   - Place this file in the root of the repository.
5. **Install NDK for specific projects**:
   - Some projects, like `firebase-crashlytics-ndk`, require a specific version of the Android NDK.
     You can install it using `sdkmanager`. For example, to install NDK version 21.4.7075529, you
     would run `sdkmanager "ndk;21.4.7075529"`. Always refer to the project's `README.md` for the
     exact version required.

## Building and Running

The project is built using Gradle. The `gradlew` script is provided in the root directory.

### Building

To build the entire project, you can run the following command:

```bash
./gradlew build
```

To build a specific project, you can run:

```bash
./gradlew :<firebase-project>:build
```

### Running Tests

The project has three types of tests: unit tests, integration tests, and smoke tests.

#### Unit Tests

Unit tests run on the local JVM. They can be executed with the following command:

```bash
./gradlew :<firebase-project>:check
```

#### Running Specific Unit Tests

To run a specific test class or method, use the `--tests` filter with the appropriate test task (usually `testDebugUnitTest` for Android   
library modules, or `test` for Java library modules):

```bash
./gradlew :<firebase-project>:testDebugUnitTest --tests "com.google.firebase.package.ClassName"
./gradlew :<firebase-project>:testDebugUnitTest --tests "com.google.firebase.package.ClassName.methodName"
```

#### Integration Tests

Integration tests run on a hardware device or emulator. Before running integration tests, you need
to add a `google-services.json` file to the root of the project.

To run integration tests on a local emulator, use the following command:

```bash
./gradlew :<firebase-project>:connectedCheck
```

To run integration tests on Firebase Test Lab, use the following command:

```bash
./gradlew :<firebase-project>:deviceCheck
```

### Publishing

To publish a project locally, you can use the following command:

```bash
./gradlew -PprojectsToPublish="<firebase-project>" publishReleasingLibrariesToMavenLocal
```

By default, the `release` variant is published. To publish the `debug` variant instead, use the
`publishDebugVariant` property:

```bash
./gradlew -PprojectsToPublish="<firebase-project>" -PpublishDebugVariant=true publishReleasingLibrariesToMavenLocal
```

Using the `debug` variant has the advantage of including additional logging information. For example,
the `ai-logic/firebase-ai` SDK logs the request/response information when the debug variant is used.

## Development Conventions

### Code Formatting

The project uses Spotless for code formatting. To format the code, run the following command:

```bash
./gradlew spotlessApply
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firebase/firebase-android-sdk](https://github.com/firebase/firebase-android-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
