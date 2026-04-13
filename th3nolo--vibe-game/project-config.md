---
trigger: always_on
description: Always use Yarn for dependencies.
---

# .cursorrules

Always use Yarn for dependencies.

This file outlines the coding rules and best practices for the MMO RPG server project. These guidelines ensure clarity, maintainability, scalability, and performance in the codebase. They are intended for both developers and AI/LLM assistance.

## General Coding Rules

1. **Use WebAssembly (WASM) for CPU-Intensive Tasks**
   - Offload heavy computations (e.g., procedural generation, physics) to C++ or Rust compiled to WASM.
   - Use Emscripten to compile and integrate WASM modules with Node.js via `WebAssembly.instantiate`.

2. **Leverage Worker Threads with SharedArrayBuffer**
   - Use Node.js worker threads for concurrent game logic (e.g., player updates, region events).
   - Share memory using `SharedArrayBuffer` for efficient data access across threads.

3. **Maintain Thread Safety**
   - Use `Atomics` to synchronize access to shared memory and prevent race conditions.

4. **Batch WASM Calls**
   - Minimize the frequency of WASM calls by batching computations (e.g., generating multiple map chunks at once).

5. **Avoid Blocking the Main Thread**
   - Keep the main Node.js thread free for I/O operations (e.g., WebSocket communication).
   - Offload CPU-intensive tasks to WASM or worker threads.

6. **Use Efficient Data Structures**
   - Store game state and map data in typed arrays (e.g., `Int32Array`, `Float32Array`) with `SharedArrayBuffer`.

7. **Implement Scalable Architecture**
   - Design for scalability, starting with worker threads and scaling out with multiple Node.js instances if needed.

8. **Monitor and Optimize Performance**
   - Regularly profile the server using tools like Node.js's `--prof` or external services.

9. **Ensure Compatibility and Stability**
   - Use well-tested versions of Node.js (e.g., LTS), WASM, and libraries. Avoid experimental features.

10. **Document and Comment Code**
    - Include clear comments and documentation, especially for WASM integration and worker thread logic.
    - Use JSDoc for functions and classes.

## Dependency Management

11. **Mandatory Yarn Usage**
    - Use Yarn exclusively for installing, updating, and managing dependencies (e.g., `yarn add`, `yarn install`).
    - Do not use npm or other package managers in this project.
    - Lock dependency versions in `yarn.lock` for consistent installations.

## TypeScript Adoption

12. **Use TypeScript Exclusively**
    - Write all new code in TypeScript with `.ts` file extensions.
    - Migrate existing `.js` files to `.ts` and add type annotations for parameters, return types, and variables.
    - Use TypeScript's type inference to reduce explicit annotations where possible (e.g., `const x = 5` instead of `const x: number = 5`).
    - Configure `tsconfig.json` with strict type checking enabled.

13. **Define Interfaces for Abstraction**
    - Use TypeScript interfaces to define contracts for classes and modules (e.g., `interface Player { id: string; position: number }`).
    - Implement interfaces to ensure type safety and modularity.
    - Prefer interfaces over type aliases for better extensibility.

14. **Type Safety Best Practices**
    - Use strict null checks and avoid `any` type.
    - Leverage union types and type guards for runtime type safety.
    - Use generics for reusable type-safe components.

## Lean and Small Code

15. **Minimize Code Size**
    - Avoid code duplication; refactor repeated logic into reusable functions or modules.
    - Use concise syntax (e.g., arrow functions `() => x`, optional chaining `?.`, destructuring `{ a, b }`).
    - Remove unused code, variables, or imports immediately.
    - Use short, meaningful variable names where context is clear.

16. **Optimize Algorithm Efficiency**
    - Use the smallest, most efficient algorithms and data structures for tasks (e.g., `Set` for unique items, typed arrays for numeric data).
    - Implement caching strategies for frequently accessed data.
    - Use appropriate data structures for specific use cases (e.g., Map for key-value pairs).

## SOLID Principles

17. **Single Responsibility Principle (SRP)**
    - Ensure each class or module has one responsibility (e.g., `LobbyManager` only manages lobbies, not game logic).
    - Example: Split `server.ts` into separate I/O and logic handlers if it grows too complex.
    - Keep methods focused and small.

18. **Open-Closed Principle (OCP)**
    - Design modules to be extensible without modification (e.g., use interfaces and polymorphism instead of altering base classes).
    - Example: Add new lobby types via subclasses rather than changing `LobbyManager`.
    - Use abstract classes and interfaces for extensibility.

19. **Liskov Substitution Principle (LSP)**
    - Ensure subclasses can replace their base classes without breaking functionality (e.g., a `VIPPlayer` class should work wherever `Player` is expected).
    - Maintain consistent behavior across inheritance hierarchies.
    - Avoid overriding methods in ways that violate the base class contract.

20. **Interface Segregation Principle (ISP)**
    - Define small, specific interfaces rather than large, general ones (e.g., separate `Movable` and `Damageable` interfaces for entities).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/th3nolo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
