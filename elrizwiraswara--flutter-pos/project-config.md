---
trigger: always_on
description: **Clean Architecture** with five layers:
---

# CLAUDE.md - Flutter Project Conventions

## Architecture

**Clean Architecture** with five layers:

```
lib/
├── app/               # App root, DI, routing
├── core/              # Shared utilities, services, themes, constants, extensions
├── data/              # Datasources (local/remote), models, repository implementations
├── domain/            # Entities, abstract repositories, usecases
└── presentation/      # Providers (state), screens (UI), widgets (reusable)
```

### Detailed Structure

```
data
- datasources
  - interfaces
    - user_datasource.dart                # Datasource interface (shared by local & remote)  # abstract class UserDatasource {}
    - ...
  - remote
    - user_remote_datasource_impl.dart    # Remote datasource implementation                 # class UserRemoteDatasourceImpl implements UserDatasource {}
    - ...
  - local
    - user_local_datasource_impl.dart     # Local datasource implementation                  # class UserLocalDatasourceImpl implements UserDatasource {}
    - ...
- models
  - user_model.dart                       # Data model                                       # class UserModel {}
  - ...
- repositories
  - user_repository_impl.dart             # Repository implementation                        # class UserRepositoryImpl implements UserRepository {}
  - ...
domain
- entities
  - user_entity.dart                      # Entity object                                    # class UserEntity extends Equatable {}
- repositories
  - user_repository.dart                  # Repository interface                             # abstract class UserRepository {}
- usecases
  - params
    - base_params.dart                    # Shared usecase param types                       # class BaseParams {}
    - no_param.dart                       # Empty param marker                               # class NoParam {}
  - user_usecases.dart                    # Multiple usecase classes per file                # class GetUserUsecase extends Usecase<...> {}
  - ...
presentation
- providers
  - user
    - user_notifier.dart                  # Notifier                                         # class UserNotifier extends Notifier<UserState> {}
    - user_state.dart                     # State                                            # class UserState {}
  - ...
- screens
  - user
    - components
      - user_card.dart                    # Scoped widget                                    # class UserCard {}
      - ...
    - user_screen.dart                    # Screen                                           # class UserScreen {}
  - ...
- widgets
  - app_button.dart                       # Shared & reusable widgets                        # class AppButton {}
  - ...
```

- Models transform data via `fromJson`/`toJson` and `fromEntity`/`toEntity`
- Datasource interfaces live in `data/datasources/interfaces/` and are shared by both local and remote impls
- Repositories may have local, remote, or both datasources
- Entities are business objects used in repositories, usecases, and presentation
- Usecases group related operations into one file per domain (e.g. `user_usecases.dart` contains `GetUserUsecase`, `CreateUserUsecase`, `UpdateUserUsecase`, `DeleteUserUsecase`)
- Usecase params live in `domain/usecases/params/`; use `NoParam` for parameterless usecases
- Notifiers hold UI logic separate from screens; State classes hold UI state managed by the Notifier
- Components are widgets scoped to a screen folder; Widgets (`app_*`) are shared across screens

### Data Flow

**Read (simple):** Notifier → Repository → Datasource → DB/Network
**Read (complex):** Notifier → Usecase → Repositories → Datasources → DB/Network
**Write (simple):** Notifier → Repository → Datasource → DB/Network
**Write (complex):** Notifier → Usecase → Repositories → Datasources → DB/Network

### Implementation Approach

- **Never use the simplest approach** — always use the correct, well-structured approach that follows the existing codebase patterns.
- **Before implementing any task**, search the codebase for a similar existing pattern and follow it exactly. If a screen, notifier, or service already does something similar, replicate its structure.
- **Never access repositories or datasources directly from UI (screens/widgets)**. Always go through a Notifier. Even for simple reads, create a notifier if one doesn't exist.
- **Every screen that loads data** should have a corresponding notifier+state. No inline `ref.read(repoProvider)` from screens.
- **Follow existing naming conventions** — match verb patterns already used in the codebase (e.g. if existing notifiers use `loadX`, don't introduce `fetchX`).

### Usecase Rules

- Usecases are grouped by domain in a single `*_usecases.dart` file (e.g. `user_usecases.dart`, `auth_usecases.dart`).
- Each operation is its own class extending `Usecase<Result, Params>`.
- Use `NoParam` from `lib/domain/usecases/params/no_param.dart` for parameterless calls.
- Notifiers instantiate usecases on-demand: `await GetUserUsecase(repo).call(id)`.

## State Management

**Riverpod** with Notifier/State pattern:

- `NotifierProvider<T, S>` for global state
- `AutoDisposeNotifier` for ephemeral state (forms, dialogs) that should clean up automatically

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elrizwiraswara/flutter_pos](https://github.com/elrizwiraswara/flutter_pos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
