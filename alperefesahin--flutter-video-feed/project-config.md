---
trigger: always_on
description: **Flutter Video Feed** - Ultra-performant TikTok-style video feed with **INTERACTIVE MEDIA FEED** and Clean Architecture.
---

# CLAUDE.md

## 🎯 Project Overview

**Flutter Video Feed** - Ultra-performant TikTok-style video feed with **INTERACTIVE MEDIA FEED** and Clean Architecture.

**Purpose:** Showcase project for personal branding and job opportunities (NOT production app)

**What Makes This Unique:**
- 🎥 Ultra-performant video feed (60 FPS, optimized memory)
- 🎮 **INTERACTIVE MEDIA FEED**: Hotspots, polls, quizzes on videos
- 🏗️ Clean Architecture (Feature-First, Layer-Within)
- 📱 Full responsive design system (no hardcoded values)

**Status:** Under active transformation - see PROJECT_STATUS.md

---

## 🏗️ Architecture: Feature-First, Layer-Within

### Structure
```
lib/
├── features/
│   └── video_feed/
│       ├── data/           # Firebase, external APIs
│       ├── domain/         # Business logic (pure Dart)
│       └── presentation/   # UI and state management
└── core/
    ├── design_system/
    ├── utils/extensions/
    └── di/
```

### Data Flow
```
User Input → Presentation → Domain → Data → Firebase
           ← Presentation ← Domain ← Data ← Response
```

### Layer Rules
- **Domain**: NO external dependencies (no Firebase, no Flutter, pure Dart)
- **Data**: Depends ONLY on Domain
- **Presentation**: Depends ONLY on Domain

---

## 📦 Key Technologies

**State Management:** flutter_bloc, equatable
**Error Handling:** fpdart (Either pattern)
**Video:** video_player, flutter_cache_manager, preload_page_view
**Backend:** Firebase (Firestore only)
**DI:** get_it
**Navigation:** go_router
**Caching:** cached_network_image (optional)

---

## 🎮 Interactive Media Feed (USP)

### What It Is
Videos with interactive overlays that users can tap:
- **Hotspots**: Tappable dots that reveal info
- **Polls**: Vote while watching
- **Quizzes**: Answer questions (optional)

### Data Structure
```dart
class InteractiveElement {
  final String type;           // 'hotspot', 'poll', 'quiz'
  final double x, y;           // Position (0.0-1.0, relative)
  final int showAtSecond;      // When to appear
  final Map<String, dynamic> data; // Element-specific content
}
```

### Implementation
- Position using relative coordinates (responsive)
- Show/hide based on video time
- Stack overlay on video player
- Simple, impressive implementation

---

## 🎨 Responsive Design System

**MANDATORY FOR ALL WIDGETS** - Zero hardcoded values allowed

**Reference Design:** iPhone 16 Pro Max (430×932)
**Core Class:** `lib/core/utils/helpers/adaptive_helper.dart`
**Extension:** `lib/core/utils/extensions/context_size_extensions.dart`

### AdaptiveHelper (Core)
```dart
class AdaptiveHelper {
  static const double designWidth = 430;
  static const double designHeight = 932;
  static const double fontScaleMin = 0.7;
  static const double fontScaleMax = 1.5;

  static double height(BuildContext context, double pixels);
  static double width(BuildContext context, double pixels);
  static double spacing(BuildContext context, double pixels);
  static double corner(BuildContext context, double pixels);
  static double text(BuildContext context, double pixels);
}
```

### Context Extension Methods (Use These)
```dart
// Dimensions
context.h(100)           // Responsive height
context.w(200)           // Responsive width
context.sq(24)           // Square dimension
context.fontSize(18)     // Responsive font (with min/max bounds)

// Spacing Widgets
context.hSpace(16)       // Vertical SizedBox
context.wSpace(20)       // Horizontal SizedBox
context.empty()          // SizedBox.shrink()

// Padding
context.paddingAll(16)
context.paddingHorizontal(12)
context.paddingVertical(8)
context.paddingTop(8)
context.paddingBottom(8)
context.paddingLeft(12)
context.paddingRight(12)
context.paddingNone       // EdgeInsets.zero

// Border Radius
context.radiusAll(12)
context.radiusTop(8)
context.radiusBottom(8)
context.radiusLeft(8)
context.radiusRight(8)
context.radiusTopLeft(8)
context.radiusTopRight(8)
context.radiusBottomLeft(8)
context.radiusBottomRight(8)

// Device Info
context.safeTop          // Safe area top
context.safeBottom       // Safe area bottom
context.screenWidth      // Total screen width
context.screenHeight     // Total screen height
```

### Usage Examples
```dart
// ✅ CORRECT - ALWAYS use responsive methods
Container(
  width: context.w(200),
  height: context.h(100),
  padding: context.paddingAll(16),
  decoration: BoxDecoration(
    borderRadius: context.radiusAll(12),
  ),
  child: Text(
    'Text',
    style: TextStyle(fontSize: context.fontSize(16)),
  ),
)

Icon(LucideIcons.heart, size: context.sq(24))

Column(
  spacing: context.h(8),
  children: [
    Text('A'),
    context.hSpace(16),
    Text('B'),
  ],
)

// ❌ FORBIDDEN - NO hardcoded values
Container(width: 200, height: 100)
SizedBox(height: 24)
SizedBox(width: 16)
Text('Text', style: TextStyle(fontSize: 16))
Icon(Icons.home, size: 28)
EdgeInsets.all(16)
BorderRadius.circular(12)
Padding(padding: EdgeInsets.symmetric(horizontal: 20))
```

---

## 💻 Code Patterns

### Repository Pattern
```dart
// Domain (interface)
abstract interface class VideoFeedRepository {
  Future<Either<String, List<VideoEntity>>> fetchVideos();
  Future<Either<String, List<VideoEntity>>> fetchMoreVideos();
}

// Data (implementation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alperefesahin/flutter_video_feed](https://github.com/alperefesahin/flutter_video_feed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
