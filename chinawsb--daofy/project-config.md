---
trigger: always_on
description: Daofy — Python 3.10-3.14, Windows, pytest.
---

# AGENTS.md - Agent Coding Guidelines

Daofy — Python 3.10-3.14, Windows, pytest.

## Quick Command Reference

| Action | Command |
|--------|---------|
| Install deps | `pip install -r requirements.txt && pip install -e ".[dev]"` |
| Install file watcher | `pip install daofy-for-delphi[watcher]` (可选，自动增量 KB) |
| Install embedding | `pip install daofy-for-delphi[embedding]` (可选，语义搜索) |
| Run all tests | `pytest` (or `python tests/run_all_tests.py` for basic) |
| Run one test | `pytest tests/test_validator.py -v` |
| Lint/type | `mypy src/` |
| Environment | Always set `$env:PYTHONIOENCODING='utf-8'` on Windows |
| Run server | `python src/server.py` |

## Project Structure

```
src/
├── server.py              # MCP entry point
├── tools/                 # MCP tool implementations (delphi_file, code_hosting, delphi_project, delphi_kb, ...)
├── services/              # Business logic
│   ├── compiler_service.py, config_manager.py, process_manager.py, args_generator.py
│   └── knowledge_base/    # KB modules (schema, smart_cache, project, thirdparty, scan, embedding, async_task_manager)
├── models/                # Pydantic/dataclass models
└── utils/                 # Utilities (delphi_env, dproj_parser, validator, logger)
```

## 知识库自动生命周期

项目知识库（`.delphi-kb/`）有以下自动化机制：

| 机制 | 触发时机 | 说明 |
|------|---------|------|
| **启动时自动构建** | MCP Server 启动 | 自动检测 CWD 下的 `.dproj`，提交后台增量 KB 构建（不阻塞 MCP 就绪） |
| **热切换重建** | 用户手动 rebuild | 构建到临时目录 `.delphi-kb-tmp-{ts}/`，旧 KB 在构建期间保持可查，构建完成原子 swap |
| **文件变更监听** | 用户保存 `.pas/.dfm/.dproj` 等 | 需要 `pip install daofy-for-delphi[watcher]`，3秒去抖后自动触发增量更新 |

**文件变更监听依赖**: `watchdog` 可选包，未安装时静默降级，不影响其他功能。
**查看服务器状态**: MCP 资源 `delphi://health` 返回版本号、运行时长、监听器状态。

## 工具使用规则

### Git 操作必须使用 code_hosting
所有 Git 操作（status/add/commit/push/clone/push_retry）必须通过 `code_hosting` 工具，禁止直接使用 bash 运行 git 命令。`code_hosting` 会统一格式化输出、自动处理异步推送重试，比原始 bash git 更省 token。

### 经验库维护规则
`experience` 工具会自动去重（embedding 相似度 >0.85 时合并到旧记录而非新增），但 AI 仍需主动维护经验质量：

0. **保存前先泛化**：调用 `experience(action="save")` 前，先 `experience(action="search", query=...)` 搜索是否已存在覆盖同类问题的抽象经验。如果找到，用 `action=merge` 或 `action=update` 将当前场景合并进去，不要另存为一条具体场景的记录。示例：不要存「编译后 output_file 冗余」，而是合并到「MCP 工具接口输出精简」——后者可以覆盖任意工具的返回值清理。
1. **任务完成后**：如果刚解决的问题与已有经验高度相关，但解决方式不同，用 `action=merge` 手动合并两条经验
2. **定期清理**：用 `action=prune` 列出低价值（低 hit_count、长期未更新）的经验，检查后 `action=delete` 删除
3. **抽象合并**：发现多条经验描述的是同一类问题（如不同工具的「消息精简」），手动合并为一条抽象经验，`tags` 要覆盖各类场景
4. **重建缺失向量**：若 embedding 模型后来才加载（首次使用时未加载），旧记录缺少向量。`search()` 在模型已加载但无结果时会自动触发 `rebuild_embeddings()` 补全，无需手动调用。也可通过 `experience(action="rebuild_embedding")` 显式触发。

## Agent 编码工作流（优先级顺序）

### 编辑 Delphi 文件前

> ⚠️ **工具调用规则**：以下每一步都使用对应的 **MCP 工具名** 调用。每个工具名即是你在 `list_tools()` 中看到的工具名称。如果工具未出现在列表，刷新 MCP 连接。

