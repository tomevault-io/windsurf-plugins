---
trigger: always_on
description: IceMilkTea は Unity 6 (6000.3.x) 向けのカーネルフレームワーク。ゲーム基板を構築するための基盤ライブラリで、Service Locator パターンと PlayerLoopSystem によるカスタムアップデートを提供する。UPM パッケージ (`jp.sinoa.icemilktea`) として配布される。
---

## プロジェクト概要

IceMilkTea は Unity 6 (6000.3.x) 向けのカーネルフレームワーク。ゲーム基板を構築するための基盤ライブラリで、Service Locator パターンと PlayerLoopSystem によるカスタムアップデートを提供する。UPM パッケージ (`jp.sinoa.icemilktea`) として配布される。

## 技術スタック

- **Unity**: 6000.3.16f1（`ProjectSettings/ProjectVersion.txt`。package.json の最小対応バージョンは `6000.0`）
- **言語**: C# 9.0 (厳密準拠)
- **外部依存**: なし（`UnityEngine.*`, `Unity.*`, `System.*` のみ使用可。UniTask, UniRx, VContainer 等は禁止）
- **unsafe コード**: 許可（asmdef で `allowUnsafeCode: true`）
- **Enter Play Mode**: Domain Reload / Scene Reload を両方無効化（`ProjectSettings/EditorSettings.asset` の `m_EnterPlayModeOptions: 3`）。再生開始の高速化のため

## テスト

- **フレームワーク**: NUnit 3 (Unity Test Framework)
- **テスト場所**: `Packages/IceMilkTea/Tests/Editor/` を予定（現在 `Tests/` ディレクトリ・テスト用 asmdef ともに未作成）
- **実行方法**: Unity Editor の Test Runner ウィンドウから実行（Editor テストのみ）
- **InternalsVisibleTo**: DEBUG ビルド時に `IceMilkTeaEditor`, `IceMilkTeaTestDynamic`, `IceMilkTeaTestStatic` へ内部公開（3アセンブリとも現在は未作成の将来向け宣言）

## アーキテクチャ

パッケージのルートは `Packages/IceMilkTea/`

### ディレクトリ構成
```
Packages/IceMilkTea/
├── Runtime/
│   ├── Core/           # ユーティリティ・汎用機能
│   ├── Kernel/         # ゲームフレームワーク中核
│   ├── AssemblyInfo.cs
│   └── Unity.IceMilkTea.asmdef
├── package.json
└── LICENSE.md
```

パッケージ外の `Assets/Scripts/Sample/`（アセンブリ `IceMilkTea.Sample`）に起動フローの実例がある: `SampleEntryPoint.cs` が `[GameMain]` 属性付き静的メソッドから `new SampleGameMain().Run()` を呼び出し、`SampleGameMain` / `SampleService` が `GameMain` / `GameService` の最小実装例になっている。

### アセンブリ構成
- **Runtime**: `IceMilkTea` — `Packages/IceMilkTea/Runtime/`
- **名前空間**: `IceMilkTea.Core`（全ソースファイル共通）

### コア設計

**サービス駆動アーキテクチャ**: ゲームロジックは `MonoBehaviour` を継承しない純粋な C# クラス（サービス）として定義する。サービスは `GameMain` が保持する `GameServiceManager` に登録され、`GameServiceManager` 経由で取得・制御される。

**PlayerLoop 注入**: サービスの定期実行は `MonoBehaviour.Update` を使わず、`ImtPlayerLoopSystem` を通じて Unity の `PlayerLoopSystem` ツリーに直接注入する。`GameServiceUpdateTiming` enum（`[Flags] UInt32`）で24種類のタイミングポイントを定義。`GameServiceManager.Startup()` は登録済みサービスが実際に使用するタイミングのみを PlayerLoop に注入する（未使用タイミングは注入しない）。

