---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Flutter Todo app with Supabase backend, featuring OAuth authentication (Google/Kakao), local/cloud sync, notifications, and multi-platform support (Web, Android, iOS).

**Package**: `kr.bluesky.dodo`
**Current Version**: 1.0.17+56 (see pubspec.yaml)
**Latest Uploaded to Play Store**: 1.0.17+53

## Development Commands

### Running the App

```bash
# Web
flutter run -d chrome

# Android emulator
flutter run -d emulator-5554

# iOS simulator
flutter run -d <ios-simulator-id>

# Physical device (Samsung Galaxy example)
flutter run -d RF9NB0146AB

# List devices
flutter devices

# Hot reload (send to running Flutter process)
kill -SIGUSR1 <pid>

# Hot restart
kill -SIGUSR2 <pid>
```

### Build Commands

#### 플랫폼별 독립 버전 빌드 (권장)

**빌드 스크립트 사용** (플랫폼별 버전 자동 관리):

```bash
# Android 빌드 (기본값: 1.0.10+34)
./scripts/build_android.sh

# Android 커스텀 버전 빌드
./scripts/build_android.sh 1.0.11 35

# iOS 빌드 (기본값: 1.0.5+15)
./scripts/build_ios.sh

# iOS 커스텀 버전 빌드
./scripts/build_ios.sh 1.0.6 16
```

**자동 기능**:
- 버전 번호 파일명 자동 생성 (예: `app-release-1.0.11+35.aab`)
- pubspec.yaml 자동 백업 및 복원
- Clean 및 dependency 설치 자동화
- 상세한 빌드 로그 및 결과 표시

**상세 가이드**: [VERSION_MANAGEMENT.md](VERSION_MANAGEMENT.md)

#### 수동 빌드 (고급)

```bash
# Android Development APK
flutter build apk --debug

# Android Release (커스텀 버전)
flutter build apk --release --build-name=1.0.11 --build-number=35
flutter build appbundle --release --build-name=1.0.11 --build-number=35

# iOS Release (커스텀 버전)
flutter build ios --release --build-name=1.0.6 --build-number=16 --no-codesign

# Build outputs:
# Android:
#   - AAB: build/app/outputs/bundle/release/app-release.aab
#   - APK: build/app/outputs/flutter-apk/app-release.apk
# iOS:
#   - 추가로 Xcode에서 Archive 필요 (ios/Runner.xcworkspace)
```

**버전 관리 전략**:
- Android와 iOS는 독립적인 버전 번호 사용 가능
- 각 스토어별로 빌드 번호는 항상 증가해야 함
- **현재 빌드**: Android: **1.0.17+56** (AAB 준비), iOS: 1.0.5+15
- **마지막 업로드**: Android: **1.0.17+53** (Google Play), iOS: 1.0.5+15

**CRITICAL: 빌드 전 최신 업로드 버전 확인 필수**
```bash
# Google Play Console에서 최신 업로드된 빌드 번호 확인
# Settings > App integrity > App bundles > 최신 버전 번호 확인
#
# 현재 상황: Google Play Console에 1.0.17+53이 업로드되어 있음
# 새 빌드는 반드시 54 이상이어야 함
#
# WRONG: ./scripts/build_android.sh 1.0.18 52  # 52 < 53 (거부됨)
# RIGHT: ./scripts/build_android.sh 1.0.18 54  # 54 > 53 (승인됨)
```

**빌드 번호 규칙**:
- 새 빌드 번호는 반드시 Google Play에 업로드된 최신 빌드 번호보다 커야 함
- 빌드 전 항상 Google Play Console에서 최신 버전 확인
- 빌드 번호가 작으면 업로드 시 "Version code X has already been used" 오류 발생

**현재 상황**:
- Google Play Console: **1.0.17+53** (업로드됨)
- 다음 빌드: **1.0.17+56** 이상이어야 함 (AAB 빌드 완료)

**IMPORTANT**: 빌드 스크립트 사용 시 버전 번호가 포함된 파일이 자동 생성되므로 수동 복사 불필요. 수동으로 AAB 빌드할 때는 다음 명령어로:
```bash
flutter build appbundle --release --build-name=1.0.17 --build-number=56
cp build/app/outputs/bundle/release/app-release.aab build/app/outputs/bundle/release/app-release-1.0.17+56.aab
```

### Code Generation

```bash
# Generate code (Freezed, Drift, JSON Serializable)
dart run build_runner build --delete-conflicting-outputs

# Watch mode for continuous generation
dart run build_runner watch --delete-conflicting-outputs
```

### Testing & Analysis

```bash
# Run all tests
flutter test

# Run specific test file
flutter test test/unit/utils/recurrence_utils_test.dart

# Run specific test group
flutter test test/unit/utils/recurrence_utils_test.dart --plain-name "getNextOccurrences"

# Generate mock files (for tests using @GenerateMocks)
dart run build_runner build --delete-conflicting-outputs

# Analyze code
flutter analyze

# Check dependencies
flutter pub outdated
```

**Testing Standards**:
- **Date Convention**: All test dates use **2025 as the base year** for consistency and to avoid past-date issues
- **UTC DateTime**: Always use `DateTime.utc()` for RRULE-related tests (required by rrule package)
- **Test Structure**: Follow Arrange-Act-Assert pattern with clear group organization
- **Mock Setup**: Use mockito with `@GenerateMocks` annotation and fpdart's Either type

### Android Debugging

```bash
# Install APK to device
~/Library/Android/sdk/platform-tools/adb install -r build/app/outputs/apk/release/app-release.apk

# View logs
~/Library/Android/sdk/platform-tools/adb logcat

# View filtered logs (Flutter/Supabase/Auth)
~/Library/Android/sdk/platform-tools/adb logcat | grep -E "(flutter|kr.bluesky.dodo|OAuth|Supabase|Auth)"

# Clear logs
~/Library/Android/sdk/platform-tools/adb logcat -c

# Launch app
~/Library/Android/sdk/platform-tools/adb shell am start -n kr.bluesky.dodo/.MainActivity

# Uninstall app
~/Library/Android/sdk/platform-tools/adb uninstall kr.bluesky.dodo

# Take screenshot
~/Library/Android/sdk/platform-tools/adb exec-out screencap -p > screenshot.png
```

## Architecture

### Clean Architecture Layers

```
lib/
├── core/                    # Cross-cutting concerns
│   ├── config/             # Supabase, OAuth configuration
│   ├── router/             # GoRouter setup, auth guards
│   ├── services/           # Notifications, battery optimization
│   ├── theme/              # Colors, theming
│   └── utils/              # Logger, helpers
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bluesky78060/flutter-todo](https://github.com/bluesky78060/flutter-todo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
