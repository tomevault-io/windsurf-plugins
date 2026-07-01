---
trigger: always_on
description: Guía operativa y de estilo para trabajar con este repositorio usando agentes.
---

# Guía de estilo y operaciones generales

Guía operativa y de estilo para trabajar con este repositorio usando agentes.

Precedencia de reglas
- Las reglas se aplican en orden de precedencia creciente: las que aparecen más tarde prevalecen sobre las anteriores.
- Las reglas de proyecto (asociadas a rutas concretas) tienen prioridad sobre reglas personales.
- Entre reglas de proyecto, las de subdirectorios prevalecen sobre las del directorio padre.

## Comunicación y formato
- Conversaciones y asistencia: en español.
- Código, mensajes de commit, comentarios de código y resúmenes de PR: en inglés.
- PR: usar texto sin escapar en asunto y cuerpo.

## Terminal y ejecución
- No cerrar la terminal ni ejecutar comandos que finalicen la sesión.
- Evitar comandos interactivos salvo que sea estrictamente necesario.
- Extremar cuidado con comillas simples y dobles en los comandos.

## Despliegue y CI

- Se realiza mediante azure-pipelines.yml.
- La build debe pasar correctamente antes de fusionar una PR.

## Lineamientos de diseño y estilo (C# / Reactive)

- Preferir programación funcional y reactiva cuando no complique en exceso.
- Validación: preferir ReactiveUI.Validations.
- Result handling: usar CSharpFunctionalExtensions cuando sea posible.
  - Preferir métodos idiomáticos (`Bind`, `Map`, `Tap`, `Ensure`, `Try`, `MapTry`, `Compensate`) frente a comprobaciones de estado (`IsFailure`, `IsSuccess`).
  - Evitar `Match` salvo que aporte mayor claridad.
- Convenciones:
  - No usar sufijo “Async” en métodos que devuelven Task.
  - No usar guiones bajos para campos privados.
  - Evitar eventos (salvo indicación explícita).
  - Favorecer inmutabilidad; mutar solo lo estrictamente necesario.
  - Evitar poner lógica en Observable.Subscribe; preferir encadenar operadores y proyecciones.

# Errores y notificaciones

- Para flujos de Result<T> usar el operador Successes.
- Para fallos, HandleErrorsWith() empleando INotificationService para notificar al usuario.

# Toolkit Zafiro

Es mi propio toolkit. Disponible en https://github.com/SuperJMN/Zafiro. Muchos de los métodos que no conozcas pueden formar parte de este toolkit. Tenlo en consideración.

# Manejo de bytes (sin Streams imperativos)

- Usar Zafiro.DivineBytes para flujos de bytes evitables con Stream.
- ByteSource es la abstracción observable y componible equivalente a un stream de lectura.

# Refactorización guiada por responsabilidades

1. Leer el código y describir primero sus responsabilidades.
2. Enumerar cada responsabilidad como una frase nominal clara.
3. Para cada responsabilidad, crear una clase o método con nombre específico y semántico.
4. Extraer campos y dependencias según cada responsabilidad.
5. Evitar variables compartidas entre responsabilidades; si aparecen, replantear los límites.
6. No introducir patrones arbitrarios; mantener la interfaz pública estable.
7. No eliminar logs ni validaciones existentes.


# General guidelines about this repo

This file guides Warp (and future contributors) on how CI/CD and packaging work in this repository.

Scope: whole repository (DotnetPackaging).

CI pipeline (Azure Pipelines)
- Definition: azure-pipelines.yml at repo root.
- Agent: windows-latest.
- Versioning: computed with GitVersion.Tool; packages use MajorMinorPatch as Version; GitHub Release tag uses v{SemVer}.
- Behavior on master:
  - Restore, build and pack all projects; push .nupkg (non-symbol) to NuGet (skip-duplicate) with $(NuGetApiKey).
  - Publish Windows EXE stubs (DotnetPackaging.Exe.Installer) for win-x64 and win-arm64 as single-file self-extract apps (IncludeNativeLibrariesForSelfExtract/IncludeAllContentForSelfExtract, no trimming).
  - Produce .sha256 for each stub and upload both .exe and .sha256 to a GitHub Release tagged v{SemVer} using gh CLI.
- Other branches/PRs: build and pack only (no push, no release).
- Packable projects: every project with IsPackable/PackAsTool set. The CLI tool lives in src/DotnetPackaging.Tool (PackAsTool=true).

Versioning (GitVersion)
- GitVersion.Tool runs in CI to produce:
  - Version: MajorMinorPatch (used for dotnet build/pack).
  - TagName: v{SemVer} (used to create/update the GitHub Release).
- Practical effect: merging to master triggers package publish to NuGet and stub upload to a GitHub Release for the computed tag.

Secrets
- The pipeline expects a variable group named api-keys providing:
  - NuGetApiKey: API key used to push packages to NuGet.
  - GitHubApiKey: token exposed as GITHUB_TOKEN to create/update releases and upload stub assets via gh.
- Do not hardcode secrets. Locally, export environment variables and pass them to the CLI tools.

Local replication
- Pack locally:
  - dotnet restore
  - dotnet build -c Release -p:ContinuousIntegrationBuild=true -p:Version=1.2.3 --no-restore
  - dotnet pack -c Release --no-build -p:IncludeSymbols=false -p:SymbolPackageFormat=snupkg -p:Version=1.2.3 -o ./artifacts/nuget
- Push to NuGet:
  - For each .nupkg (non-symbol): dotnet nuget push ./artifacts/nuget/<pkg>.nupkg --api-key "$env:NUGET_API_KEY" --source https://api.nuget.org/v3/index.json --skip-duplicate
- Build Windows stubs (on Windows):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SuperJMN/DotnetPackaging](https://github.com/SuperJMN/DotnetPackaging) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
