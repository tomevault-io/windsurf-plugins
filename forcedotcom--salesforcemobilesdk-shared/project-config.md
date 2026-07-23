---
trigger: always_on
description: The Salesforce Mobile SDK Shared repository contains JavaScript libraries and Cordova plugin source code that are shared across the Hybrid Mobile SDK implementations for iOS and Android. This is a **foundational repository** that provides the JavaScript layer for Cordova-based hybrid applications.
---

# CLAUDE.md — Salesforce Mobile SDK Shared JavaScript Libraries

---

## About This Project

The Salesforce Mobile SDK Shared repository contains JavaScript libraries and Cordova plugin source code that are shared across the Hybrid Mobile SDK implementations for iOS and Android. This is a **foundational repository** that provides the JavaScript layer for Cordova-based hybrid applications.

**Key constraint**: This is a **public SDK** and a **submodule dependency** of both Android and iOS-Hybrid repositories. Every change impacts hybrid apps across both platforms. Backward compatibility and cross-platform consistency are critical.

## Repository Role in SDK Architecture

This repository serves as the **source of truth** for hybrid JavaScript code:

```
SalesforceMobileSDK-Shared (this repo)
  ├── JavaScript Libraries (libs/)
  │   ├── cordova.force.js      - Cordova plugins
  │   ├── force.js              - REST API client
  │   ├── mobilesync.js         - Data sync framework
  │   └── force+*.js            - Extensions (promises, files)
  │
  ├── Test Suites (test/)
  │   └── JavaScript tests for SmartStore, MobileSync
  │
  ├── Sample Apps (samples/)
  │   └── HTML/JS sample applications
  │
  └── Dependencies (dependencies/)
      └── Third-party libraries (jQuery, Backbone, etc.)

           │
    ┌──────┴──────┐
    │             │
    ▼             ▼
Android Repo   iOS-Hybrid Repo
(submodule)    (submodule)
    │             │
    └──────┬──────┘
           │
           ▼
    CordovaPlugin Repo
    (copies from all sources)
           │
           ▼
     Hybrid Templates
     (use Cordova plugin)
```

### Submodule Relationships

This repo is included as a git submodule in:
- **SalesforceMobileSDK-Android** at `external/shared/`
- **SalesforceMobileSDK-iOS-Hybrid** at `external/shared/`

### Distribution to CordovaPlugin

The `SalesforceMobileSDK-CordovaPlugin` repository runs `tools/update.sh` to copy JavaScript files from this repo to create the final Cordova plugin package.

## Repository Structure

```
SalesforceMobileSDK-Shared/
├── libs/                         # Core JavaScript libraries
│   ├── cordova.force.js          # Cordova plugins (OAuth, SmartStore, MobileSync, SDKInfo)
│   ├── force.js                  # REST API client (callbacks)
│   ├── force+files.js            # File upload/download utilities
│   ├── force+promise.js          # Promise-based REST API client
│   └── mobilesync.js             # Data synchronization framework
│
├── test/                         # Test suites
│   ├── test.html                 # Test runner HTML page
│   ├── SFTestSuite.js            # Test framework base class
│   ├── SFSmartStoreTestSuite.js  # SmartStore tests
│   ├── SFMobileSyncTestSuite.js  # MobileSync tests
│   ├── MockCordova.js            # Mock Cordova for browser testing
│   ├── MockSmartStore.js         # Mock SmartStore plugin
│   └── MockMobileSyncPlugin.js   # Mock MobileSync plugin
│
├── samples/                      # Sample applications
│   ├── accounteditor/            # Account CRUD sample
│   ├── contactexplorer/          # Contact browser sample
│   ├── smartstoreexplorer/       # SmartStore demo
│   ├── mobilesyncexplorer/       # MobileSync demo
│   ├── fileexplorer/             # File operations sample
│   ├── userlist/                 # User list sample
│   ├── usersearch/               # User search sample
│   ├── userandgroupsearch/       # User and group search
│   ├── simplesyncreact/          # React-based sync sample
│   └── vfconnector/              # Visualforce connector sample
│
├── dependencies/                 # Third-party dependencies
│   ├── backbone/                 # Backbone.js (for MobileSync)
│   ├── jquery/                   # jQuery
│   ├── underscore/               # Underscore.js (for MobileSync)
│   ├── promise-polyfill/         # Promise polyfill
│   ├── qunit/                    # QUnit testing framework
│   ├── react/                    # React.js
│   ├── ratchet/                  # Ratchet mobile UI framework
│   ├── ratchet2/                 # Ratchet v2
│   └── fastclick/                # FastClick for mobile
│
├── gen/                          # Generated plugin files
│   ├── plugins/                  # Cordova plugin format
│   └── plugins_with_define/      # AMD module format
│
├── tools/                        # Build and generation scripts
└── credshelper/                  # Credential helper utilities
```

## JavaScript Libraries

### cordova.force.js
**Purpose**: Cordova plugins that bridge JavaScript to native iOS/Android SDK functionality

**Plugins**:
- **oauth**: Authentication, user management, logout
- **smartstore**: Encrypted local storage (soup operations, Smart SQL)
- **mobilesync**: Data synchronization (sync up/down, sync targets)
- **sdkinfo**: SDK version information
- **network**: REST API requests to Salesforce
- **sfaccountmanager**: Multi-user account management

**Usage**: Include after `cordova.js` in HTML:
```html
<script src="cordova.js"></script>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [forcedotcom/SalesforceMobileSDK-Shared](https://github.com/forcedotcom/SalesforceMobileSDK-Shared) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
