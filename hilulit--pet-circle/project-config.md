---
trigger: always_on
description: State management architecture for Pet Circle. Covers store patterns, access conventions, seeding from mock data, and how screens should read/write app state.
---


# Pet Circle -- State Management Rules

## Architecture Overview

Pet Circle uses **global `ChangeNotifier` stores** for all shared app state. Global preferences (locale, dark mode) use `ValueNotifier`. No third-party state management packages are required.

```
lib/stores/
├── pet_store.dart            # Pet profiles, active pet, care circle ops
├── measurement_store.dart    # SRR measurements per pet
├── note_store.dart           # Clinical notes per pet
├── medication_store.dart     # Medications per pet
├── notification_store.dart   # In-app notifications
├── user_store.dart           # Current user and role
└── settings_store.dart       # Thresholds, preferences
```

## Store Pattern

Every store follows the same structure:

```dart
import 'package:flutter/foundation.dart';

class ExampleStore extends ChangeNotifier {
  List<Item> _items = [];

  List<Item> get items => List.unmodifiable(_items);

  void addItem(Item item) {
    _items.add(item);
    notifyListeners();
  }

  void seed(List<Item> initial) {
    _items = List.of(initial);
    notifyListeners();
  }
}
```

### Rules

- IMPORTANT: Stores extend `ChangeNotifier`, **not** `ValueNotifier`
- IMPORTANT: All public getters return **unmodifiable** views (`List.unmodifiable`, `Map.unmodifiable`)
- IMPORTANT: Every mutation method MUST call `notifyListeners()` at the end
- State fields are private (`_items`), exposed only through getters
- Stores live in `lib/stores/`, one store per file, `snake_case.dart`
- Each store has a `seed()` method for initialization from mock data

## Global Store Instances

Each store file exports its own global singleton instance:

```dart
// In lib/stores/pet_store.dart:
final petStore = PetStore();

// In lib/stores/measurement_store.dart:
final measurementStore = MeasurementStore();
```

Stores are seeded in `main()` before `runApp()` via `_seedMockStores()` (only when `kEnableFirebase == false`). When Firebase is enabled, `PetStore` subscribes to Firestore streams via `subscribeForUser(uid)` in `AuthGate` and coordinates Firestore subcollection subscriptions for measurements, notes, and medications.

## Accessing Stores in Widgets

### Reading (rebuilds on change)

Use `ListenableBuilder` to rebuild a widget subtree when a store changes:

```dart
@override
Widget build(BuildContext context) {
  return ListenableBuilder(
    listenable: petStore,
    builder: (context, _) {
      final pets = petStore.ownerPets;
      return ListView(
        children: pets.map((p) => PetCard(pet: p)).toList(),
      );
    },
  );
}
```

For multiple stores, use `Listenable.merge`:

```dart
ListenableBuilder(
  listenable: Listenable.merge([petStore, measurementStore]),
  builder: (context, _) {
    // Rebuilds when either store changes
  },
)
```

### Writing (mutations)

Call store methods directly -- no context needed:

```dart
onPressed: () {
  measurementStore.addMeasurement(petStore.activePet!.id!, Measurement(
    bpm: calculatedBpm,
    recordedAt: DateTime.now(),
  ));
}
```

### One-time reads (no rebuild)

Access store getters directly when you don't need reactive rebuilds:

```dart
final currentUser = userStore.currentUser;
final activePet = petStore.activePet;
```

## Import Convention

```dart
import 'package:pet_circle/stores/pet_store.dart';
import 'package:pet_circle/stores/measurement_store.dart';
```

The global instance (e.g. `petStore`) is exported from the store file itself, NOT from `main.dart`.

## Store Registry

| Store | Global | Key Methods |
|-------|--------|-------------|
| `PetStore` | `petStore` | `addPet`, `createPetWithFirestore`, `updatePet`, `updatePetWithFirestore`, `removePet`, `removePetWithFirestore`, `getPetByName`, `getPetById`, `activePet`, `activePetIndex`, `setActivePetIndex`, `currentUserRoleFor`, `removeCareCircleMember`, `removeCareCircleMemberWithFirestore`, `subscribeForUser`, `cancelSubscription` |
| `MeasurementStore` | `measurementStore` | `addMeasurement`, `removeMeasurement`, `getMeasurements`, `latestForPet`, `countForPet`, `thisWeekCount`, `subscribeForPets`, `cancelSubscriptions` |
| `NoteStore` | `noteStore` | `addNote`, `getNotes`, `subscribeForPets`, `cancelSubscriptions` |
| `MedicationStore` | `medicationStore` | `addMedication`, `updateMedication`, `removeMedication`, `toggleMedication`, `getMedications`, `getActiveMedications`, `subscribeForPets`, `cancelSubscriptions` |
| `NotificationStore` | `notificationStore` | `seed`, `reset`, `subscribeForUser`, `cancelSubscription`, `addNotification`, `markRead`, `markAllRead`, `unreadCount` |
| `UserStore` | `userStore` | `seed`, `seedFromAppUser`, `setUser`, `setRole`, `currentUser`, `appUser`, `role`, `isVet`, `isOwner`, `currentUserUid`, `currentUserEmail`, `currentUserDisplayName`, `currentUserAvatarUrl` |
| `SettingsStore` | `settingsStore` | `seedFromAppUser`, `reset`, `updateThresholds`, `setPushNotifications`, `togglePushNotifications`, `setEmergencyAlerts`, `toggleEmergencyAlerts`, `setVisionRREnabled`, `toggleVisionRR`, `setAutoExport`, `toggleAutoExport`, `classifyStatus` |

## Providers

| Provider | Global | Purpose |
|----------|--------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HiLuLiT/pet-circle](https://github.com/HiLuLiT/pet-circle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
