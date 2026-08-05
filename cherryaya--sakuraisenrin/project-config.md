---
trigger: always_on
description: SakuraiSenrin 插件开发规范（Plugin Development Spec）
---

# AGENTS.md

SakuraiSenrin 插件开发规范（Plugin Development Spec）

版本：v1.2  
适用范围：`src/plugins/**`、`src/hooks/**`、`tests/plugins/**`  
最后更新：2026-07-01

---

## 1. 目标与定位

本文件用于给人类开发者与 AI coding agents 提供统一、可执行的项目规则，重点解决：

1. 插件如何在当前架构中正确接入。
2. 插件代码如何保持可维护、可测试、可审计。
3. 新增功能如何不破坏已有运行时约束（同步检查、黑白名单、缓存与落库策略）。

本文件是“面向 agent 的 README”，不替代业务文档。

---

## 2. AGENTS 规范兼容约定（参考 agentsmd）

本项目采用 `agentsmd/agents.md` 的核心原则：

1. `AGENTS.md` 使用标准 Markdown，无强制字段。
2. 若未来存在多个 `AGENTS/Agents` 文件，按“离目标文件最近者优先”处理冲突。
3. 用户在当前会话中的显式指令优先级高于文档默认规则。
4. 文档中写出的检查命令视为可执行质量门禁，agent 需尽量自动执行并修复失败。

注：为兼容不同工具链，若后续新增子目录级 `AGENTS.md`，需遵循“近目录优先”并避免与根规范冲突。

---

## 3. 项目架构快照（当前事实）

### 3.1 启动与加载链路

1. 入口：`bot.py`
2. 启动阶段：
   - `init_fonts()`
   - `nonebot.init()`
   - 注册 OneBot V11 adapter
3. `on_startup`：`init_db()` 初始化 `core/log/snapshot` 库
4. `on_bot_connect`：
   - 各 repository `warm_up()`
   - API 同步用户/群组缓存
5. 插件加载：
   - `nonebot.load_plugins("src/hooks/")`
   - `nonebot.load_plugins("src/plugins/")`

### 3.2 运行时拦截与鉴权

`src/hooks/processor.py` 通过 `run_preprocessor` 统一执行：

1. 运行时同步：用户、群组、群成员缓存回填。
2. 运行时检查：全局忽略、超管放行、黑名单拦截、群授权检查、全员禁言检查。
3. `PluginMetadata.extra.no_check` 可绕过该检查链（仅限明确需要的系统级插件）。

### 3.3 数据层职责分离

1. `core_db`：主业务实体（用户/群组/成员/邀请/黑名单/插件开关）。
2. `log_db`（分片）：审计日志与插件使用日志。
3. `snapshot_db`（分片）：名称/名片等变更快照。
4. Repository 层默认 `cache -> diff -> writer/db`，不鼓励插件直接跨层写 SQL。

---

## 4. 插件分层模型（强制）

插件按“入口编排 + 业务处理 + 数据访问”分层：

1. 插件入口文件：只做 matcher 注册、元数据声明、薄编排。
2. `handlers/`：处理命令解析、事件分流、交互文案。
3. `services/`：封装业务规则、流程编排、幂等控制。
4. `database/`（可选）：插件独立数据模型、ops、repo、writers。
5. 文档统一通过 `PluginMetadata.extra.docs` 暴露给 `help` 自动注册。
6. 多文件命令插件必须提供 `docs/README.MD`；单文件命令插件允许将文档直接内嵌在 `.py` 中。
7. 被动插件可仅保留代码内文档实现。

说明：
包级 `__init__.py` 若仅用于导入子模块、声明 `__all__` 或组织目录结构，而不是被 NoneBot 直接视作插件入口，则不受 `extra.docs` 强制约束。

禁止在 handler 中堆积复杂 SQL、跨模块状态机和大段流程逻辑。

---

## 5. 目录与命名规范

### 5.1 推荐目录模板

```text
src/plugins/<plugin_name>/
  __init__.py               # 多文件插件入口
  docs/README.MD            # 多文件 COMMAND 插件必需；PASSIVE/EVENT/CRON 可选
  handlers/
    __init__.py
    admin.py
    passive.py
  services/
    __init__.py
    core.py
  database/                 # 仅在插件有独立持久化需求时创建
    __init__.py
    instances.py
    tables.py
    ops.py
    repo.py
    types.py
    writers.py
```

### 5.2 命名要求

1. 插件目录名使用小写语义名，如 `water`、`notice`。
2. 命令插件优先使用 `on_command` / `CommandGroup`，命令前缀和别名集中定义。
3. 文件命名表达职责，不使用 `misc.py`、`tmp.py` 一类模糊命名。
4. 单文件插件可直接位于 `src/plugins/*.py`，此时文档允许直接内嵌在该文件中。

