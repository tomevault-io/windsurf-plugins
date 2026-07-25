---
trigger: always_on
description: > Bu dosya, projede iş yapan herkes (ve Claude) için **tek doğru kaynaktır**.
---

# CLAUDE.md — life_client Konvansiyonları

> Bu dosya, projede iş yapan herkes (ve Claude) için **tek doğru kaynaktır**.
> `.claude/skills/*` ve `.claude/agents/*` bu kurallara referans verir; kuralları
> tekrar etmez. Bir konvansiyon değişecekse önce burada değişir.

Proje: Hatay **life_client** — Flutter + Firebase mobil uygulaması.
Stack: Flutter, `flutter_riverpod` (v3, `@riverpod` codegen), `get_it`, `go_router` +
`go_router_builder`, Firebase (Firestore/Functions/Messaging), `hive_ce`,
`easy_localization`, `very_good_analysis`.

---

## 1. Mimari

**Feature-first + paylaşılan `product/` katmanı.**

```
lib/
├── core/            # DI (project_dependency*.dart), düşük seviye altyapı
├── features/        # ana feature'lar (her biri kendi viewmodel+view)
│   └── sub_feature/ # ikincil/modüler feature'lar
├── sub_feature/     # uygulama kabuğu (main_tab, onboard, ...)
├── product/         # paylaşılan: navigation, init, widget, utility, model
└── main.dart
```

Bir feature'ın iç yapısı:

```
lib/features/<feature>/
├── provider/                 # (veya view_model/) state mantığı
│   ├── <feature>_view_model.dart
│   ├── <feature>_view_model.g.dart    # üretilen (commit edilir)
│   └── <feature>_state.dart
├── view/
│   ├── <feature>_view.dart
│   ├── mixin/                # initState/dispose/iş mantığı mixin'leri
│   └── widget/               # parçalanmış alt widget'lar
```

İsimlendirme: `*_view_model.dart`, `*_state.dart`, `*_view.dart`, mixin `*_mixin.dart`,
model `*_model.dart`. Sınıf isimleri `XViewModel`, `XState`, `XView`, `XModel`.

---

## 2. State Management — Riverpod `@riverpod` Notifier

**Freezed YOK.** State = `Equatable` + manuel `copyWith()`. Async için `AsyncValue`
yerine **açık `isLoading` / `isFetching` / `isError` flag'leri**.

### ViewModel

```dart
part 'home_view_model.g.dart';

@riverpod
final class HomeViewModel extends _$HomeViewModel with ProjectDependencyMixin {
  @override
  HomeState build() {
    final categories = ref.read(productProviderState).categoryItems;
    return HomeState(categories: categories);
  }

  void changeHomeViewCardType() {
    state = state.copyWith(isGridView: !state.isGridView);
  }

  Future<void> fetchStoreModel(String id) async {
    state = state.copyWith(isFetching: true);
    final result = await firestoreService.getSingleData<StoreModel>(/* ... */);
    state = switch (result) {
      FirebaseSuccess(:final data) =>
        state.copyWith(storeModel: data, isFetching: false, isError: data == null),
      FirebaseFailure() => state.copyWith(isFetching: false, isError: true),
    };
  }
}
```

- `@riverpod final class XViewModel extends _$XViewModel with ProjectDependencyMixin`.
- `build()` initial state'i kurar (provider/cache'ten okuyarak).
- Mutasyon **daima** `state = state.copyWith(...)` ile. Asla doğrudan alan ataması yok.
- İstisnayı yutma; hata flag'ine çevir (`isError: true`).

Referans: [home_view_model.dart](lib/features/main/home/provider/home_view_model.dart),
[place_detail_view_model.dart](lib/features/details/view_model/place_detail_view_model.dart).

### State

```dart
final class HomeState extends Equatable {
  const HomeState({
    required this.categories,
    this.isGridView = false,
    this.isLoading = false,
  });

  final List<CategoryModel> categories;
  final bool isGridView;
  final bool isLoading;

  @override
  List<Object> get props => [categories, isGridView, isLoading];

  HomeState copyWith({
    List<CategoryModel>? categories,
    bool? isGridView,
    bool? isLoading,
  }) {
    return HomeState(
      categories: categories ?? this.categories,
      isGridView: isGridView ?? this.isGridView,
      isLoading: isLoading ?? this.isLoading,
    );
  }
}
```

- `final class XState extends Equatable`, tüm alanlar `final`, varsayılan değerli.
- `props` tüm alanları içerir. `copyWith` manuel.

Referans: [home_state.dart](lib/features/main/home/provider/home_state.dart).

---

## 3. Dependency Injection — GetIt + Riverpod hibrit

- Servisler/global provider'lar GetIt'te kurulur: [project_dependency.dart](lib/core/dependency/project_dependency.dart),
  `ApplicationInit.start()` içinde `ProjectDependency.setup()`.
- **ViewModel içinde** servise erişim → `ProjectDependencyMixin`
  ([project_dependency_mixin.dart](lib/core/dependency/project_dependency_mixin.dart)):
  `firestoreService`, `storageService`, `appProvider`, `productProvider`, `productCache`,
  `appProviderState`, `productProviderState`.

### Firebase servisleri (geçiş dönemi)

life_shared v5.4.17 ile Firebase erişimi ikiye ayrıldı; **iki servis bir süre yan yana yaşayacak**:

| | Yeni (kullan) | Eski (deprecated) |
|---|---|---|
| Firestore | `firestoreService` → `CustomFirestoreService` | `firebaseService` → `FirebaseCustomService` |
| Storage | `storageService` → `CustomStorageService` | `FirebaseStorageService()` (inline) |

- **Yeni yazılan her şey `firestoreService` / `storageService` kullanır.** Eski servis yalnızca
  henüz migrate edilmemiş çağrı yerleri için ayakta; yeni kodda kullanılırsa deprecation uyarısı verir.
- Yeni servisler `FirestoreResult<T>` / `StorageResult<T>` döner (`FirebaseSuccess` | `FirebaseFailure`).
  Hata artık yutulmuyor: timeout, permission, parse hataları `FirestoreError` / `StorageError`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VB-CORE/hatayi_yasat](https://github.com/VB-CORE/hatayi_yasat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
