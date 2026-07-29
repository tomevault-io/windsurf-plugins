---
trigger: always_on
description: matter.js is a comprehensive TypeScript implementation of the Matter/Thread smart home protocol. This is a monorepo containing multiple packages that work together to provide Matter protocol support for JavaScript/TypeScript applications.
---

# GitHub Copilot Instructions for matter.js

## Project Overview

matter.js is a comprehensive TypeScript implementation of the Matter/Thread smart home protocol. This is a monorepo containing multiple packages that work together to provide Matter protocol support for JavaScript/TypeScript applications.

## Architecture & Key Packages

### Core Packages

- `@matter/general` - Core utilities, crypto, networking abstractions
- `@matter/protocol` - Matter protocol implementation, commissioning, clustering
- `@matter/model` - Matter data model, cluster definitions, device types
- `@matter/node` - Node/endpoint implementations, behaviors, supervision
- `@matter/types` - TypeScript type definitions for Matter clusters and data types

### Platform Packages

- `@matter/nodejs` - Node.js platform implementation
- `@matter/nodejs-ble` - Bluetooth Low Energy support for Node.js
- `@matter/nodejs-shell` - Interactive shell for Matter operations

### Application Packages

- `@matter/main` - Main entry point package
- `@matter/examples` - Example applications and devices
- `@matter/create` - Project scaffolding tool
- `@project-chip/matter.js` - Legacy compatibility package

### Development Tools

- `packages/tools` - Build system, documentation generation, project management
- `support/codegen` - Code generation from Matter specifications
- `support/chip-testing` - Integration with Project CHIP/connectedhomeip for testing

## Code Generation System

This project heavily uses code generation:

### Cluster Generation

- Clusters are generated from Matter specifications in `support/codegen/src/clusters/`
- Use `ClusterFile`, `ClusterComponentGenerator` for cluster definitions
- Generated files follow pattern: `src/clusters/[ClusterName].ts`

### Endpoint Generation

- Device endpoints generated in `support/codegen/src/endpoints/`
- Use `EndpointFile`, `RequirementGenerator` for device type definitions
- Generated files follow pattern: `src/endpoints/[DeviceType].ts`

### Forward Exports

- Re-export generation in `support/codegen/src/forwards/`
- Creates proxy modules for clean package boundaries
- Generated files include header: `/*** THIS FILE IS GENERATED, DO NOT EDIT ***/`
- Pattern for main package forwards: `packages/main/src/forwards/[category]/[name].ts`

## Development Patterns

### Behaviors

- Core abstraction for endpoint functionality in `@matter/node`
- Extend `Behavior` class for cluster implementations
- Use `@behavior` decorator for registration
- File pattern: `src/behaviors/[cluster-name]/[ClusterName]Behavior.ts`

### Environment and ServerNode

- `Environment` provides platform-specific runtime services registered by each platform (Node.js, React Native, etc.)
- Access the default environment for your platform using `Environment.default`
- Create `ServerNode` instances for Matter devices:
    ```typescript
    const server = await ServerNode.create({
        id: "unique-device-id",
        network: { port: 5540 },
        commissioning: { passcode: 20202021, discriminator: 3840 },
        // ... other config
    });
    ```
- Add endpoints to nodes: `await server.add(endpoint);`
- Start the server non-blocking: `await server.start();` (resolves when online)
- Run the server blocking: `await server.run();` (resolves when server shuts down)
- See `examples/device-onoff-advanced/src/DeviceNodeFull.ts` for comprehensive examples

### Models

- Use `ClusterModel`, `DeviceTypeModel`, `AttributeModel` etc. from `@matter/model`
- Models represent Matter specification elements
- Support variance analysis for conditional features

### Type Safety

- Extensive use of TypeScript generics and conditional types
- **IMPORTANT**: Requires at least `"strictNullChecks": true` or preferably `"strict": true`
- Base TypeScript configuration in `packages/tools/tsc/tsconfig.base.json` uses `"strict": true`
- Schema validation with `Schema` classes

## CLI Tools and Examples

### Available CLI Tools

- `nacho-build` - Build packages and documentation
- `nacho-run` - Execute TypeScript files with automatic transpilation and source maps
- `matter-test` - Run tests across workspace packages
- `matter-create` - Scaffolding tool for new Matter.js projects
- `matter-version` - Version management tool

### Example Applications

The repository includes ready-to-run example applications:

```bash
npm run matter-device       # Simple on/off device
npm run matter-bridge       # Bridge with multiple devices
npm run matter-composeddevice # Composed device example
npm run matter-multidevice  # Multiple device example
npm run matter-controller   # Controller example
npm run shell              # Interactive Matter shell
```

### Running Examples

Use `nacho-run` to execute any TypeScript example directly:

```bash
nacho-run examples/device-onoff/src/DeviceNode.ts
nacho-run examples/controller/src/ControllerNode.ts
```

## TypeScript Configuration

### Required Settings

- **Minimum required**: `"strictNullChecks": true`
- **Recommended**: `"strict": true` for best type safety
- **Module settings**: `"module": "node16"`, `"moduleResolution": "node16"`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matter-js/matter.js](https://github.com/matter-js/matter.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
