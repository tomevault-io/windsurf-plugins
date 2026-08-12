---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 文档指引
/home/han/code/linglong-store/linglong-server 这个是后端代码，你在对接接口的时候需要参考。


## 重点（极其重要）
- 未经允许，禁止使用git worktree功能。
- 所有的业务细节都要落实到文档里面去，详细的细节文档，docs目录
- 当前项目要求绝对的高性能，高UI响应速度。
- 严禁使用 `PopupMenuButton`。按钮和设置字段展开的轻量菜单必须统一复用
  `AppAnchoredMenu<T>` / `AppAnchoredMenuButton<T>`；右键上下文操作继续使用项目既有
  原生菜单入口，禁止以 `showMenu` 或页面私有 Overlay 规避此约定。
- 严禁硬编码物理方向做布局对齐：禁止新增 `Alignment.centerLeft/centerRight/topLeft` 等、
  `EdgeInsets.only(left:/right:)`、`TextAlign.left/right`、`Positioned(left:/right:)`，
  必须使用方向感知写法（`AlignmentDirectional` / `EdgeInsetsDirectional` /
  `TextAlign.start/end` / `PositionedDirectional`），否则 RTL（阿拉伯语）下布局不镜像；
  方向性图标（如展开箭头）按 `Directionality` 镜像。CI 门禁
  `build/scripts/verify_directional_layout.dart` 会拦截违规；确属窗口物理几何等合理豁免时，
  在源码加 `// ignore: hardcoded_direction` 注释说明理由。
- 每开发一个功能点就进行一次commit
- Git commit 必须遵循 Conventional Commits，统一使用 `type: 简短描述`，不要再写无类型前缀的自然语句提交信息。
- 在接到用户的任务的时候，先不要着急开始修改代码，要先分析需求，分析代码，列举解决方案，
- 详细的向用户说明你的思路，和你打算如何实现这个需求。
- 要分析整个项目的架构，一切都要从整个项目的角度入手，不能直接看完一个文件就写代码。
- 先问清楚、绝对不允许猜测：遇到需求或现状不确定时，先明确提问，不要主观假设；方案需先得到用户确认再开工。
- 每一处代码修改都要有必要的注释
- 先方案后编码：先梳理背景/现状 → 列备选方案（含改动面、影响范围、取舍理由）→ 让用户确认 → 再动手。**只有在用户确认你的方案后，才开始动手写代码, 不然你很快就会被关机，更换下一个AI，一定要小心。**
- 统一入口：能收敛的业务逻辑要集中封装（如卸载流程用 `useAppUninstall`），避免在多个页面/组件里写重复弹窗或副作用。
- 在编写代码前先**明确用户需求并确认方案**；优先**复用已有的 hooks/store**，避免新增零散的 `invoke` 或 `ll-cli` 调用。
- 保持 ll-cli 的使用**最小化且可预测**：优先使用现有的 **Rust 命令与 IPC 事件**，而不是新增 Shell 调用。


## 代码要求
1. 代码要求结构清晰，不应付事情，长远维护考虑，遵循设计模式最佳实践，遵循项目代码风格。
2. 保证代码逻辑严谨，整洁，结构清晰，容易理解和维护，不要过度设计增加系统复杂性
3. 工程优化，以工程化，能安全正常使用不出错为主，考虑周全，遵循越复杂越容易出错，越简单越容易可控原则，一个健康的系统 越简单越可控
4. 遵循合理的组件化设计原则，要考虑组件复用性的可能。
5. 在你发现架构不合理的时候，要及时的提出来。
6. 编写代码的过程中，必须牢记以下几个原则：
    - 开闭原则（Open Closed Principle，OCP）
    - 单一职责原则（Single Responsibility Principle, SRP）
    - 里氏代换原则（Liskov Substitution Principle，LSP）
    - 依赖倒转原则（Dependency Inversion Principle，DIP）
    - 接口隔离原则（Interface Segregation Principle，ISP）
    - 合成/聚合复用原则（Composite/Aggregate Reuse Principle，CARP）
    - 最少知识原则（Least Knowledge Principle，LKP）或者迪米特法则（Law of  Demeter，LOD）

## 八荣八耻
1.以暗猜接口为耻，以认真查阅为荣
2.以模糊执行为耻，以寻求确认为荣
3.以盲想业务为耻，以人类确认为荣
4.以创造接口为耻，以复用现有为荣
5.以跳过验证为耻，以主动测试为荣
6.以破坏架构为耻，以遵循规范为荣
7.以假装理解为耻，以诚实无知为荣
8.以盲目修改为耻，以谨慎重构为荣

