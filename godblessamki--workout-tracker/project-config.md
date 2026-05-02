---
trigger: always_on
description: This document provides the architectural overview, development conventions, operational instructions, and the implementation plan for the WorkoutTracker project.
---

# Gemini Context: WorkoutTracker

This document provides the architectural overview, development conventions, operational instructions, and the implementation plan for the WorkoutTracker project.

## Overview

A native iOS app designed for tracking gym workouts accurately and efficiently. Users define reusable **Workout Routines** (e.g., "Chest + Back"), populate them with specific **Exercises**, then start **Sessions** where they log actual reps and weights. The app surfaces the personal record (max weight × reps) from the previous session to guide the user during their workout.

**Stack:** Swift 5.10+ · SwiftUI · SwiftData · MVVM

---

## Architecture & Project Structure

The project follows an MVVM (Model-View-ViewModel) architecture.

```text
WorkoutTracker/
├── WorkoutTrackerApp.swift         # App entry point, sets up ModelContainer
├── Models/                         # SwiftData @Model classes
│   ├── WorkoutRoutine.swift        # Routine blueprint
│   ├── RoutineExercise.swift       # Exercise within a routine
│   ├── WorkoutSession.swift        # A single workout session
│   └── LoggedSet.swift             # Data for a specific exercise set
├── ViewModels/                     # Observable classes, business logic
│   ├── RoutineViewModel.swift      # Manages routines and exercises
│   ├── SessionViewModel.swift      # (Planned) Manages session states
│   └── ActiveSessionViewModel.swift# (Planned) Logic for the active workout
└── Views/                          # SwiftUI Views
    ├── ContentView.swift           # Root tab/nav container
    ├── Dashboard/
    │   └── DashboardView.swift     # Main entry screen
    ├── Routines/
    │   ├── RoutineListView.swift   # Routine creation & editing
    │   ├── RoutineDetailView.swift
    │   └── AddExerciseView.swift
    └── Session/
        ├── ActiveSessionView.swift # Active workout logging
        └── ExerciseSetRowView.swift
```

---

## Building and Running

### Prerequisites
- macOS with Xcode 15.0+ installed.

### Commands
- **Open Project:** `open WorkoutTracker1.xcodeproj`
- **Build & Run:** Press `Cmd + R` in Xcode.
- **Run Tests:** Press `Cmd + U` in Xcode (Tests to be added to a dedicated Test target).

---

## Development Conventions

### Coding Style & Standards
- **Model Isolation:** Use SwiftData's `@Model` for all persistent entities. Ensure relationships are explicitly defined with appropriate delete rules (e.g., `.cascade` for parent-child ownership).
- **ViewModel Usage:** ViewModels should be `@Observable` and focus on mutation logic. Avoid redundant state that is already available in the `@Model` or via `@Query`. Receive `ModelContext` via method injection so they remain testable and decoupled from SwiftUI's environment.
- **View Pattern:** 
    - Use `@Query` for top-level data fetching.
    - Inject `ModelContext` via `@Environment(\.modelContext)`.
    - Pass models directly to subviews for better performance and simplicity.
- **SwiftUI:** Prefer modern SwiftUI primitives (e.g., `NavigationStack`, `TabView`, `ContentUnavailableView`).

### Data Integrity
- **Cascading Deletes:** Deleting a `WorkoutRoutine` must clean up all associated `RoutineExercise` and `WorkoutSession` records automatically, preventing orphan records in SwiftData.
- **Persistence:** All data is local-only using SwiftData. Completed sessions are never deleted to provide a rich dataset for progressive overload statistics.

---

## Implementation Plan

### Step 1 — SwiftData Schema Setup
> **Goal:** Define all four `@Model` classes with correct relationships so SwiftData can generate the full persistence layer automatically.

- **`WorkoutRoutine`**: `id`, `name`, `createdAt`, `exercises` (Cascade), `sessions` (Cascade).
- **`RoutineExercise`**: `id`, `name`, `order`, `routine`.
- **`WorkoutSession`**: `id`, `date`, `routine`, `loggedSets` (Cascade).
- **`LoggedSet`**: `id`, `reps`, `weight`, `exercise`, `session`.
- **`WorkoutTrackerApp.swift`**: Register all four models in the `ModelContainer`.

### Step 2 — Routine Creator View
> **Goal:** A screen that lets the user create named routines and add named exercises to each one, persisted immediately via SwiftData.

- **`RoutineListView`**: Shows all routines via `@Query`. "+" button to create a new routine.
- **`RoutineDetailView`**: Displays routine name and list of exercises. "Add Exercise" button. Swipe-to-delete.
- **`AddExerciseView`**: TextField for exercise name. Saves and increments order.
- **`RoutineViewModel`**: Handles `addRoutine`, `addExercise`, `deleteExercise` using injected `ModelContext`.

### Step 3 — Main Dashboard
> **Goal:** A landing screen listing every routine with a "Start Session" button that kicks off an active workout.

- **`DashboardView`**: `@Query` fetches routines. List rows with "▶ Start Session" button.
- **Navigation**: Tapping "Start Session" creates a new `WorkoutSession` and pushes `ActiveSessionView`.

### Step 4 — Active Session View *(Core Feature)*

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [godblessamki/Workout-Tracker](https://github.com/godblessamki/Workout-Tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
