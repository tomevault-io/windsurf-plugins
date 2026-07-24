---
trigger: always_on
description: VITAP Student App is a Flutter mobile application for VIT-AP University students. It scrapes the VTOP student portal using a Rust library (via Flutter-Rust Bridge) and presents data — attendance, timetable, marks, exam schedule, profile — in a native app experience.
---

# Copilot Instructions

## Project Overview

VITAP Student App is a Flutter mobile application for VIT-AP University students. It scrapes the VTOP student portal using a Rust library (via Flutter-Rust Bridge) and presents data — attendance, timetable, marks, exam schedule, profile — in a native app experience.

## Build, Test, and Lint Commands

```bash
# Install dependencies
flutter pub get

# Run the app (build Rust first on initial setup)
cd rust && cargo build --release && cd ..
flutter run

# Lint / static analysis
flutter analyze

# Run all Flutter tests
flutter test

# Run a single Flutter test file
flutter test test/widget_test.dart

# Regenerate all code (ObjectBox, Riverpod, JSON, Freezed)
dart run build_runner build --delete-conflicting-outputs

# Watch mode for code generation during development
dart run build_runner watch

# Regenerate Flutter-Rust bridge bindings (run from project root)
flutter_rust_bridge_codegen generate

# Rust: format, lint, test
cd rust
cargo fmt
cargo clippy
cargo test
cargo test test_function_name   # single Rust test
```

## Architecture

### Flutter layer (`lib/`)

**Feature-based structure** — each feature lives in `lib/features/<feature>/` with four mandatory sub-layers. All cross-feature shared code lives in `lib/core/`.

Current features: `auth`, `attendance`, `timetable`, `course_page`, `digital_assignment`, `home`, `onboarding`, `account`, `vtop_webview`.

---

### Feature sub-layer guide

Follow this pattern **exactly** when creating a new feature. Every layer has a corresponding `.g.dart` generated companion; always run `build_runner` after changes.

#### `repository/`

One file per data source. Repositories are the only layer that calls VTOP (via Rust) or external HTTP APIs. They always return `Either<Failure, T>`.

**Pattern for VTOP-backed repositories:**
```dart
part 'my_remote_repository.g.dart';

@riverpod
MyRemoteRepository myRemoteRepository(Ref ref) {
  final vtopService = serviceLocator<VtopClientService>();
  return MyRemoteRepository(vtopService);
}

class MyRemoteRepository {
  final VtopClientService vtopService;
  MyRemoteRepository(this.vtopService);

  Future<Either<Failure, MyModel>> fetchData({
    required String registrationNumber,
    required String password,
    required String semSubId,
  }) async {
    try {
      final credentials = Credentials(
        registrationNumber: registrationNumber,
        password: password,
        semSubId: semSubId,
      );
      final result = await vtopService.executeWithRetry(
        credentials: credentials,
        operation: (client) => vtop.fetchMyData(client: client, semesterId: semSubId),
      );
      return Right(myModelFromJson(result));
    } on SocketException {
      return Left(Failure('No internet connection'));
    } on VtopError catch (rustError) {
      final msg = await VtopException.getFailureMessage(rustError);
      return Left(Failure(msg));
    } on FormatException catch (e) {
      debugPrint('JSON parsing failed: ${e.toString()}');
      return Left(Failure('Invalid response format from server'));
    } catch (e) {
      debugPrint('Error: ${e.toString()}');
      return Left(Failure('An unexpected error occurred. Please try again.'));
    }
  }
}
```

**Pattern for REST HTTP repositories** (e.g., For You feed):
```dart
@riverpod
MyRepository myRepository(Ref ref) {
  final client = serviceLocator<http.Client>();
  return MyRepository(client);
}

class MyRepository {
  final http.Client _client;
  MyRepository(this._client);

  Future<Either<Failure, MyModel>> fetchData() async {
    try {
      final response = await _client.get(Uri.parse('$_baseUrl/endpoint'), headers: _headers);
      if (response.statusCode == 200) {
        return Right(MyModel.fromJson(json.decode(response.body) as Map<String, dynamic>));
      }
      return Left(Failure('Request failed: ${response.statusCode}'));
    } catch (e) {
      return Left(Failure('Failed: ${e.toString()}'));
    }
  }
}
```

#### `model/`

Models used exclusively within a feature (not shared across features). If a model needs to be shared, it belongs in `lib/core/models/`.

There are two kinds of models:

**ObjectBox entity** (persisted locally — used for data that is part of the `User` aggregate):
```dart
part 'my_model.g.dart';

@Entity()
@JsonSerializable()
class MyModel {
  @Id()
  int? id;

  @JsonKey(name: 'field_name')
  final String fieldName;

  MyModel({this.id, required this.fieldName});

  factory MyModel.fromJson(Map<String, dynamic> json) => _$MyModelFromJson(json);
  Map<String, dynamic> toJson() => _$MyModelToJson(this);
}
```

**JSON-only model** (transient — not persisted, used for API response parsing):
```dart
part 'my_detail.g.dart';

List<MyDetail> myDetailFromJson(String str) =>
    List<MyDetail>.from((json.decode(str) as List).map((x) => MyDetail.fromJson(x as Map<String, dynamic>)));

@JsonSerializable()
class MyDetail {
  @JsonKey(name: 'field_name')
  final String fieldName;

  MyDetail({required this.fieldName});


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Udhay-Adithya/vitap_student_app](https://github.com/Udhay-Adithya/vitap_student_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