**Domain Reload 無効対応**: 本プロジェクトは Enter Play Mode Options で Domain Reload / Scene Reload を無効化しているため、static フィールドは Play Mode をまたいで残存する。`GameMain`・`ImtPlayerLoopSystem`・`ImtAwaitableUpdateBehaviourScheduler` は `[RuntimeInitializeOnLoadMethod(SubsystemRegistration)]` を用いて Play Mode 開始時に静的状態をリセット（`GameMain.Current` クリア、`Awaiter` のスケジューラ初期化）またはイベント再購読（`Application.quitting`）する。`GameMain.InternalShutdown()` は Play Mode 終了時に `Current` を `try/finally` でクリアする（`GameMain` の SubsystemRegistration リセットはこれが失敗した場合に備えた `#if UNITY_EDITOR` 限定の保険。`ImtAwaitableUpdateBehaviourScheduler` のリセットも `#if UNITY_EDITOR` 限定）。新たに static フィールドを追加する場合は Play Mode 間の残存で問題が出ないか確認し、必要なら同様のリセット処理を設けること。

**主要クラスの関係**:
- `GameMain` — 純粋な抽象 C# クラス。アプリケーションのエントリポイント。利用者が `[GameMain]` 属性を付与した静的メソッドから `new MyGameMain().Run()` を呼び出して明示的に起動する。`GameMain.Current` でシングルトンアクセス。`ServiceManager` プロパティで `GameServiceManager` を保持。`Config` プロパティで `IGameConfig` を保持（`Run()` 以降は常に非 null、NullObject パターン）。起動順序: `Run()` → `CreateConfig()` → `new GameServiceManager()` → `RegisterHandler()` → `Startup()`（サービス登録）→ `ServiceManager.Startup()`（PlayerLoop 注入）。`Restart()` で `ServiceManager` のみ再起動可能（`ServiceManager.Shutdown()` → `OnRestart()` → `ServiceManager.Startup()` の順で実行）。virtual フック: `CreateConfig()`, `Startup()`, `OnRestart()`（既定では `Startup()` を再呼び出し）, `Shutdown()`, `Update()`
- `IGameConfig` — ゲームコンフィグのマーカーインターフェイス（空）。アプリケーション固有の設定はアプリ側でこのインターフェイスを実装して定義する
- `NullGameConfig` — `IGameConfig` の NullObject 実装（`internal sealed`）。`CreateConfig()` 未オーバーライド時のデフォルト値
- `GameService` — サービスの抽象基底クラス。`Startup(out GameServiceStartupInfo info)` で更新関数テーブルを登録、`Shutdown()` で終了処理
- `GameServiceManager` — サービスのライフサイクル管理。サービスは `GameMain.Startup()` 内で `AddService()` / `TryAddService()` により登録する必要がある（`ServiceManager.Startup()` 後に追加されたサービスの更新タイミングは PlayerLoop に反映されない）。API: `AddService()`, `TryAddService()`, `GetService<T>()`, `TryGetService<T>()`, `RemoveService<T>()`, `RemoveAllServices()`, `Exists<T>()`, `SetActiveService<T>()`, `IsActiveService<T>()`, `ServiceForEach()`, `ServiceProcessTime`（サービス処理時間の計測値プロパティ）
- `GameServiceStartupInfo` — サービス起動時に `UpdateFunctionTable`（`Dictionary<GameServiceUpdateTiming, Action>`）を設定する構造体
- `ImtPlayerLoopSystem` — `PlayerLoopSystem` 構造体をクラスとしてラップ。`Insert<T>()`, `Remove<T>()`, `Find<T>()`, `IndexOf<T>()`, `BuildAndSetUnityPlayerLoop()` で PlayerLoop ツリーを操作。`PlayerLoopSystem` との相互明示キャスト対応
- `PlayerLoopUpdater` — PlayerLoop で動作するアップデータの抽象基底クラス
- `MonoBehaviourEventBridge` — MonoBehaviour ライフサイクルイベント（Focus, Pause, EndOfFrame）をコールバックへ中継（`internal`、所在は `Runtime/Core/`）。いずれかのタイミングを使用するサービスが存在する場合のみ生成される
- `ImtGameServiceReferenceCache<T>` — サービス参照の遅延キャッシュ構造体
- `InsertTiming` — `BeforeInsert`, `AfterInsert` を持つ enum

**ユーティリティ (Core/)**:
- `ImtStateMachine<TContext, TEvent>` — ジェネリックステートマシン。イベント型を `int` に固定した派生 `ImtStateMachine<TContext>` もある

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sinoa/IceMilkTea](https://github.com/Sinoa/IceMilkTea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
