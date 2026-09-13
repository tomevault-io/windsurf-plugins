---
trigger: always_on
description: 本文件是仓库的权威工程契约。修改代码或文档前必须先读。其他工具入口可以摘要本文件，但不得覆盖本文件的规则。
---

# Flutter AI Harness 项目契约

本文件是仓库的权威工程契约。修改代码或文档前必须先读。其他工具入口可以摘要本文件，但不得覆盖本文件的规则。

## 项目目标

本仓库是一套 AI 原生工程 Harness，也是一套有明确架构取向的 Flutter 混合应用工作区。AI Agent 和开发者共同使用相同的架构规则、任务产物、执行命令与质量门禁。

仓库分两个阶段建设：

1. 建立中立的 Harness 和仓库边界。
2. 通过 Harness 设计并实现全新 Demo，让任务卡、Review、App 文档和项目 Memory 从真实工作中自然产生。

不得从其他应用复制业务代码、凭据、历史任务或私有依赖到本仓库。

## 技术栈

### Flutter

- Flutter 3.41.9 / Dart 3.11
- 状态管理与轻量 DI：GetX 精简版 fork（公开 Git 源，固定 Commit）
- 路由：`go_router ^15.1.2`
- 网络：`dio ^5.7.0` + Protocol Buffers `^6.0.0`
- 本地存储：`drift ^2.20.0` + `flutter_secure_storage ^9.2.2`
- 不可变数据：Freezed `^3.0.0`
- JSON 序列化：`json_serializable ^6.9.0`
- 测试：`flutter_test` + `mocktail ^1.0.4` + `integration_test`
- Monorepo 编排：Melos

### 原生平台

- Android：Kotlin / Gradle
- iOS：Swift / Xcode / CocoaPods

Demo 固定采用本节技术栈，不在产品设计或任务拆解阶段重新选型。依赖在首个真实消费者出现时加入所属 Package 并锁定兼容版本，不为填充清单引入未使用依赖。

## 仓库结构

```text
app/
├── apps/demo/
└── packages/
    ├── app_core/
    ├── app_data/
    ├── app_ui/
    ├── app_features/
    ├── app_media/
    └── app_media_capture_bridge/
```

工作区随 Demo 实施逐步形成。不得为了填充目录而预先创建没有真实需求的业务抽象。

## 架构不变量

1. Domain Entity 或明确的 Value Object 是数据适配层进入业务层以及业务公共 API 之间唯一允许传递的数据类型。
2. `app_core` 只能定义并处理传输中立的 Request、Response、Failure 和不透明 Payload，不得定义、import 或解析具体 Fixture Payload、Proto Message 或数据库 Row；这些具体类型及其解析必须留在 `app_data`，不得进入 Feature、Controller 或 UI。`MediaResourceId` 是唯一批准的聚焦基础设施 ID 例外：它只提供传输中立的闭合格式校验，不包含路径、URI、Native handle、文件行为或业务规则；Store、Resolver、媒体 metadata 和预览 API 不得继续下沉到 `app_core`。
3. 每个包只定义自己需要的接口，不建立中央万能契约包。
4. 依赖方向保持单向。下图中 `A -> B` 表示 Package A 可以 import Package B：

   ```text
   apps/demo -> app_features, app_data, app_ui
   app_features -> app_data, app_core, app_ui, app_media, app_media_capture_bridge
   app_media -> app_core, app_ui
   app_media_capture_bridge -> 不依赖其他 Workspace Package
   app_data -> app_core
   app_core / app_ui -> 不依赖其他 Workspace Package
   ```

5. Feature 不得 import 其他 Feature 的内部实现。
6. 业务抽象接口放在 `app_features/lib/api/`，具体实现放在对应 `feature_xxx/api/`；跨 Feature 交互只依赖抽象接口，并由统一 Registry 绑定实现。
7. `app_data` 提供 Domain Entity、LocalDataSource、确定性 Fixture 及其 Transport，以及协议或持久化出现后的 Mapper/Adapter；不得承载页面、Controller 或 Feature 业务编排。
8. 壳工程只负责模块与回调装配，不得 import Feature 实现类。
9. Controller 通过构造函数接收必需 API。服务定位器只允许出现在装配点或显式全局服务中。
10. 只有在确实降低复杂度或保护真实边界时才新增抽象。
11. Native Consumer 直接依赖对应 Native Module；Flutter Consumer 通过聚焦的 Dart Client 和 Android/iOS Bridge Adapter 委托同一 Module。Host 只负责装配和注册，Native Module 不依赖 Flutter。

详细规则见 `docs/architecture.md` 和当前任务相关的 Skill。

## Flutter 默认约定

