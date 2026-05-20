---
trigger: always_on
description: enableSessionMemory: true                 // Always Allow/Block を記憶
---

# SwiftAgent

Apple FoundationModelsを基盤とした型安全で宣言的なAIエージェントフレームワーク。

> **Note**: デフォルトはApple FoundationModelsを使用。`--traits OpenFoundationModels` で OpenFoundationModels に切り替え可能。

## コア概念

| 概念 | 説明 |
|------|------|
| **Step** | `Input -> Output` の非同期変換単位。`run(_:)` を直接実装するか、`body` を定義して宣言的に合成 |
| **Session** | TaskLocalベースのセッション伝播（`@Session`, `.session()`） |
| **Memory/Relay** | Step間の状態共有（`@Memory` で保持、`$` で `Relay` を取得） |
| **Context** | 汎用TaskLocal伝播（`@Contextable`, `@Context`, `.context()`） |
| **Generate** | LLMによる構造化出力生成 |

## Step 一覧

| 種別 | Steps |
|------|-------|
| プリミティブ | `Transform`, `Generate`, `GenerateText`, `EmptyStep`, `Join`, `Gate` |
| 合成 | `Chain2-8`, `Pipeline`, `Parallel`, `Race`, `Loop`, `Map`, `Reduce` |
| 修飾 | `Monitor`, `TracingStep`, `AnyStep` |

## 基本パターン

```swift
// Session伝播（TaskLocal経由で自動伝播）
struct MyStep: Step {
    @Session var session: LanguageModelSession
    func run(_ input: String) async throws -> String {
        try await session.respond { Prompt(input) }.content
    }
}
try await MyStep().session(session).run("Hello")

// Memory/Relay による状態共有
struct OrchestratorStep: Step {
    @Memory var visitedURLs: Set<URL> = []  // 状態を保持

    func run(_ input: Query) async throws -> Result {
        // $visitedURLs で Relay を取得し、子Stepに渡す
        try await CrawlStep(visited: $visitedURLs).run(input.startURL)
    }
}

struct CrawlStep: Step {
    let visited: Relay<Set<URL>>  // 親からRelayを受け取る

    func run(_ input: URL) async throws -> CrawlResult {
        if visited.contains(input) { return .alreadyVisited }
        visited.insert(input)
        // クロール処理...
    }
}

// Context による汎用TaskLocal伝播（@Contextable マクロで簡潔に定義）
@Contextable
struct AppConfig: Contextable {
    static var defaultValue: AppConfig { AppConfig(maxRetries: 3) }
    let maxRetries: Int
}

struct MyStep: Step {
    @Context var config: AppConfig  // 型から自動でContextKeyを解決
    func run(_ input: String) async throws -> String { /* config.maxRetries を使用 */ }
}
try await MyStep().context(AppConfig(maxRetries: 5)).run("input")

// Step による宣言的な合成（body を定義すると run が自動実装）
struct TextPipeline: Step {
    @Session var session: LanguageModelSession
    var body: some Step<Prompt, String> {
        GenerateText<Prompt>(session: session)
    }
}

// 構造化出力
@Generable struct Output: Sendable {
    @Guide(description: "説明") let field: String
}
let step = Generate<String, Output>(session: session) { Prompt($0) }

// Tool定義（Arguments は @Generable、Output は不要）
struct MyTool: Tool {
    typealias Arguments = MyInput  // @Generable 必須
    let name = "my_tool"
    let description = "説明"
    func call(arguments: MyInput) async throws -> MyOutput { ... }
}
```

## Memory / Relay

Step間で状態を共有するためのプロパティラッパー。

| 型 | 用途 |
|---|------|
| `@Memory<Value>` | 値を参照型ストレージに保持。`$` で `Relay` を取得 |
| `Relay<Value>` | getter/setter クロージャによる間接アクセス |

```swift
// 基本的な使い方
@Memory var counter: Int = 0
counter += 1              // 値の変更
let relay = $counter      // Relay を取得
relay.wrappedValue = 10   // Relay 経由で変更

// コレクション拡張
@Memory var urls: Set<URL> = []
$urls.insert(url)         // Relay.insert
$urls.contains(url)       // Relay.contains
$urls.formUnion(newURLs)  // Relay.formUnion

@Memory var items: [String] = []
$items.append("item")     // Relay.append

// Int 拡張
@Memory var count: Int = 0
$count.increment()        // count += 1
$count.decrement()        // count -= 1
$count.add(5)             // count += 5

// Relay 変換
let doubled = $counter.map({ $0 * 2 }, reverse: { $0 / 2 })
let readOnly = $counter.readOnly { $0 * 2 }

// 定数 Relay
let constant = Relay<Int>.constant(42)  // 書き込み無視
```

## Pipeline / Gate

Stepの宣言的な合成とフロー制御を提供する。

| 型 | 用途 |
|---|------|
| `Pipeline` | `@StepBuilder` でStepを順番に実行するコンテナ |
| `Gate` | 入力を変換またはブロックするStep |
| `GateResult` | `.pass(value)` で続行、`.block(reason:)` で中断 |

```swift
// 基本的な Pipeline + Gate
Pipeline {
    // 入口ゲート：検証・変換
    Gate { input in
        guard !input.isEmpty else {
            return .block(reason: "Empty input")
        }
        return .pass(input.lowercased())
    }

    // メイン処理
    MyAgent()

    // 出口ゲート：後処理
    Gate { output in
        .pass(output.trimmingCharacters(in: .whitespaces))
    }
}

// 宣言的Step内での使用（body は既に @StepBuilder なので Pipeline 不要）
struct SecurePipeline: Step {
    @Session var session: LanguageModelSession

    var body: some Step<Prompt, String> {
        GenerateText<Prompt>(session: session)
        Gate { output in .pass(filterSensitive(output)) }
    }
}

// Pipeline が必要なケース：宣言的Stepの外で Step を合成
let step = Pipeline {
    Gate { .pass(validate($0)) }
    MyProcessingStep()
}
try await step.run(input)

// Gate ファクトリメソッド
Gate<String, String>.passthrough()           // 入力をそのまま通す
Gate<String, String>.block(reason: "Blocked") // 常にブロック
```

**GateError:**
- `GateError.blocked(reason:)` - ゲートがブロックした場合にスロー

## Event

型安全なイベント発火システム。`Notification.Name` 風の `EventName` と `@Context` で伝播する `EventBus` を使用。

| 型 | 用途 |
|---|------|
| `EventName` | 型安全なイベント名（`Notification.Name` 風） |
| `EventBus` | イベントの発火とリスナー管理（`@Contextable`） |
| `EventTiming` | `.before` / `.after` - イベント発火タイミング |

```swift
// イベント名の定義（アプリ側）
extension EventName {
    static let sessionStarted = EventName("sessionStarted")
    static let sessionEnded = EventName("sessionEnded")
}

// Step の .emit() モディファイア

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1amageek/SwiftAgent](https://github.com/1amageek/SwiftAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
