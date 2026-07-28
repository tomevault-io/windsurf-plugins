---
trigger: always_on
description: **Purpose**: Package manifests enable Hubitat Package Manager (HPM) to install and update drivers on user hubs.
---


# HPM / PackageManifests instructions

**Purpose**: Package manifests enable Hubitat Package Manager (HPM) to install and update drivers on user hubs.

---

## IMPORTANT: Active vs. Legacy code

**Only create/update manifests for active drivers**:

- Drivers in `WebhookWebsocket/` folder
- Drivers in `ComponentDrivers/` folder
- Drivers in `Bluetooth/` folder
- The shared `ShellyDriverLibrary/`

**DO NOT create manifests for legacy code**:

- Drivers in `PLUS/` folder
- Root-level `Shelly*.groovy` files
- Drivers in `contribs/` folder

Legacy drivers should not be distributed through HPM.

---

## Manifest structure

HPM manifests are JSON files that describe:

- Package metadata (name, author, description)
- Driver/app files to install
- Version information
- Compatibility requirements
- Update/release notes

**Key fields**:

```json
{
  "packageName": "Shelly Driver Name",
  "author": "Your Name",
  "version": "X.Y.Z",
  "minimumHEVersion": "2.2.0",
  "dateReleased": "2026-02-04",
  "drivers": [
    {
      "id": "unique-driver-id",
      "name": "Driver Display Name",
      "namespace": "ShellyUSA",
      "location": "https://raw.githubusercontent.com/.../Driver.groovy",
      "required": true
    }
  ],
  "releaseNotes": "Description of changes in this release"
}
```

---

## When to update manifests

**Update a manifest when**:

1. Releasing a new driver version with user-facing changes
2. Adding a new driver to an existing package
3. Changing driver file locations or URLs
4. Updating compatibility requirements (`minimumHEVersion`)
5. Including new library files

**Do NOT update for**:

- Changes to legacy drivers
- Internal code refactors with no functional changes
- Documentation-only updates

---

## Manifest update workflow

### 1. Identify the correct manifest file

```
PackageManifests/
  ShellyWebhookDrivers/
    manifest.json  // For WebhookWebsocket drivers
  ShellyComponents/
    manifest.json  // For ComponentDrivers
  ShellyBluetooth/
    manifest.json  // For Bluetooth drivers
```

### 2. Update version and date

```json
{
  "version": "2.3.1", // Match version in resources/version.json
  "dateReleased": "2026-02-04" // Use ISO format: YYYY-MM-DD
}
```

**Version number must match `resources/version.json`** for consistency.

### 3. Update driver entries

Ensure all active drivers in the package are listed:

```json
"drivers": [
  {
    "id": "shellyplus1pm-driver",
    "name": "Shelly Plus 1PM",
    "namespace": "ShellyUSA",
    "location": "https://raw.githubusercontent.com/user/repo/main/WebhookWebsocket/ShellyPlus1PM.groovy",
    "required": true,
    "version": "2.3.1"  // Individual driver version
  },
  {
    "id": "shelly-library",
    "name": "Shelly Driver Library",
    "namespace": "ShellyUSA",
    "location": "https://raw.githubusercontent.com/user/repo/main/ShellyDriverLibrary/ShellyUSA.ShellyUSA_Driver_Library.groovy",
    "required": true,
    "version": "3.1.0"  // Library version
  }
]
```

**Critical**: Include the shared library in every manifest that uses it.

### 4. Add release notes

```json
"releaseNotes": "v2.3.1 - Fixed power reporting accuracy. Improved websocket reconnection logic."
```

**Release notes guidelines**:

- Keep concise (1-2 sentences)
- Focus on user-visible changes
- Match the tone of `UpdateInfo` in `resources/version.json`
- Use plain text (HTML not supported in HPM)

### 5. Verify URLs

Ensure all `location` URLs:

- Point to the correct branch (typically `main` or a release tag)
- Use `raw.githubusercontent.com` (not `github.com`)
- Are publicly accessible
- Point to the correct file path

**URL format**:

```
https://raw.githubusercontent.com/[owner]/[repo]/[branch-or-tag]/[path-to-file]
```

---

## Compatibility requirements

### Minimum Hubitat version

```json
"minimumHEVersion": "2.2.0"
```

Set conservatively:

- Use `2.2.0` for most drivers (stable feature set)
- Use `2.3.0+` if using newer API features
- Document any specific firmware requirements in driver README

### Device compatibility

Document compatible Shelly device models in:

- Manifest `description` field
- Driver metadata
- Repository README

---

## Library dependencies

**CRITICAL**: Every manifest using `ShellyDriverLibrary` must include it.

```json
"drivers": [
  {
    "id": "shelly-library",
    "name": "Shelly Driver Library",
    "namespace": "ShellyUSA",
    "location": "https://raw.githubusercontent.com/.../ShellyDriverLibrary/ShellyUSA.ShellyUSA_Driver_Library.groovy",
    "required": true,
    "version": "3.1.0"
  },
  // ... individual drivers that include the library
]
```

**Why**: Individual drivers use `#include ShellyUSA.ShellyUSA_Driver_Library` and won't work without the library installed.

---

## Validation before commit

### 1. JSON syntax validation

```bash
# Validate JSON
jq empty PackageManifests/*/manifest.json

# Pretty-print
jq . PackageManifests/ShellyWebhookDrivers/manifest.json
```

### 2. URL verification

```bash
# Test each driver location URL
curl -I "https://raw.githubusercontent.com/.../Driver.groovy"
# Should return 200 OK
```

### 3. Version consistency check

Ensure versions match between:

- `resources/version.json`
- Package manifest
- Driver metadata (in `.groovy` file)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShellyUSA/Hubitat-Drivers](https://github.com/ShellyUSA/Hubitat-Drivers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
