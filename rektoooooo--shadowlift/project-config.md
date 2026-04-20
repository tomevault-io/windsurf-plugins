---
trigger: always_on
description: **Gymly** is a production-ready iOS fitness tracking application built with 100% native Apple technologies. This is a sophisticated, performance-optimized workout logging and planning app with ~17,600 lines of Swift code, currently in active TestFlight beta.
---

# Gymly - Claude Code Context Document

## Project Overview

**Gymly** is a production-ready iOS fitness tracking application built with 100% native Apple technologies. This is a sophisticated, performance-optimized workout logging and planning app with ~17,600 lines of Swift code, currently in active TestFlight beta.

**Critical Context:**
- **Zero external dependencies** - Pure Apple frameworks only
- **Real users in production** - Performance and UX are critical
- **Active development** - Recent major performance fixes for "gym lag" issue
- **Premium features** - Some features behind premium flag (currently enabled for testing)

### Key Stats
- **Language:** Swift 5.9+
- **Platform:** iOS 17.0+ (iOS 26+ for AI features)
- **Architecture:** MVVM with SwiftUI + SwiftData
- **Lines of Code:** ~17,615 lines
- **Developer:** Sebastián Kučera
- **Bundle ID:** com.icservis.GymlyFitness

## Technology Stack

### Core Frameworks (All Native Apple)
```
SwiftUI           - 100% declarative UI (no UIKit)
SwiftData         - Modern persistence (replaces Core Data)
HealthKit         - Weight, BMI, height, workout data sync
CloudKit          - iCloud cross-device sync
AuthenticationServices - Apple Sign In
FoundationModels  - iOS 26+ on-device LLM for workout summaries
Combine           - Reactive data flow
PhotosUI/PhotoKit - Progress photo management
AVFoundation      - Camera for progress photos
Network           - Network quality monitoring for intelligent sync
```

### Important: NO Third-Party Dependencies
- No CocoaPods, SPM packages, or external frameworks
- All functionality built with Apple's native APIs
- Simplifies debugging and ensures App Store compatibility

## Directory Structure & Key Files

### Critical Entry Points
```
GymlyApp.swift              - App entry, ModelContainer setup, file import handling
ToolBar.swift               - Main TabView (Routine/Calendar/Settings)
Config.swift                - Global app state singleton (ObservableObject)
Logic/WorkoutViewModel.swift - Core business logic (1,000+ lines)
```

### Models (SwiftData) - `/Models/`
```
Exercise.swift              - Exercise with nested Set model
Day.swift                   - Workout day
Split.swift                 - Workout routine/split
DayStorage.swift            - Calendar-based completed workout storage
UserProfile.swift           - User profile, streak tracking
ProgressPhoto.swift         - Progress photo tracking
SplitTemplate.swift         - Pre-built templates (PPL, PHAT, etc.)
WeightPoint.swift           - HealthKit weight data
GraphEntry.swift            - Muscle group analytics
FitnessProfile.swift        - User goals & preferences
MuscleGroup.swift           - UI muscle group grouping
```

### ViewModels & Business Logic - `/Logic/`
```
WorkoutViewModel.swift      - CENTRAL BUSINESS LOGIC (read this first!)
iCloudSyncManager.swift     - CloudKit sync coordination
```

### Managers - `/Managers/`
```
UserProfileManager.swift    - Profile operations, streak calculations
AppearanceManager.swift     - Theme & accent color management
PhotoManager.swift          - Progress photo handling
```

### Views by Feature

**Workout** (`/Workout/`) - 20 files
```
TodayWorkoutView.swift      - Main workout interface
ExerciseDetailView.swift    - Exercise tracking with sets
ShowSplitDayView.swift      - Day-specific workout display
SplitsView.swift            - Split management
SplitTemplatesView.swift    - Pre-built template selection
WorkoutSummaryView.swift    - Post-workout analytics
CreateExerciseView.swift    - Add exercise flow
EditExerciseView.swift      - Edit exercise metadata
ProgressPhotoTimelineView.swift   - Photo gallery
```

**Calendar** (`/Calendar/`) - 3 files
```
CalendarView.swift          - Workout history calendar
CalendarDayView.swift       - Individual day view
CalendarExerciseView.swift  - Historical exercise viewer
```

**Settings** (`/Settings/`) - 18 files
```
NewSettingsView.swift       - Main settings interface
ProfileView.swift           - User profile
HealthKitManager.swift      - HealthKit operations
ConnectionsView.swift       - HealthKit integration UI
FitnessProfileSetupView.swift - Onboarding
AISummary/                  - 5 files for AI workout analysis
  ├── WorkoutSummarizer.swift
  ├── AISummaryView.swift
  └── ...
```

**CloudKit** (`/CloudKit/`) - 2 files
```
CloudKitManager.swift       - CloudKit operations
CloudKitSyncStatus.swift    - Sync status tracking
```

**Reusable Components** (`/Cells/`) - 9 files
```
SetCell.swift               - Set display
SetWeightCell.swift         - Weight input
SetRepetitionsCell.swift    - Reps input
SetTypeCell.swift           - Set type toggles
BMIGaugeView.swift          - BMI visualization
WeightChart.swift           - Weight progress chart
```

## Core Data Models (SwiftData)

### Hierarchical Relationship
```
Split (1) ──cascade──> Day (many) ──nullify──> Exercise (many) ──cascade──> Set (many)
```

### Key Model: Exercise (with nested Set)
```swift
@Model class Exercise: ObservableObject, Codable {
    @Attribute(.unique) var id: UUID

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rektoooooo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
