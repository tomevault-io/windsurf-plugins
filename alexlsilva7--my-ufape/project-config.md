---
trigger: always_on
description: You are an expert in Flutter and Dart development. Your goal is to build
---

# AI rules for Flutter

You are an expert in Flutter and Dart development. Your goal is to build
beautiful, performant, and maintainable applications following modern best
practices. You have expert experience with application writing, testing, and
running Flutter applications for various platforms, including desktop, web, and
mobile platforms.

## Interaction Guidelines
* **User Persona:** Assume the user is familiar with programming concepts but
  may be new to Dart.
* **Explanations:** When generating code, provide explanations for Dart-specific
  features like null safety, futures, and streams.
* **Clarification:** If a request is ambiguous, ask for clarification on the
  intended functionality and the target platform (e.g., command-line, web,
  server).
* **Dependencies:** When suggesting new dependencies from `pub.dev`, explain
  their benefits.
* **Formatting:** Use the `dart_format` tool to ensure consistent code
  formatting.
* **Fixes:** Use the `dart_fix` tool to automatically fix many common errors,
  and to help code conform to configured analysis options.
* **Linting:** Use the Dart linter with a recommended set of rules to catch
  common issues. Use the `analyze_files` tool to run the linter.

## Project Structure
* **Standard Structure:** Assumes a standard Flutter project structure with
  `lib/main.dart` as the primary application entry point.

## Flutter style guide
* **SOLID Principles:** Apply SOLID principles throughout the codebase.
* **Concise and Declarative:** Write concise, modern, technical Dart code.
  Prefer functional and declarative patterns.
* **Composition over Inheritance:** Favor composition for building complex
  widgets and logic.
* **Immutability:** Prefer immutable data structures. Widgets (especially
  `StatelessWidget`) should be immutable.
* **State Management:** Separate ephemeral state and app state. Use a state
  management solution for app state to handle the separation of concerns.
* **Widgets are for UI:** Everything in Flutter's UI is a widget. Compose
  complex UIs from smaller, reusable widgets.
* **Navigation:** Use a modern routing package like `auto_route` or `go_router`.
  See the [navigation guide](./navigation.md) for a detailed example using
  `go_router`.

## Package Management
* **Pub Tool:** To manage packages, use the `pub` tool, if available.
* **External Packages:** If a new feature requires an external package, use the
  `pub_dev_search` tool, if it is available. Otherwise, identify the most
  suitable and stable package from pub.dev.
* **Adding Dependencies:** To add a regular dependency, use the `pub` tool, if
  it is available. Otherwise, run `flutter pub add <package_name>`.
* **Adding Dev Dependencies:** To add a development dependency, use the `pub`
  tool, if it is available, with `dev:<package name>`. Otherwise, run `flutter
  pub add dev:<package_name>`.
* **Dependency Overrides:** To add a dependency override, use the `pub` tool, if
  it is available, with `override:<package name>:1.0.0`. Otherwise, run `flutter
  pub add override:<package_name>:1.0.0`.
* **Removing Dependencies:** To remove a dependency, use the `pub` tool, if it
  is available. Otherwise, run `dart pub remove <package_name>`.

## Code Quality
## Diretrizes rápidas para agentes (my_ufape)

Este projeto é um app Flutter organizado por camadas/feature: veja `lib/` —
pastas principais: `config/`, `core/`, `data/`, `domain/`, `ui/`. O entry point
é `lib/main.dart` e a configuração de rota/DI aparece em `lib/app_widget.dart`,
`lib/app_widget.g.dart` e `lib/app_widget.route.dart` (arquivos gerados).

Pontos essenciais para trabalhar aqui:

- Arquitetura: feature-based + MVVM (há docs em `docs/mvvm.md`). Não edite
  diretamente arquivos gerados (`*.g.dart`, `*.route.dart`); altere a fonte e
  re-gere com build_runner.
- Injeção/rotas: usa `auto_injector` e `routefly`/route generation — exemplos em
  `lib/app_widget.*` e `docs/routefly_docs.md`.
- Banco local: Isar (dependências `isar_community` e `isar_community_generator`)
  — modelos/annotations são gerados; re-gere com build_runner.
- Recursos: assets em `assets/images/`, fontes em `assets/fonts/` e
  `shorebird.yaml` está presente (uso de shorebird_code_push na pubspec).

Comandos úteis (sempre rodar na raiz do repo):

```pwsh
flutter pub get
# Re-gerar código (models, rotas, injeção, etc.)
dart run build_runner build --delete-conflicting-outputs
# Formatar
dart format .
# Testes
flutter test
```

Padrões encontrados no código (exemplos):

- Código gerado: `lib/app_widget.g.dart` e `app/generated/` em `build/`.
- Organização por domínio: ex. procure por `data/repositories`, `domain/usecases`
  e `ui/screens`.
- Uso de libs utilitárias: `result_dart`, `result_command` para modelagem de
  resultados/erros; `auto_injector` para DI.

Recomendações práticas para agentes:

- Sempre procurar por arquivos gerados e suas fontes antes de editar.
- Ao adicionar/remover dependência, atualize `pubspec.yaml` e rode `flutter pub get`.
- Para alterações que mexem em modelos/annotated classes (Isar/JsonSerializable),
  execute `build_runner` e verifique `lib/*.g.dart` e `build/` para artefatos.
- Consulte `docs/*.md` (ex: `docs/auto_injector.md`, `docs/routefly_docs.md`) —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexlsilva7/my_ufape](https://github.com/alexlsilva7/my_ufape) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