---

## 6. 插件元数据与权限契约

每个可加载插件必须声明 `PluginMetadata`，且 `extra` 至少包含：

1. `author`
2. `version`
3. `trigger`（建议复用 `src/lib/consts.py::TriggerType`）
4. `permission`（建议复用 `src/database/core/consts.py::Permission`）
5. `docs`（帮助系统自动注册文档的统一接口）

可选字段：

1. `no_check`：仅允许用于需要绕过运行时拦截的系统通知/防护插件。

约束：

1. 默认不设置 `no_check`。
2. 涉及管理能力的命令必须设置明确权限（如 `SUPERUSER` 或群管理角色检查）。
3. 所有插件必须实现 `extra.docs`，否则视为不合规插件。
4. 本项目插件层不再维护独立 `usage` 文本，帮助文档统一由 `extra.docs` schema 描述并由项目级文档引擎渲染。

### 6.1 文档接口契约（强制）

`extra.docs` 推荐最小结构如下：

```python
from src.lib.plugin_meta import create_plugin_metadata

__plugin_meta__ = create_plugin_metadata(
    name=name,
    description=description,
    extra={
        "author": "SakuraiCora",
        "version": "0.1.0",
        "trigger": TriggerType.PASSIVE,
        "permission": Permission.SUPERUSER,
        "docs": {
            "kind": "plugin",
            "source": {
                "kind": "readme",
                "readme_path": str(DOCS_SOURCE),
            },
            "tree": {
                "slug": "notice.user",
                "parent_slug": "notice",
                "category": "system",
                "order": 100,
            },
            "visibility": {
                "visible": False,
                "hidden": False,
                "internal": False,
            },
            "permission": Permission.SUPERUSER,
            "aliases": ("用户事件处理", "notice.user"),
        },
    },
)
```

字段要求：

1. `kind`：必填，描述节点类型，如 `plugin`、`overview`、`internal`。
2. `source`：必填，当前至少支持 `readme_path`。
3. `tree.slug`：必填，要求全仓唯一。
4. `tree.parent_slug`：子节点必填；顶层节点留空。
5. `visibility.visible`：必填，控制是否出现在默认 help 列表。
6. `permission`：建议显式填写；节点默认权限与插件权限一致时也可重复声明。
7. `aliases`：建议填写，便于 help 精确/模糊命中。

### 6.2 文档内容格式（统一由引擎渲染）

README 仍是首要文档内容载体，但它已经从“最终协议”降级为“结构化数据源”。渲染结果统一由项目级文档引擎生成。

约束：

1. `help` 插件只依赖 `extra.docs` schema、README 数据源和统一渲染器，不硬编码插件说明。
2. 命令插件的 README 必须覆盖命令、参数、示例、权限说明。
3. 被动插件可简化 README 内容，但仍必须提供可被精确查询的节点。
4. 多文件命令插件必须提供 `docs/README.MD`；单文件插件可将文档内嵌在 `.py`。
5. `help` 对节点/功能查询必须支持精确命中、模糊命中和歧义提示三种结果。
6. 总览节点与子节点关系必须通过 `tree.parent_slug` 显式表达，禁止继续依赖“继续发送 #help xxx”这类隐式跳转文案。

---

## 7. 运行时行为规范

1. 高并发热路径默认使用 `WritePolicy.BUFFERED`，降低 I/O 抖动。
2. 管理类强一致操作可使用 `WritePolicy.IMMEDIATE`。
3. 对外 API 调用必须可失败并降级，避免阻塞主流程。
4. 事件处理需要防止凛凛自触发循环（如 `event.user_id == bot.self_id`）。
5. 定时任务需保证：
   - 唯一 `id`
   - 幂等控制
   - 失败日志可追踪

### 7.1 OneBot Message 注入约束（强制）

1. 禁止使用 `Message(...)` 直接包裹文本消息，避免将用户输入或插值字符串按 CQ 码重新解析，造成 message 注入。
2. 纯文本消息必须使用 `MessageSegment.text(...)` 手动组装；若项目已有统一辅助函数，应优先复用。
3. 图片、艾特、回复等复合消息必须显式逐段拼装 `MessageSegment.image(...)`、`MessageSegment.at(...)`、`MessageSegment.reply(...)` 等，不得把整段 CQ 文本塞进 `Message(...)`。
4. `Message()` 空构造仅允许用于“先建空消息、再逐段 append segment”的场景；禁止后续再通过 `Message(raw_text)` 回退到不安全构造。
5. 若文件导入 `Message` 仅用于 type hint、返回类型标注、`CommandArg()` / `Arg()` 参数声明、`isinstance` 判断或消息对象遍历，可保留，不视为违规。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CherryAya/SakuraiSenrin](https://github.com/CherryAya/SakuraiSenrin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
