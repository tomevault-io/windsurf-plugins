---
trigger: always_on
description: >-
---


## Gate

On skill activation, emit verbatim once:

> building-flutter-apps active. Pre-flight required.

Before writing any `.dart` code, emit verbatim:

> Reading building-flutter-apps gate.

After every code change to a `.dart` file (or to `pubspec.yaml` / `build.yaml` / `analysis_options.yaml`):

1. Run `dart analyze` from the package root. Block on any ERROR or WARNING.
2. Emit the filled-in Pre-Flight checklist. T0 always. T1 / T2 only if their domain was touched.
3. If `dart analyze` is not wired with `flutter_skill_lints`, run Setup before continuing.

## Critical Rules

1. **Use `dart analyze` from package root**, never `flutter analyze` and never path-scoped. Copy [references/analysis_options.yaml](references/analysis_options.yaml) to project root and wire `flutter_skill_lints` + `riverpod_lint` under `plugins:`. `flutter analyze lib` silently drops plugin diagnostics ([flutter#184190](https://github.com/flutter/flutter/issues/184190)).

2. **Use `@riverpod` / `@Riverpod` codegen for every provider** — state, computed, repository, datasource, service, family, stream. Never manual `Provider`, `FutureProvider`, `StreamProvider`, `StateProvider`, `StateNotifierProvider`, `NotifierProvider`, `AsyncNotifierProvider`, `ChangeNotifierProvider`. Run `dart run build_runner watch --delete-conflicting-outputs`.

3. **Guard every `await`** in notifiers and repositories with `if (!ref.mounted) return;`. Guard every `await` in widgets and `State` with `if (!context.mounted) return;`. Inside `State`, never `if (mounted)` — always `if (!context.mounted) return;`. **Inside `finally`, use the guard form `if (ref.mounted) { ... }`** — never `if (!ref.mounted) return;`.

4. **Extract widgets to public classes.** No `_buildXxx()` helpers. No `class _Foo extends StatelessWidget | StatefulWidget | ConsumerWidget | ConsumerStatefulWidget | HookWidget | HookConsumerWidget`. Mark file-internal widgets `@visibleForTesting`. `_FooState extends State<Foo>` stays private (Flutter convention — exempt).

5. **Use `Object?` or a specific type** for unknown values. `dynamic` only for `Map<String, dynamic>` JSON. Never `value!` — use `if (value case final v?)`.

6. **Use `AppLocalizations` (gen-l10n)** for every user-facing string. Never hardcode UI copy in widgets, notifiers, repositories, or datasources. In widgets, bind `final l10n = context.l10n;` at the top of `build` and use `l10n.someKey`; never chain `context.l10n.someKey`. `*Strings` constants only for non-user-facing IDs. For l10n config, put ARB files in `arb-dir` (`lib/l10n` by default). Generated Dart is written to `${arb-dir}/${output-localization-file}` unless `output-dir` is set; import `app_localizations.dart` from that directory.

7. **Use `sealed class` for Freezed unions and states.** Never `abstract class` with `@freezed`. Match with Dart native `switch` — never Freezed `.when()` / `.map()`. For VOs in `/domain/values/`, annotate `@Freezed(map: FreezedMapOptions.none, when: FreezedWhenOptions.none)` to disable codegen of those methods entirely. Lint: `freezed_disable_map_when_required`.

8. **Never prop-drill state.** Child widgets read providers directly with `ref.watch` / `ref.read` / `ref.listen`. Do not pass entity / state / notifier instances through constructors. Constructor params allowed: immutable IDs (for routing/lookup), callbacks, `Key`, and primitive props on leaf atoms. `ConsumerState` may own lifecycle handles, not provider-derived `*Cache`/`*Source`/`*DayStart` fields; use computed providers or local `build` values. Lint: `riverpod_consumer_state_derived_cache`.

9. **Use a mixin when the same behavior appears in 2+ classes.** Extract to a `mixin` with an `on` clause (e.g. `mixin RetryMixin on AsyncNotifier<X>`). Suffix the name with `Mixin`. Copy-paste sharing across notifiers, widgets, or services is forbidden — replace with a mixin.

10. **Storage SDK calls live in Local Datasource, never in Notifier.** Hive (`Hive.openBox`, `box.get/put/delete`, `Hive.box`), `SharedPreferences`, `flutter_secure_storage`, `dart:io` file ops, `path_provider` directory access — all live behind a `Local<X>Datasource` interface, called by `<X>Repository`. Notifiers and widgets never import `hive_ce` / `shared_preferences` / `flutter_secure_storage` / `dart:io` / `path_provider`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sgaabdu4/building-flutter-apps](https://github.com/sgaabdu4/building-flutter-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
