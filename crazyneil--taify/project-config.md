---
trigger: always_on
description: OllamaSpring 是一个使用 SwiftUI 构建的 macOS 应用程序，遵循 MVVM 架构模式。它是一个用于管理 Ollama 模型和创建对话式 AI 体验的综合性客户端。
---

# OllamaSpring 项目开发规范

## 项目概述
OllamaSpring 是一个使用 SwiftUI 构建的 macOS 应用程序，遵循 MVVM 架构模式。它是一个用于管理 Ollama 模型和创建对话式 AI 体验的综合性客户端。

## 架构模式
- **MVVM (Model-View-ViewModel)**: 严格遵循此模式
  - **Model**: 数据结构和业务逻辑（位于 `Model/` 目录）
  - **View**: SwiftUI 视图（位于 `View/` 目录）
  - **ViewModel**: 管理状态和业务逻辑的 ObservableObject 类（位于 `ViewModel/` 目录）

## 代码风格与命名规范

### 命名规则
- **类/结构体/枚举**: PascalCase（例如：`CommonViewModel`、`ChatModel`、`OllamaApi`）
- **变量/函数**: camelCase（例如：`selectedApiHost`、`loadPreference`、`fetchOllamaModels`）
- **常量**: 使用描述性名称的 camelCase（例如：`defaultApiHost`、`ollamaApiDefaultBaseUrl`）
- **私有属性**: 使用 `private` 关键字的 camelCase
- **文件名**: PascalCase，与主类/结构体名称匹配（例如：`CommonViewModel.swift`）

### 文件组织
```
OllamaSpring/
├── Model/              # 数据模型（结构体、类）
├── View/               # SwiftUI 视图
│   ├── Panel/         # 主面板视图
│   ├── Modal/          # 模态对话框视图
│   ├── Menu/           # 菜单相关视图
│   └── ScrollViewRow/  # 列表行视图
├── ViewModel/          # ViewModels（ObservableObject）
├── Network/            # API 客户端类
├── Database/           # Realm 模型和管理器
├── Utilities/          # 工具函数
└── Constant.swift      # 全局常量
```

## SwiftUI 模式

### 状态管理
- 使用 `@Published` 标记需要触发 UI 更新的 ViewModel 属性
- 使用 `@State` 标记本地视图状态
- 使用 `@Binding` 在父子视图之间进行双向数据绑定
- 将 ViewModels 传递给子视图时使用 `@ObservedObject`
- 在视图内创建 ViewModels 时使用 `@StateObject`
- 始终使用 `@MainActor` 标记 ViewModels 以确保线程安全

### 视图结构
```swift
struct ExampleView: View {
    @ObservedObject var viewModel: ExampleViewModel
    @State private var localState: Bool = false
    
    var body: some View {
        // 视图实现
    }
    
    // 子视图的私有计算属性
    private var someSubView: some View {
        // 实现
    }
    
    // 私有辅助函数
    private func helperFunction() {
        // 实现
    }
}
```

## ViewModel 指南

### 结构
```swift
@MainActor
class ExampleViewModel: ObservableObject {
    @Published var property: String = ""
    
    let manager = SomeManager()
    
    func asyncFunction() async {
        // 实现
    }
    
    func syncFunction() {
        // 实现
    }
}
```

### 最佳实践
- 所有 ViewModels 必须使用 `@MainActor` 标记
- 需要触发 UI 更新的属性使用 `@Published`
- 网络调用和异步操作使用 `async/await`
- 从后台线程更新 `@Published` 属性时使用 `DispatchQueue.main.async`
- 将业务逻辑保留在 ViewModels 中，而不是 Views 中
- 为管理器和服务使用依赖注入

## 网络层