| 步骤 | 调用的 MCP 工具 | 说明 |
|------|----------------|------|
| ① 环境检查 | `check_environment(action="check")` | 确认编译器可用 |
| ② 编码规则 | `get_coding_rules(project_path=...)` | 获取项目编码规范 |
| ③ API 搜索 | `delphi_kb(query=...)` | 搜索 API 定义（详见 `config/CODING_RULES.mdc` ② 节） |
| ④ 读源码 | `delphi_file(action="read", file_path=...)` | 读取文件确认修改点 |
| ⑤ 写代码 | `delphi_file(action="write", edits=[...])` | 写入代码（自动备份到 __history） |
| ⑤b 批量写 | `delphi_file(action="write", edits=[...])` | 批量写入多处（edits 顺序不限，自动备份到 __history） |
| ⑥ 格式化 | `delphi_file(action="format", file_path=...)` | pasfmt 格式化代码 |
| **⑦ 编译验证** | **`delphi_project(action="compile", project_path=...)`** | **编译 `.dproj`/`.dpr`/`.dpk` 项目** |
| **⑧ 运行验证** | **`delphi_project(action="compile", ..., run_verify=True)`** | **编译后启动 exe 运行 3 秒，检测运行时崩溃** |
| **⑨ 运行时检查** | **`delphi_project(action="runtime", base_dir=...)`** | **扫描组件类名，检测遗漏 uses 单元** |

各步骤补充说明：

- **⑧ run_verify**: 编译成功后自动启动 exe 运行 3 秒，若进程崩溃则标记验证失败（秒级，自动结束进程）。检测到 `exception.log` 时使用 `detect_encoding`（与 delphi_file 同款 BOM/编码检测）读取内容直接嵌入 MCP 响应，无需 AI 额外调用 delphi_file。
- **⑨ runtime 检查**: 扫描 .pas/.dfm 中组件类名，匹配 `src/rules/runtime_registry.json` 规则表，检测是否遗漏必需 uses 单元（如 FireDAC.DApt）。独立于编译步骤，纯源码级分析。

> 详细 KB 搜索策略、优先级规则、kb_type 范围、Entity Kind Codes 见 `config/CODING_RULES.mdc` ② 节。

## Code Style (Python)

- **类型**: 全部用 type hints
- **导入顺序**: stdlib → third-party → local (每组空格分隔, 内部字母序)
- **命名**: `snake_case` 函数/变量, `PascalCase` 类, `UPPER_SNAKE` 常量, `_前缀` 私有
- **文档**: Google-style docstring, 公共函数必须有
- **异常**: 用具体异常类型, 不要 `except Exception: pass`
- **测试**: `test_*.py`, `test_` 前缀函数, pytest fixture, `unittest.mock` 打桩

## Agent 操作硬规则

> 脚本执行、字符串格式化、Python 陷阱等通用规则见 `config/CODING_RULES.mdc`「Agent 操作硬规则」。

### 多进程 Worker
- **Worker 内部禁用 `print()`**：MCP 环境下 stdout 是 JSON-RPC 通信管道，worker print 破坏协议边界，构建从 8s 飙到 172s
- Worker 标准模式：
  ```python
  def worker(args):
      import io, sys
      sys.stdout = sys.stderr = io.StringIO()
      try:
          ...  # 实际工作
          return result
      except:
          return None  # 错误通过返回值传递
  ```

### 数据库
- **DDL 统一**：所有建表语句集中在 `schema.py`，各 Builder 调 `create_source_tables()`/`create_document_tables()`
- **同一 DB 文件所有连接用相同 journal 模式**：本项目统一使用 WAL（`PRAGMA journal_mode=WAL`），切换模式需要独占锁，运行中若有其他连接会 locked
- 修改表结构后 `grep` 全项目旧表名/列名的所有 INSERT/DELETE/SELECT/ALTER 引用

### delphi_file 工具使用规则（详见 CODING_RULES.mdc）

编辑 Delphi 文件时的行号规则、脏标记保护、输出格式等详细规范已迁移至 `config/CODING_RULES.mdc`，AI 应通过以下方式获取：

```python
get_coding_rules(section="writing")                     # 写代码全部规则

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chinawsb/daofy](https://github.com/chinawsb/daofy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
