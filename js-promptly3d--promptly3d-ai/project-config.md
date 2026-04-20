---
trigger: always_on
description: This rule provides guidelines and best practices for developing efficient, maintainable, and robust 3D web applications using Three.js. It covers aspects like code organization, performance optimization, security, testing, and common pitfalls to ensure a high-quality development experience.
---

# Three.js Best Practices and Coding Standards

This document outlines the best practices and coding standards for developing with Three.js. Adhering to these guidelines will help you create efficient, maintainable, and robust 3D web applications.

## Library Information:

- Name: three.js
- Tags: 3d, graphics, webgl, javascript

## 1. Code Organization and Structure

### 1.1. Directory Structure Best Practices:

A well-organized directory structure enhances project maintainability and scalability. Consider the following structure:


project-root/
├── src/
│   ├── components/        # Reusable 3D components (e.g., models, scenes)
│   │   ├── MyComponent.js
│   │   └── ...
│   ├── scenes/           # Specific scenes for different parts of the application
│   │   ├── MainScene.js
│   │   └── ...
│   ├── models/           # 3D model files (e.g., .glb, .obj)
│   │   ├── myModel.glb
│   │   └── ...
│   ├── textures/         # Texture files (e.g., .jpg, .png)
│   │   ├── myTexture.jpg
│   │   └── ...
│   ├── shaders/          # Custom shader code (GLSL)
│   │   ├── vertexShader.glsl
│   │   └── fragmentShader.glsl
│   ├── utils/            # Utility functions (e.g., math, helpers)
│   │   ├── mathUtils.js
│   │   └── ...
│   ├── core/             # Core application logic (e.g., scene initialization, rendering loop)
│   │   ├── renderer.js
│   │   ├── camera.js
│   │   ├── scene.js
│   │   └── controls.js
│   ├── app.js            # Main application entry point
│   └── index.html        # HTML file to load the application
├── dist/              # Distribution build
├── node_modules/      # Node modules
├── .gitignore
├── package.json
└── webpack.config.js    # Or equivalent build tool configuration


### 1.2. File Naming Conventions:

Consistent file naming improves code readability.

-   **Components:** Use PascalCase (e.g., `MyComponent.js`).
-   **Scenes:** Use PascalCase (e.g., `MainScene.js`).
-   **Models:** Use camelCase (e.g., `myModel.glb`).
-   **Textures:** Use camelCase (e.g., `myTexture.jpg`).
-   **Shaders:** Use camelCase (e.g., `vertexShader.glsl`, `fragmentShader.glsl`).
-   **Utilities:** Use camelCase (e.g., `mathUtils.js`).

### 1.3. Module Organization:

Employ ES modules for better code organization and dependency management.

javascript
// MyComponent.js
import * as THREE from 'three';

export class MyComponent extends THREE.Mesh {
 constructor() {
 super();
 // ... component logic ...
 }
}


javascript
// app.js
import { MyComponent } from './components/MyComponent.js';
import { MainScene } from './scenes/MainScene.js';

const scene = new MainScene();
const myComponent = new MyComponent();
scene.add(myComponent);

// ... rest of the application logic ...


### 1.4. Component Architecture:

Use a component-based architecture to create reusable and modular 3D elements.

-   Encapsulate Three.js objects (e.g., meshes, lights) within components.
-   Create components with well-defined interfaces (props and methods).
-   Use a scene graph to manage the hierarchy of components.

### 1.5. Code Splitting Strategies:

For large applications, use code splitting to improve initial load time.

-   Split the application into smaller chunks based on routes or features.
-   Use dynamic imports to load components or scenes on demand.
-   Leverage Webpack or Parcel to configure code splitting.

## 2. Common Patterns and Anti-patterns

### 2.1. Design Patterns:

-   **Factory Pattern:**  Use factory functions or classes to create complex Three.js objects. This can encapsulate the object creation logic, making it easier to manage and reuse. For example, a `ModelFactory` can load and process different 3D model formats.
-   **Observer Pattern:** Implement an observer pattern for handling events and interactions in the 3D scene. For instance, you can use this to notify components when a user clicks on a specific object or when the scene changes.
-   **Singleton Pattern:** For global resources like the renderer or scene manager, consider using a singleton pattern to ensure only one instance exists.  However, be cautious as singletons can sometimes make testing more difficult.
-   **Command Pattern:** Decouple the execution of actions from their trigger by encapsulating them in command objects. This allows for easier undo/redo functionality and more flexible control over the 3D scene.

### 2.2. Recommended Approaches for Common Tasks:

-   **Loading Models:** Use `GLTFLoader` for loading GLTF/GLB models, which are efficient and widely supported.  Handle asynchronous loading using `async/await` or Promises.
-   **Texture Management:** Use `TextureLoader` to load textures. Ensure textures are properly disposed of when no longer needed to prevent memory leaks.
-   **Animation:** Use `AnimationMixer` to play animations from loaded models.  Optimize animation performance by reducing the number of animated objects and keyframes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/js-promptly3d) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