- 路由统一使用 `go_router`，根应用使用 `MaterialApp.router`。
- GetX 只用于状态管理和轻量 DI，不负责路由和 UI Overlay。
- GetX 必须使用 `https://github.com/bladeofgod/getx.git` 的精简版 fork，并固定到项目约定的完整 Commit；不得改回 pub.dev 官方版。
- 只使用公开且可复现的依赖；开源模板不得依赖私有或本机 fork。
- 即使存在服务定位器，也优先使用构造函数注入。
- 响应式刷新必须包裹读取状态的最小子树。
- `*.g.dart`、`*.freezed.dart` 和 Protobuf 生成文件只能由生成器修改。

依赖写入真实消费者所属的 Package `pubspec.yaml`；只有 Workspace 工具依赖写入根 `app/pubspec.yaml`。

Demo 当前没有真实远程 API 或 Wire Contract，业务数据使用确定性的本地 Fixture。首个真实消费者出现后，`app_core` 的 `ApiClient` 通过构造函数接收 `ApiTransport`；当前由 `app_data` 提供 `FixtureApiTransport`、LocalDataSource 和 Mapper，在进入业务 API 前把 Fixture Payload 转换为 Domain Entity。不得为模拟远程链路而引入 Dio、Proto 或伪造 HTTP Server；只有真实 Endpoint/协议成为事实来源后才能增加 `DioApiTransport` 和 Proto 生成链路。Drift 只在出现跨 App 重启持久化需求时引入。

## 混合工程 Bridge 契约

Android、iOS 都是长期维护的一等平台。

MethodChannel 和 EventChannel 必须遵守：

- 实现改动前先更新 `docs/bridge/` 下的契约。
- 使用可替换的反向域名命名空间，例如 `com.example.<module>.<feature>`。
- method、event type、error code 和枚举 wire 值使用小写 `snake_case`。
- payload key 可以使用 `lowerCamelCase`，但同一契约必须保持一致。
- 只传输 `String`、`num`、`bool`、`List`、`Map<String, dynamic>` 和 `Uint8List`。
- 禁止通过平台通道传递 Proto 对象。
- 错误使用 `PlatformException(code, message, details)`，`code` 必须是稳定字符串。
- Native 回调 Flutter 时必须切回平台 UI 线程。
- 所有声明支持的平台必须保持一致；有意差异必须写入契约。

详见 `docs/bridge/README.md` 和 `bridge-engineer` Agent。

## AI 工程资产

项目 Skill 的路径触发和路径级文件权限依赖 Claude Code 2.1.228 或更高版本。

只按当前任务加载必要文件：

- 工作流：`.claude/commands/*.md`
- 角色：`.claude/agents/*.md`
- 技能：`.claude/skills/*/SKILL.md`
- 低频工程经验：`.claude/memories/*.md`

`.claude/` 是 Command、Agent、Skill 和 Memory 的唯一事实来源。Codex 原生适配由仓库工具确定性生成：

- `AGENTS.md`：要求 Codex 完整读取本文件的薄入口。
- `.agents/skills/*/SKILL.md`：从 Claude Skill 和 Command 生成，使 Codex 支持 Skill 语义匹配与 `$skill-name` 显式调用。
- `.codex/agents/*.toml`：从 Claude Agent 生成，使 Codex 原生发现项目角色。

不得手工编辑带生成标记的适配文件。修改 `.claude` 事实源后运行 `make codex-adapters`；`make codex-adapters-check`、`pre-push` 和 `make harness-check` 会阻断缺失、过期或被篡改的适配。`.claude/memories/` 继续按任务读取，不批量注册为 Skill。

命名约定：

- Agent 用主体名词，例如 `architect`、`code-reviewer`、`task-executor`。
- Command 使用动宾结构，例如 `plan-tasks`、`review-changes`。
- Skill 使用聚焦领域名，例如 `go-router`、`testing-strategy`。

新增 Skill 必须包含 `name`、面向触发场景的 `description` 和相关 `paths`。`paths` 使用 glob 模式限定 Skill 适用的仓库路径；`description` 必须说明适用场景、不适用场景和触发关键词。

## 支持的工作流

- `/plan-tasks`：把产品或技术输入拆成任务卡。
- `/plan-figma`：结合 Figma 和代码上下文生成 UI 任务卡，不做实现。
- `/plan-spec`：人工明确安排 UI 自动化时，根据任务、产品规则或原型生成独立行为 Spec。
- `/execute-ui-spec`：人工显式选择 ready Spec 和平台后，执行静态审计与 App Operator 运行验证。
- `/execute-tasks`：执行已有任务卡并完成验证与 Review。
- `/review-changes`：只读审查当前改动并输出问题报告，不修改实现。
- `/review-security`：对明确任务或 diff 独立执行只读安全审查，不修改实现。
- `/review-batch`：按用户明确指定的任务和 diff 范围，只读审查跨任务影响。
- `/fix-review-findings`：在用户明确要求后，修复已有 Review 报告中的问题并复审。
- `/check-release`：执行发版前就绪检查。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bladeofgod/flutter-ai-harness](https://github.com/bladeofgod/flutter-ai-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
