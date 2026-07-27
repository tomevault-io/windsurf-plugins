---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Homebridge SwitchBot Plugin Development Guide

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Topic-Based Reference Priority

When a prompt includes the topics below, consult these upstream references first before implementation decisions.

### Matter (keyword examples: "Matter", "child bridge", "cluster", "conformance")

Primary references:
- https://github.com/homebridge-plugins/homebridge-matter
- https://github.com/homebridge/homebridge/ - only if latest is 2.0.0 or later and contains Matter-related code or reference: 
   - https://github.com/homebridge/homebridge/tree/beta-2.0.0/
- https://github.com/matter-js/matter.js

Usage rule:
- For Matter behavior, conformance, clustering, or registration logic, align implementation with these references before applying local changes.

### SwitchBot BLE (keyword examples: "BLE", "Bluetooth", "RSSI", "advertisement", "scan")

Primary references:
- https://github.com/OpenWonderLabs/SwitchBotAPI-BLE
- https://github.com/sblibs/pySwitchbot

Usage rule:
- For BLE command/state/scan behavior, prefer these references for protocol expectations and payload shape.

### SwitchBot OpenAPI (keyword examples: "OpenAPI", "cloud API", "device list", "command")

Primary reference:
- https://github.com/OpenWonderLabs/SwitchBotAPI

Usage rule:
- For OpenAPI device models, commands, and response fields, prioritize this reference for compatibility.

### Conflict Resolution

If references conflict:
1. Prefer official upstream docs/API specs.
2. Keep existing plugin behavior unless a clear bug is identified.
3. Document any intentional deviation in PR notes.

## Branch Targeting Strategy

When creating pull requests, ALWAYS follow this branch targeting strategy:

1. **Primary Target:** Look for branches that start with "beta-" (e.g., `beta-4.3.2`, `beta-4.4.0`)
   - If a beta branch exists, target the most recent beta branch for your PR
   - Beta branches are used for testing and staging changes before they go to the main branch

2. **Beta Branch Creation:** If no beta branches exist, create one based on the next possible version
   - Parse the current version from `package.json` (e.g., "4.3.1")
   - Determine version increment type using issue/PR labels (patch/minor/major)
   - Calculate the appropriate next version based on detected labels
   - Create a new beta branch with the format `beta-X.Y.Z` (e.g., `beta-4.3.2`)
   - Base the new beta branch on the `latest` branch
   - Push the new beta branch to origin and target it for the PR

3. **Fallback Target:** Only if beta branch creation fails, target the `latest` branch
   - The `latest` branch is the main development branch when no beta releases are in progress

4. **Branch Detection and Creation:** Use these methods to check and create branches:
   - **Detection:** GitHub API: Use `github-mcp-server-list_branches` to get all branches and filter for "beta-" prefix
   - **Detection:** Git command: `git ls-remote --heads origin | grep beta | sort -V | tail -1` to find the latest beta branch
   - **Version parsing:** Extract version from package.json: `node -p "require('./package.json').version"`
   - **Label Detection:** Check for version increment labels on the issue/PR:
     - Use GitHub API to get issue/PR labels
     - Look for labels: `patch`, `minor`, `major`
     - These labels should be set before assigning the issue to Copilot
   - **Version Increment Logic:** Choose increment type based on detected labels:
     - **patch label found:** `node -p "const v=require('./package.json').version.split('.'); v[2]=parseInt(v[2])+1; v.join('.')"` (4.3.1 → 4.3.2)
     - **minor label found:** `node -p "const v=require('./package.json').version.split('.'); v[1]=parseInt(v[1])+1; v[2]='0'; v.join('.')"` (4.3.1 → 4.4.0)
     - **major label found:** `node -p "const v=require('./package.json').version.split('.'); v[0]=parseInt(v[0])+1; v[1]='0'; v[2]='0'; v.join('.')"` (4.3.1 → 5.0.0)
     - **No relevant labels:** Default to patch increment as fallback
     - **Multiple increment labels:** Use highest priority (major > minor > patch)
   - **Label-Based Beta Branch Creation:**
     ```bash
     # Detect version increment type from issue/PR labels
     # Use GitHub API: github-mcp-server-get_issue or github-mcp-server-get_pull_request
     # Check labels array for: patch, minor, major
     
     # Calculate next version based on detected labels
     if [[ labels contains "major" ]]; then
       NEXT_VERSION=$(node -p "const v=require('./package.json').version.split('.'); v[0]=parseInt(v[0])+1; v[1]='0'; v[2]='0'; v.join('.')")
     elif [[ labels contains "minor" ]]; then
       NEXT_VERSION=$(node -p "const v=require('./package.json').version.split('.'); v[1]=parseInt(v[1])+1; v[2]='0'; v.join('.')")
     else
       # Default to patch increment (includes when "patch" label found or no labels)
       NEXT_VERSION=$(node -p "const v=require('./package.json').version.split('.'); v[2]=parseInt(v[2])+1; v.join('.')")
     fi
     
     BETA_BRANCH="beta-${NEXT_VERSION}"
     

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenWonderLabs/homebridge-switchbot](https://github.com/OpenWonderLabs/homebridge-switchbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
