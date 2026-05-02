---
trigger: always_on
description: - `src`: Source code directory
---

# UniCli

## Project Structure

- `src`: Source code directory
    - `src/UniCli.Client`: CLI project for `unicli`
    - `src/UniCli.Unity`: Unity server implementation for `unicli` (Unity project)
        - `src/UniCli.Unity/Packages/com.yucchiy.unicli-server/Editor`: Editor-side command handlers
        - `src/UniCli.Unity/Packages/com.yucchiy.unicli-server/Runtime/Remote`: Runtime debug commands (runs on device via PlayerConnection)
        - `src/UniCli.Unity/Assets/Samples`: Sample implementations for the server package
    - `src/UniCli.SourceGenerator`: Roslyn Source Generator for auto-generating Inspect command handlers
- `src/UniCli.Protocol`: Shared type definitions between `UniCli.Client` and `UniCli.Unity`
- `samples`: Sample Unity projects for multi-version testing
    - `samples/UniCli.Samples.Unity2022LTS`: Unity 2022.3 LTS project
    - `samples/UniCli.Samples.Unity6LTS`: Unity 6 (6000.0) LTS project
- `doc`: Documentation directory
    - `doc/command-naming-guidelines.md`: Command naming conventions for handlers

## Quick Commands

```bash
# Build Protocol (must be built first to trigger file copy)
dotnet build src/UniCli.Protocol

# Build Client
dotnet build src/UniCli.Client

# Publish Client and test with the built binary
dotnet publish src/UniCli.Client -o .build
UNICLI_PROJECT=src/UniCli.Unity .build/unicli commands --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec Compile --json

# GameObject operations
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec GameObject.Create '{"name":"TestObject"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec GameObject.Create '{"name":"Child","parent":"TestObject"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec GameObject.Create '{"name":"WithCollider","components":["BoxCollider"]}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec GameObject.CreatePrimitive '{"primitiveType":"Cube"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec GameObject.CreatePrimitive '{"primitiveType":"Sphere","name":"Ball","parent":"Parent"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec GameObject.GetComponents '{"path":"Main Camera"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec GameObject.AddComponent '{"path":"Main Camera","typeName":"BoxCollider"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec GameObject.RemoveComponent '{"componentInstanceId":12345}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec GameObject.Destroy '{"path":"TestObject"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec GameObject.SetTransform '{"path":"TestObject","position":[1,2,3]}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec GameObject.Duplicate '{"path":"TestObject"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec GameObject.Rename '{"path":"TestObject","name":"Renamed"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec GameObject.SetParent '{"path":"Child","parentPath":"Parent"}' --json

# Component operations
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec Component.SetProperty '{"componentInstanceId":12345,"propertyPath":"m_IsKinematic","value":"true"}' --json

# AnimatorController operations
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec AnimatorController.Create '{"assetPath":"Assets/Test.controller"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec AnimatorController.Inspect '{"assetPath":"Assets/Test.controller"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec AnimatorController.AddParameter '{"assetPath":"Assets/Test.controller","name":"Speed","type":"Float"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec AnimatorController.RemoveParameter '{"assetPath":"Assets/Test.controller","name":"Speed"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec AnimatorController.AddState '{"assetPath":"Assets/Test.controller","name":"Idle"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec AnimatorController.AddTransition '{"assetPath":"Assets/Test.controller","sourceStateName":"Idle","destinationStateName":"Walk"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec AnimatorController.AddTransitionCondition '{"assetPath":"Assets/Test.controller","sourceStateName":"Idle","destinationStateName":"Walk","parameter":"Speed","mode":"Greater","threshold":0.1}' --json

# Animator operations
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec Animator.Inspect '{"path":"SomeGameObject"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec Animator.SetController '{"path":"SomeGameObject","controllerAssetPath":"Assets/Test.controller"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec Animator.SetParameter '{"path":"SomeGameObject","name":"Speed","value":"1.5"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec Animator.Play '{"path":"SomeGameObject","stateName":"Idle"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec Animator.CrossFade '{"path":"SomeGameObject","stateName":"Walk","transitionDuration":0.25}' --json

# Prefab operations
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec Prefab.GetStatus '{"path":"Main Camera"}' --json
UNICLI_PROJECT=src/UniCli.Unity .build/unicli exec Prefab.Instantiate '{"assetPath":"Assets/Prefabs/Enemy.prefab"}' --json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yucchiy/UniCli](https://github.com/yucchiy/UniCli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
