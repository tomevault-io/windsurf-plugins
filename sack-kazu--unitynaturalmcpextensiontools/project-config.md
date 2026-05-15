---
trigger: always_on
description: This package provides unified custom MCP (Model Context Protocol) tools that extend Unity Natural MCP server capabilities, enabling comprehensive automation of Unity Editor operations with consolidated functionality.
---

# Unity Natural MCP Extension Tools

This package provides unified custom MCP (Model Context Protocol) tools that extend Unity Natural MCP server capabilities, enabling comprehensive automation of Unity Editor operations with consolidated functionality.

## パッケージ概要

Unity Natural MCP サーバーの機能を拡張し、Unityエディタ操作の包括的な自動化を実現する統合型カスタムMCPツール群です。Scene管理、Prefab編集モード、スクリーンショット機能など、包括的な開発支援を提供します。

## ツール構成

### 実装済みツール

| ツール名 | 責務 | 主要メソッド | メソッド数 |
|---------|------|-------------|-----------|
| **McpUnifiedObjectTool** | オブジェクト作成・操作・プロパティ設定 | CreateObject, ManipulateObject, ConfigureComponent, RemoveComponent, GetObjectInfo, ListSceneObjects | 6 |
| **McpUnifiedAssetTool** | マテリアル・アセット・フォルダ管理 | ManageMaterial, AssignMaterialToRenderer, ListMaterials, ManageAsset, ListPrefabs, GetPrefabAssetInfo | 6 |
| **McpUnifiedEffectTool** | パーティクルシステム管理 | ConfigureParticleSystem, ControlParticleSystem, GetParticleSystemInfo | 3 |
| **McpSceneCaptureTool** | シーンスクリーンショット機能 | CaptureScene, CaptureGameView, ListCapturedScreenshots, CapturePrefabView | 4 |
| **McpPrefabEditTool** | Prefab編集モード管理 | OpenPrefabMode, SavePrefabMode, ClosePrefabMode, GetPrefabModeStatus, ApplyPrefabInstanceChanges, RevertPrefabInstanceOverrides, GetPrefabInstanceInfo, ListPrefabInstanceOverrides | 8 |
| **McpSceneManagementTool** | Scene作成・管理・操作 | CreateScene, SaveScene, LoadScene, ListScenes, GetActiveSceneInfo, CloseScene | 6 |
| **McpProjectSettingsTool** | プロジェクト設定管理 | ManageProjectLayers | 1 |
| **合計** | **全領域カバー** | **全34メソッド** | **34** |

### ディレクトリ構造

```
com.sack-kazu.unity-natural-mcp-extension-tools/
├── Editor/                     # MCPツール実装
│   ├── Editor.asmdef          # エディタアセンブリ定義
│   ├── McpUnifiedObjectTool.cs # 統合オブジェクト操作
│   ├── McpUnifiedObjectToolBuilder.cs
│   ├── McpUnifiedAssetTool.cs  # 統合アセット・マテリアル操作
│   ├── McpUnifiedAssetToolBuilder.cs
│   ├── McpUnifiedEffectTool.cs # 統合エフェクトシステム
│   ├── McpUnifiedEffectToolBuilder.cs
│   ├── McpSceneCaptureTool.cs  # シーンキャプチャ機能
│   ├── McpSceneCaptureToolBuilder.cs
│   ├── McpPrefabEditTool.cs    # Prefab編集モード管理
│   ├── McpPrefabEditToolBuilder.cs
│   ├── McpSceneManagementTool.cs # Scene作成・管理機能
│   ├── McpSceneManagementToolBuilder.cs
│   ├── McpProjectSettingsTool.cs # プロジェクト設定管理
│   ├── McpProjectSettingsToolBuilder.cs
│   ├── McpToolBase.cs          # 共通基盤クラス
│   ├── McpToolUtilities.cs      # 共通ユーティリティ
│   ├── ComponentPropertyManager.cs # 型変換・プロパティ設定管理
│   ├── ConfigurationModels.cs   # 設定モデル定義
│   └── McpConfigurationManager.cs # 設定解析・検証管理
├── Runtime/                    # ScriptableObjectアセット
│   ├── McpUnifiedObjectToolBuilder.asset
│   ├── McpUnifiedAssetToolBuilder.asset
│   ├── McpUnifiedEffectToolBuilder.asset
│   ├── McpSceneCaptureToolBuilder.asset
│   ├── McpPrefabEditToolBuilder.asset
│   ├── McpSceneManagementToolBuilder.asset
│   └── McpProjectSettingsToolBuilder.asset
└── package.json               # パッケージマニフェスト
```

## 実装パターン

### MCPツールクラスの基本構造

```csharp
[McpServerToolType, Description("ツールの説明")]
internal sealed class McpXxxTool : McpToolBase  // McpToolBaseを継承
{
    [McpServerTool, Description("メソッドの説明")]
    public async ValueTask<string> MethodName(
        [Description("パラメータ説明")] string param1,
        [Description("オプションパラメータ")] string param2 = null)
    {
        // ExecuteOperationによる統一されたエラーハンドリング
        return await ExecuteOperation(async () =>
        {
            // Prefabモード検証が必要な場合
            await ValidatePrefabMode(inPrefabMode);
            
            // GameObjectの安全な検索
            var gameObject = await FindGameObjectSafe(objectName, inPrefabMode);
            
            // Unity Editor API 操作
            EditorUtility.SetDirty(target);
            MarkSceneDirty(inPrefabMode);
            
            // 標準化されたメッセージ生成
            return McpToolUtilities.CreateSuccessMessage($"操作が完了しました");
        }, "operation name");
    }
}
```

### ツールビルダークラスの構造

```csharp
[CreateAssetMenu(menuName = "MCP/Tool Builder/Xxx", fileName = "McpXxxToolBuilder")]
public class McpXxxToolBuilder : McpBuilderScriptableObject
{
    protected override Type McpType => typeof(McpXxxTool);
}
```

## セットアップ

Unity Package Manager を使用してこのパッケージをインストールします。Unity Natural MCP サーバー起動時に、Runtime フォルダ内の ScriptableObject が自動的に読み込まれ、対応するツールがMCPサーバーに登録されます。

## 技術的詳細

### 共通基盤クラス

#### McpToolBase
提供される共通機能：
- `ExecuteOperation`: 標準化されたエラーハンドリング付き操作実行
- `ValidatePrefabMode`: Prefabモード検証
- `MarkSceneDirty`: コンテキストに応じたシーンのダーティマーク
- `FindGameObjectSafe`: 安全なGameObject検索
- `GetCurrentPrefabStage/GetContextRoot`: コンテキスト情報取得
- `LogSuccess/LogWarning`: 統一されたログ出力

#### McpToolUtilities
追加された機能：
- `CreateSuccessMessage/CreateErrorMessage`: 標準化されたメッセージ生成
- `GetContextDescription`: コンテキスト説明文生成
- `ValidateContext`: コンテキスト検証
- `FindComponent<T>`: ジェネリックコンポーネント検索

#### ComponentPropertyManager
型変換とプロパティ設定を統一管理：
- `SetProperty`: コンポーネントのプロパティ/フィールドを自動的に検出して設定
- `SetNestedProperty`: ドット記法によるネストされたプロパティの設定（例: "material.color"）
- `ResolveComponentType`: コンポーネント型の解決とキャッシュ機能

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sack-kazu/UnityNaturalMCPExtensionTools](https://github.com/sack-kazu/UnityNaturalMCPExtensionTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
