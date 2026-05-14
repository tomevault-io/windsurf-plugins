---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 儲存庫概覽

Flutter Compositions 是一個 **Melos monorepo**，包含了受 Vue 3 Composition API 啟發的 Flutter 響應式框架。核心架構使用 **`alien_signals`** 實現細粒度響應式，並在 Flutter 的 `StatefulWidget` 之上構建了自定義 widget 系統。

### 套件結構

- **`packages/flutter_compositions/`** - 核心框架，包含響應式原語（`ref`、`computed`、`watch`）和 composables
- **`packages/flutter_compositions_lints/`** - 自定義 lint 規則，用於強制執行響應式最佳實踐
- **`packages/flutter_compositions/example/`** - Flutter 示範應用程式，展示框架功能

## 開發指令

### 初始設定

```bash
# Bootstrap monorepo（連結套件）
melos bootstrap
```

### 執行測試

```bash
# 在所有套件中執行測試
melos run test

# 測試特定套件
cd packages/flutter_compositions
fvm flutter test

# 測試特定檔案
fvm flutter test test/composables_test.dart

# Lints 套件使用 dart test（不是 flutter test）
cd packages/flutter_compositions_lints
fvm dart test test/lints/
fvm dart test test/lints/shallow_reactivity_warning_test.dart
```

### Linting 與分析

```bash
# 分析所有套件（包含 analysis_server_plugin lints）
melos run analyze
```

### 執行示範應用

```bash
cd packages/flutter_compositions/example
fvm flutter run
```

### CHANGELOG 維護

每次修改程式碼時，**必須**同步更新對應套件的 CHANGELOG.md：

- `packages/flutter_compositions/CHANGELOG.md`
- `packages/flutter_compositions_lints/CHANGELOG.md`

在 `## [Unreleased]` 下方新增條目，使用 [Keep a Changelog](https://keepachangelog.com/) 格式：`### Added`、`### Changed`、`### Fixed`、`### Removed`。

### 發佈新版本

```bash
# 使用 release script（會自動驗證 changelog、跑測試、更新版本、建立 commit 和 tag）
./scripts/release.sh <version>
# 例如: ./scripts/release.sh 0.2.4
```

## 核心架構

### 響應式系統

框架建立在三個層級上：

1. **`alien_signals`**（依賴項）- 底層響應式原語
   - `WritableSignal` → `Ref<T>`（可寫入的響應式值）
   - `Computed` → `ComputedRef<T>`（衍生值）
   - `Effect` → `watch`/`watchEffect`（副作用）

2. **`CompositionWidget`**（擴展 `StatefulWidget`）- Widget 生命週期整合
   - `setup()` 在 `initState()` 中**執行一次**，回傳一個 builder 函數
   - Builder 函數包裝在 `Effect` 中，當依賴項改變時重新執行
   - `_widgetSignal` 透過 `widget()` 擴展方法啟用響應式 props
   - `_SetupContext` 管理生命週期鉤子和 provide/inject

3. **Composables** - 可重用的組合函數（前綴：`use*`）
   - Controllers：`useScrollController()`、`useTextEditingController()`、`useFocusNode()`、`usePageController()`
   - Animations：`useAnimationController()`、`useSingleTickerProvider()`、`manageAnimation()`
   - Async：`useFuture()`、`useAsyncData()`、`useStream()`、`useStreamController()`
   - Framework：`useAppLifecycleState()`、`useSearchController()`、`useContext()`
   - InheritedWidget：`useMediaQuery()`、`useTheme()`、`useLocale()`、`useContextRef()`
   - Listenable：`useController()`、`manageListenable()`、`manageChangeNotifier()`、`manageValueListenable()`

### 關鍵實作細節

**Setup 執行流程：**
```
initState()
  → 創建 _SetupContext
  → 設定 parent context（用於 provide/inject）
  → 創建 _widgetSignal（用於響應式 props）
  → 在 effectScope 中執行一次 setup()
  → 儲存回傳的 builder 函數
  → 排程 onMounted 回呼在 post-frame 執行
```

**響應式更新流程：**
```
ref.value = newValue
  → Signal 通知訂閱者
  → Effects 在 microtask 中排隊（批次處理）
  → Builder effect 重新執行
  → 如果 widget tree 改變則呼叫 setState()
  → Flutter 重建
```

**Props 響應式：**
```
didUpdateWidget(oldWidget)
  → _widgetSignal.call(widget)
  → 依賴的 computed 重新計算
  → 如果使用了 props 則 builder 重新執行
```

### Provide/Inject 系統

使用 **parent chain**（不是 `InheritedWidget`）進行依賴注入：

- `_SetupContext._parent` 連結到最近的祖先 CompositionWidget 的 context
- O(d) 查找，其中 d = widget tree 深度
- 不會傳播重建 - refs 處理響應式
- 透過 `InjectionKey<T>` 實現類型安全（在相等比較中包含泛型類型）

### 生命週期鉤子

- **`onMounted(callback)`** - 在第一幀渲染後
- **`onUnmounted(callback)`** - 在 widget 釋放前（清理）
- **`onBuild(callback)`** - 每次 builder 執行（內部由 composables 使用）

鉤子儲存在 `_SetupContext` 中，並在適當的生命週期時刻觸發。

### Effect 管理

所有在 `setup()` 期間創建的 effects 都會透過 `effectScope` 自動追蹤：
- 註冊在 `_SetupContext._effectScope` 中
- 在 `dispose()` 中自動釋放
- `watch`、`watchEffect` 或 builder effects 不需要手動清理

## Custom Lints

### 測試方法

Lints 使用 **`analysis_server_plugin`** 搭配 **`analyzer_testing`** 測試框架：

```dart
@reflectiveTest
class MyRuleTest extends AnalysisRuleTest {
  @override
  void setUp() {
    rule = MyRule();
    super.setUp();
  }

  void test_bad_case() async {
    await assertDiagnostics(r'''
// inline Dart source
''', [lint(offset, length)]);
  }

  void test_good_case() async {
    await assertNoDiagnostics(r'''
// valid Dart source
''');
  }
}
```

**重要：** 測試使用 `@reflectiveTest` + `AnalysisRuleTest`，以 inline source strings 測試，不需要 fixture 檔案。

### Lint 規則

所有規則遵循命名慣例：`flutter_compositions_<rule_name>`

1. **`ensure_reactive_props`** - 強制在 `setup()` 中使用 `widget()` 存取 props
2. **`no_async_setup`** - 防止在 `setup()` 方法上使用 `async`
3. **`controller_lifecycle`** - 確保 controllers 使用 `use*` helpers 或手動釋放
4. **`shallow_reactivity`** - 警告淺層響應式限制，直接修改屬性或陣列項目不會觸發更新
5. **`no_conditional_composition`** - 防止條件式呼叫 composition APIs
6. **`no_logic_in_builder`** - 防止在 builder 函數中放置邏輯（除了 props 解構）
7. **`prefer_raw_controller`** - 建議在 builder 中使用 `.raw` 而非 `.value` 存取 controllers

## 常見模式

### 創建 Composables

Composables 是使用 composition APIs 並回傳響應式值的函數：

```dart
(Ref<int>, void Function()) useCounter({int initialValue = 0}) {
  final count = ref(initialValue);
  void increment() => count.value++;

  // 如需清理
  onUnmounted(() {
    // 清理程式碼
  });

  return (count, increment);
}
```

**命名慣例：** 所有 composables 必須以 `use` 前綴開頭。

### 異步操作

**異步資料的兩階段模式：**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yoyo930021/flutter_compositions](https://github.com/yoyo930021/flutter_compositions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