### API 客户端模式
```swift
class ExampleApi {
    private var apiBaseUrl: String
    private var port: String
    
    init(apiBaseUrl: String? = nil, port: String? = nil) {
        // 使用默认值或提供的值初始化
    }
    
    private func makeRequest(
        method: String,
        endpoint: String,
        params: [String: Any] = [:]
    ) async throws -> AnyObject {
        // 请求实现
    }
    
    public func someApiCall() async throws -> AnyObject {
        return try await makeRequest(method: "POST", endpoint: "endpoint", params: [:])
    }
}
```

### 最佳实践
- 所有网络调用使用 `async/await`
- 使用 `do-catch` 块处理错误
- 使用 `NSLog` 记录错误日志
- 在适用时支持 HTTP 代理配置
- 返回 `AnyObject` 或特定的模型类型

## 数据持久化 (Realm)

### 模型定义
```swift
class RealmExample: Object {
    @Persisted(primaryKey: true) var id: String
    @Persisted var name: String
    
    convenience init(id: String, name: String) {
        self.init()
        self.id = id
        self.name = name
    }
}
```

### 管理器模式
```swift
class ExampleManager {
    func saveExample(example: Example) -> Bool {
        let realm = try! Realm(configuration: RealmConfiguration.shared.config)
        // 保存逻辑
    }
    
    func getExamples() -> Results<RealmExample> {
        let realm = try! Realm(configuration: RealmConfiguration.shared.config)
        return realm.objects(RealmExample.self)
    }
}
```

### 最佳实践
- 始终使用 `RealmConfiguration.shared.config` 创建 Realm 实例
- Realm 操作使用 `try!`（错误是致命的）
- Realm 模型使用 `convenience init`
- 属性使用 `@Persisted` 标记
- 查询使用 `Results<T>`

## 常量与配置

### 常量文件 (Constant.swift)
- 在 `Constant.swift` 中定义所有全局常量
- 使用带前缀的描述性名称（例如：`defaultApiHost`、`ollamaApiDefaultBaseUrl`）
- 使用注释将相关常量分组
- 不可变常量使用 `let`

### 示例
```swift
/// Api Host
let ApiHostList = [
    ApiHost(baseUrl: "http://localhost", port: 11434, name: "Ollama"),
    // ...
]
let defaultApiHost = ApiHostList[0].name
```

## 本地化

### 字符串键
- 使用点分隔的层次化键（例如：`"righttopbar.api_host"`、`"modal.cancel"`）
- 按功能/组件对键进行分组（例如：`righttopbar.*`、`modal.*`、`chatlist.*`）
- 在代码中始终使用 `NSLocalizedString(key, comment: "")`
- 将翻译添加到所有语言文件（en、zh-Hans、de、ja、ko、fr、es、ar）

### 示例
```swift
Text(NSLocalizedString("righttopbar.api_host", comment: ""))
```

### 本地化文件
- 位置：`OllamaSpring/{language}.lproj/Localizable.strings`
- 格式：`"key" = "translation";`
- 始终将新键添加到所有语言文件

## 文件头

### 标准文件头
每个 Swift 文件必须以以下内容开头：
```swift
//
//  FileName.swift
//  OllamaSpring
//
//  Created by NeilStudio on YYYY/MM/DD.
//
```

## 错误处理

### 网络错误
- 异步操作使用 `do-catch` 块
- 使用 `NSLog("错误消息: \(error)")` 记录错误
- 提供用户友好的错误消息
- 处理特定错误情况（超时、连接失败等）

### 示例
```swift
do {
    let response = try await api.someCall()
    // 处理成功
} catch {
    NSLog("操作期间出错: \(error)")
    // 处理错误
}
```

## 异步/等待模式

### ViewModel 方法
```swift
func fetchData() async {
    do {
        let data = try await api.fetch()
        DispatchQueue.main.async {
            self.data = data
        }
    } catch {
        NSLog("错误: \(error)")
    }
}
```

### 视图使用
```swift
Task {
    await viewModel.fetchData()
}
```

## UI 组件

### 模态视图
- 使用 `.sheet(isPresented:)` 进行模态展示

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CrazyNeil/Taify](https://github.com/CrazyNeil/Taify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
