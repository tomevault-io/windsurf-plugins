---
trigger: always_on
description: ├── CustomAuthActionInputs.ts            # Input types for authentication actions
---


# Native Authentication Code Structure Guide

## Directory Structure

```
src/custom_auth/
├── CustomAuthActionInputs.ts            # Input types for authentication actions
├── CustomAuthConstants.ts               # Feature-wide constants and enums
├── CustomAuthPublicClientApplication.ts # Main SDK entry point
├── ICustomAuthPublicClientApplication.ts# Public API interface
├── UserAccountAttributes.ts             # User attribute types and utilities
├── index.ts                             # Barrel export file
├── configuration/
│   └── CustomAuthConfiguration.ts       # Configuration types and validation
├── controller/
│   ├── CustomAuthStandardController.ts  # Main business logic controller
│   └── ICustomAuthStandardController.ts # Controller interface
├── core/                                # Shared utilities and base classes
│   ├── CustomAuthAuthority.ts           # Authority logic and validation
│   ├── auth_flow/                       # Base auth flow classes and JIT components
│   │   └── jit/                         # JIT-specific auth flow components
│   │       ├── error_type/              # JIT error types
│   │       ├── result/                  # JIT result types
│   │       └── state/                   # JIT state types
│   ├── error/                           # Core error types and error codes
│   ├── interaction_client/              # Base interaction client classes
│   ├── network_client/                  # HTTP and API clients
│   ├── telemetry/                       # Telemetry and logging
│   └── utils/                           # Utility functions
├── get_account/                         # Account management flow
│   ├── auth_flow/                       # States, results, errors
│   └── interaction_client/              # Account-specific clients
├── operating_context/
│   └── CustomAuthOperatingContext.ts    # Operating context definition
├── reset_password/                      # Password reset flow
│   ├── auth_flow/                       # States, results, errors
│   └── interaction_client/              # Reset password clients
├── sign_in/                             # Sign-in flow
│   ├── auth_flow/                       # States, results, errors
│   └── interaction_client/              # Sign-in clients
└── sign_up/                             # Sign-up flow
    ├── auth_flow/                       # States, results, errors
    └── interaction_client/              # Sign-up clients
```

## File Naming Conventions

### Class Files

-   PascalCase for class names: `CustomAuthPublicClientApplication.ts`
-   Interface files prefixed with 'I': `ICustomAuthPublicClientApplication.ts`
-   Base classes suffixed with 'Base': `AuthFlowErrorBase.ts`

### Feature Folders

-   snake_case for folder names: `sign_in/`, `reset_password/`, `get_account/`
-   Consistent subfolder structure: `auth_flow/`, `interaction_client/`

### Test Files

-   Mirror source structure in `test/custom_auth/`
-   Test files suffixed with `.spec.ts`
-   Test utilities in `test_resources/` folder

## Component Relationships

### Entry Point → Controller → Client Chain

```
CustomAuthPublicClientApplication
    ↓ delegates to
CustomAuthStandardController
    ↓ creates via factory
SignInClient/SignUpClient/ResetPasswordClient
    ↓ uses
CustomAuthApiClient (composed of flow-specific API clients)
```

### State Machine Flow

```
Initial State → Action → New State → Result Object
    ↓
User checks result type (isPasswordRequired, isCodeRequired, etc.)
    ↓
User calls method on state object (submitPassword, submitCode, etc.)
    ↓
New State/Result returned
```

## Code Organization Patterns

### Flow-Specific Structure

Each authentication flow (sign_in, sign_up, reset_password) follows this pattern:

```
{flow_name}/
├── auth_flow/
│   ├── error_type/
│   │   └── {FlowName}Error.ts          # Flow-specific error types
│   ├── result/
│   │   ├── {FlowName}Result.ts         # Main flow result
│   │   └── {FlowName}*Result.ts        # Step-specific results
│   └── state/
│       ├── {FlowName}State.ts          # Base state class
│       ├── {FlowName}*State.ts         # Step-specific states
│       └── {FlowName}StateParameters.ts # State parameter interfaces
└── interaction_client/
    ├── {FlowName}Client.ts             # Main interaction client
    ├── parameter/
    │   └── {FlowName}Params.ts         # Client parameter types
    └── result/
        └── {FlowName}ActionResult.ts   # Client action result types
```

### Core Module Structure

```
core/
├── CustomAuthAuthority.ts             # Authority logic and validation
├── auth_flow/                          # Base auth flow classes and shared flow components
│   ├── AuthFlowErrorBase.ts            # Base error class for auth flows
│   ├── AuthFlowResultBase.ts           # Base result class for auth flows
│   ├── AuthFlowState.ts                # Base state class for auth flows
│   └── jit/                            # JIT-specific auth flow components
│       ├── error_type/                 # JIT error types
│       ├── result/                     # JIT result types
│       └── state/                      # JIT state types
├── error/                              # Core error types and error codes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AzureAD/microsoft-authentication-library-for-js](https://github.com/AzureAD/microsoft-authentication-library-for-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