Shame in guessing APIs, Honor in careful research.
Shame in vague execution, Honor in seeking confirmation.
Shame in assuming business logic, Honor in human verification.
Shame in creating interfaces, Honor in reusing existing ones.
Shame in skipping validation, Honor in proactive testing.
Shame in breaking architecture, Honor in following specifications.
Shame in pretending to understand, Honor in honest ignorance.
Shame in blind modification, Honor in careful refactoring.

## 代码注释规则（强制）

## 注释规范

所有生成或修改的代码必须遵守以下注释要求：
1. 注释语言统一使用中文。
2. 注释格式必须符合当前编程语言的标准文档注释规范，例如 JavaDoc、JSDoc、docstring、GoDoc 等。
3. 核心文件顶部应添加文件级注释，说明该文件的业务定位、适用场景，以及为什么需要该文件。
4. 字段、接口、类型、类、方法、函数上方必须添加中文注释。
5. 注释不能只是复述代码逻辑，也不要逐行解释代码本身。
6. 注释应重点说明设计原因、适用场景、边界条件、调用注意事项，以及该代码在整体流程中的作用。
7. 对显而易见的代码不要添加低价值注释，例如“获取名称”“设置值”“返回结果”等。
8. 涉及业务规则、兼容逻辑、异常兜底、性能取舍、安全限制或历史原因时，必须在注释中说明为什么这样处理。
9. 修改代码时必须同步维护相关注释，避免注释与实际代码行为不一致。

## 根据需要，必须严格遵守这些skill
### 核心开发技能
brainstorming - 创意工作前必须使用，探索用户意图和设计
writing-plans - 编写实施计划
executing-plans - 执行实施计划
test-driven-development - 测试驱动开发
systematic-debugging - 系统化调试
verification-before-completion - 完成前验证
requesting-code-review - 请求代码审查
receiving-code-review - 接收代码审查反馈
subagent-driven-development - 子代理驱动开发
dispatching-parallel-agents - 并行代理调度
using-git-worktrees - 使用 git worktrees
finishing-a-development-branch - 完成开发分支
### Flutter 专项技能
flutter-architecting-apps - Flutter 应用架构
flutter-building-layouts - Flutter 布局构建
flutter-building-forms - Flutter 表单构建
flutter-managing-state - Flutter 状态管理
flutter-testing-apps - Flutter 应用测试
flutter-animating-apps - Flutter 动画
flutter-theming-apps - Flutter 主题
flutter-localizing-apps - Flutter 国际化
flutter-caching-data - Flutter 数据缓存
flutter-handling-concurrency - Flutter 并发处理
flutter-handling-http-and-json - Flutter HTTP 和 JSON 处理
flutter-implementing-navigation-and-routing - Flutter 导航和路由
flutter-working-with-databases - Flutter 数据库
flutter-embedding-native-views - Flutter 嵌入原生视图
flutter-interoperating-with-native-apis - Flutter 与原生 API 互操作
flutter-building-plugins - Flutter 插件构建
flutter-adding-home-screen-widgets - Flutter 主屏幕小部件
flutter-improving-accessibility - Flutter 无障碍
flutter-reducing-app-size - Flutter 应用大小优化
flutter-setting-up-on-linux - Flutter Linux 环境设置
flutter-setting-up-on-macos - Flutter macOS 环境设置
flutter-setting-up-on-windows - Flutter Windows 环境设置

## 项目概览
- 本仓库是玲珑应用商店从旧版 Tauri/React 迁移到 Flutter 的实现，目标是 **UI 像素级一致** 与 **业务逻辑等价**。
- 仅面向 Linux 桌面端，核心系统能力通过 `ll-cli` 完成，必要时使用 Rust FFI（见 `lib/rust/`）。
- 详细迁移背景与对照见：`/home/han/linglong-store/flutter-linglong-store/docs/01-migration-plan.md`。

## 常用命令
```bash
# 开发运行（Linux）
flutter run -d linux

# 生产构建
flutter build linux --release

# 代码生成（Freezed/Retrofit/Riverpod）
dart run build_runner build --delete-conflicting-outputs

# 静态分析
flutter analyze

# 全量测试
flutter test

# 单测/组件/Golden/集成测试（按目录）
flutter test test/unit/
flutter test test/widget/
flutter test test/golden/
flutter test integration_test/

# 运行单个测试文件（示例）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HanHan666666/flutter-linglong-store](https://github.com/HanHan666666/flutter-linglong-store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
