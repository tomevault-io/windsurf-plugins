---
trigger: always_on
description: This document provides an overview of the DemodAPK project, its purpose, and how to interact with it, particularly from the perspective of an automated agent like Gemini.
---

# DemodAPK Project Documentation (for Gemini)

This document provides an overview of the DemodAPK project, its purpose, and how to interact with it, particularly from the perspective of an automated agent like Gemini.

## Project Overview

DemodAPK is a Python-based tool designed for modifying Android Application Packages (APKs). It allows for various patching operations, including:
- Updating package names in the manifest and resources.
- Modifying application names.
- Configuring Facebook integration settings.
- Performing Smali code modifications.
- Updating build configurations.
- Managing metadata within the `AndroidManifest.xml`.

The tool aims to provide a flexible and configurable way to automate common APK modification tasks.

## Key Files and Directories

-   `demodapk/`: Contains the core Python source code for the tool.
    -   `demodapk/cli.py`: The command-line interface entry point.
    -   `demodapk/patch.py`: Contains functions for various APK patching operations.
    -   `demodapk/xmls.py`: Handles modifications to XML files, primarily `AndroidManifest.xml`.
    -   `demodapk/schema.json`: The JSON schema defining the structure and valid options for the configuration file.
-   `pyproject.toml`: Project metadata and build system configuration.
-   `requirements.txt`: Python dependencies for the project.
-   `mkdocs.yml`: Configuration for the project's documentation website.

## How to Use DemodAPK

The `demodapk` tool is executed from the command line. It requires an APK file and a configuration file (typically `config.json`) to specify the desired modifications.

**Basic Usage:**

```bash
demodapk <path_to_apk_file> --config <path_to_config.json>
```

**Example Configuration (`config.json`):**

The configuration file follows the schema defined in `demodapk/schema.json`. Here's a simplified example:

```json
{
  "DemodAPK": {
    "com.example.oldpackage": {
      "package": "com.example.newpackage",
      "app_name": "My New App",
      "manifest": {
        "hide_app_icon": true,
        "app_debuggable": true
      },
      "level": 2
    }
  }
}
```

## Implemented Features

### Hide App Icon

**Functionality:** This feature hides the application's icon from the Android launcher (app drawer and home screen). The app remains installed but is not directly launchable from the standard UI.

**Technical Implementation:** It modifies the `AndroidManifest.xml` by changing the `android:name` attribute of the `<category>` tag from `android.intent.category.LAUNCHER` to `android.intent.category.DEFAULT` within any `<intent-filter>` that also contains `android.intent.action.MAIN`. This prevents the Android system from recognizing the activity as a launcher entry point.

**Configuration:** To enable this feature, set `"hide_app_icon": true` within the `"manifest"` section of your app's configuration in the JSON file.

### Manifest Configuration Options

**Functionality:** The `manifest` section within your app's configuration allows for direct manipulation of the `AndroidManifest.xml` file.

**Supported Options:**
-   `hide_app_icon` (boolean): If `true`, hides the app icon from the launcher.
-   `app_debuggable` (boolean): If `true`, sets `android:debuggable="true"` on the `<application>` tag.
-   `app_label` (string): Sets the `android:label` attribute on the `<application>` tag.
-   `remove_metadata` (list of strings): Removes specified `<meta-data>` entries from the manifest.
-   `version_targetsdk` (integer): Sets the `android:targetSdkVersion` attribute on the `<uses-sdk>` tag.
-   `version_code` (integer): Sets the `android:versionCode` attribute on the `<manifest>` tag.
-   `version_name` (string): Sets the `android:versionName` attribute on the `<manifest>` tag.

**Example Configuration (`config.json`):**

```json
{
  "DemodAPK": {
    "com.example.oldpackage": {
      "package": "com.example.newpackage",
      "app_name": "My New App",
      "manifest": {
        "hide_app_icon": true,
        "app_debuggable": true,
        "app_label": "My Renamed App",
        "remove_metadata": ["com.example.SOME_KEY"],
        "version_targetsdk": 33,
        "version_code": 100,
        "version_name": "1.0.0-beta"
      },
      "level": 2
    }
  }
}
```

## Development Guidelines for Gemini

-   **Adhere to `demodapk/schema.json`:** When generating or modifying configuration files, always ensure they conform to the defined schema.
-   **XML Modifications:** Use `xml.etree.ElementTree` for `AndroidManifest.xml` modifications, following existing patterns in `demodapk/xmls.py`.
-   **Messaging:** Maintain consistency with the `demodapk.utils.msg` module for all output messages.
-   **Code Quality (Pylint):** Run `pylint .` to ensure code quality and aim to maintain a high score (e.g., 9.0 or above).
-   **Testing:** When adding new features, consider adding corresponding tests to ensure functionality and prevent regressions.

## Building Executables with PyInstaller


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Veha0001/DemodAPK](https://github.com/Veha0001/DemodAPK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
