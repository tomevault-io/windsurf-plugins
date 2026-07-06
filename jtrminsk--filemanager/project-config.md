---
trigger: always_on
description: > 目标:把现有 PySide6 桌面文件管理器改造为「可对话、可拖文件、有记忆」的 Agent 应用。
---

# FileManager Agent 化技术方案

> 目标:把现有 PySide6 桌面文件管理器改造为「可对话、可拖文件、有记忆」的 Agent 应用。
> 本文档供 Cursor 实现使用。**核心原则:复用现有逻辑,新增能力全部解耦,GUI 改动最小化。**

---

## 0. 现状摘要(实现前必读)

现有代码分层很干净,绝大部分业务逻辑已与 Qt 无关。改造时**严禁重写下列已验证可直接复用的模块**:

| 文件 | Qt 依赖 | 处理方式 |
|---|---|---|
| `models.py` | 无 | **原样复用**(`FileEntry` 数据类) |
| `fs_ops.py` | 无 | **原样复用**(复制/删除/卷类型判断) |
| `profile.py` | 无 | **原样复用**(目录画像) |
| `scanner.py` | 仅 `QThread` 外壳 | 抽出纯函数,外壳改为调用它 |
| `table_model.py` | 筛选逻辑在 proxy 内 | 抽出纯函数,proxy 改为调用它 |
| `window.py` | 全 Qt | 加对话面板 + 拖拽;预览逻辑抽出 |

可直接复用的现有函数签名(实现工具层时调用这些,不要另写):

```python
# fs_ops.py —— 原样使用
copy_paths(paths: list[Path], dest_dir: Path) -> tuple[list[str], list[str]]  # (成功路径, 错误信息)
trash_paths(paths: list[Path]) -> tuple[list[str], list[str]]                  # 回收站
delete_paths_permanent(paths: list[Path]) -> tuple[list[str], list[str]]       # 永久删除
path_expects_recycle_bin(path: Path) -> bool                                   # True=可回收站,False=永久删除

# profile.py —— 原样使用
summarize_directory(root: Path, entries: list[FileEntry]) -> str

# models.py —— 原样使用
FileEntry(path: Path, size: int, mtime: float)
  .name -> str            # 文件名
  .suffix -> str          # 小写扩展名含点,无则空串
  .relative_display(root) -> str
  .modified_dt() -> datetime
```

---

## 1. 目标架构

```
┌──────────────────────────── GUI 层 (PySide6) ────────────────────────────┐
│  window.py (改)                                                          │
│  ┌────────────────────────────┬──────────────────────────────────────┐  │
│  │ 现有左区:                    │ chat_panel.py (新)                     │  │
│  │  顶栏/筛选/文件表格/预览/画像  │  消息流 + 拖文件区 + 输入框 + 新会话按钮 │  │
│  └────────────────────────────┴──────────────────────────────────────┘  │
│  scanner.py(改 run)   table_model.py(改 filter)   agent_thread.py (新)    │
└────────────────────────────────┬─────────────────────────────────────────┘
                                  │ 调用 / 信号
┌─────────────────────────────── 逻辑层 (无 Qt 依赖) ───────────────────────┐
│  agent.py (新)        Agent 循环:LLM ↔ 工具调用 ↔ 记忆                     │
│  llm/ (新)            可切换 LLM 抽象层 + 各家适配器                        │
│  tools.py (新)        工具注册表(把 core/fs_ops 包成 LLM 可调 schema)      │
│  memory.py (新)       记忆系统(MD 为主 + SQLite 为辅 + 向量可选)           │
│  core.py (新)         纯函数:scan_directory / filter_entries / preview_file│
│  models.py fs_ops.py profile.py (原样)                                    │
└───────────────────────────────────────────────────────────────────────────┘
```

目标目录结构:

```
src/filemanager/
  # ── 现有,原样 ──
  __init__.py  __main__.py  main.py
  models.py  fs_ops.py  profile.py
  # ── 现有,小改 ──
  scanner.py          # run() 改为调用 core.scan_directory
  table_model.py      # filterAcceptsRow 改为调用 core.filter_entries
  window.py           # 嵌入 chat_panel;现有功能不动
  # ── 新增:逻辑层(无 Qt)──
  core.py             # 扫描/筛选/预览的纯函数
  tools.py            # 工具定义与分发
  agent.py            # Agent 主循环 + 上下文管理
  memory.py           # 长期记忆
  config.py           # 配置(模型选择、API key、路径约束、阈值)
  llm/
    __init__.py
    base.py           # LLMClient 抽象基类 + 统一消息/工具/响应格式
    anthropic_client.py
    openai_client.py
    ollama_client.py
  # ── 新增:GUI 组件 ──
  chat_panel.py       # 对话面板 widget(拖拽 + 消息流)
  agent_thread.py     # QThread 跑 Agent,避免阻塞界面
```

---

## 2. 核心层 `core.py`(第一步,不碰 GUI)

把分散在 `scanner.py` / `table_model.py` / `window.py` 里的逻辑抽成无 Qt 依赖的纯函数。**逻辑从现有代码原样搬,不要重新设计算法。**

```python
from dataclasses import dataclass
from pathlib import Path
from filemanager.models import FileEntry

# ---- 扫描:逻辑搬自 scanner.ScanThread.run ----
def scan_directory(
    root: Path,
    recursive: bool,
    progress_cb: callable | None = None,   # 可选回调 progress_cb(count)
) -> list[FileEntry]:
    """递归 rglob 或单层 iterdir,仅保留 is_file,逐个 stat 构造 FileEntry。
    单文件 stat 失败跳过;根路径错误抛 OSError 由调用方处理。"""

# ---- 筛选:逻辑搬自 FileFilterProxy.filterAcceptsRow ----
def filter_entries(
    entries: list[FileEntry],
    exts: set[str] | None = None,      # 小写带点,如 {'.pdf', '.txt'};None=不限
    min_size: int | None = None,       # 字节
    max_size: int | None = None,
    name_sub: str = "",                # 小写子串匹配
    min_mtime: float | None = None,    # unix 秒
    max_mtime: float | None = None,
) -> list[FileEntry]:
    """按条件过滤,返回新列表。规则与现有 proxy 完全一致。"""

# ---- 预览:逻辑搬自 window._update_file_preview + 三个 helper ----
@dataclass
class PreviewResult:
    kind: str          # "image" | "text" | "hex" | "error"
    text: str = ""     # text/hex 时的内容;error 时的错误信息
    image_path: str = ""  # image 时的路径(GUI 用 QPixmap 加载;Agent 只读 kind)
    truncated: bool = False

def preview_file(path: Path) -> PreviewResult:
    """图片→image;否则读前 512KB 判文本/二进制→text/hex。
    复用 window.py 的 _is_probably_text / _format_hex_preview 逻辑(一并搬到 core)。"""

# ---- 解析 helper:搬自 table_model._parse_ext_filter / window._parse_mb ----
def parse_ext_filter(text: str) -> set[str] | None: ...
def parse_mb(text: str) -> int | None: ...
```

**改动 `scanner.py`**:`ScanThread.run()` 内部改为 `entries = scan_directory(self._root, self._recursive, self.progress.emit)`,信号 emit 逻辑保留。
**改动 `table_model.py`**:`FileFilterProxy` 改为持有当前筛选条件,`filterAcceptsRow` 调用 `core.filter_entries` 对单条判断(或保留逐行判断但复用同一套规则常量)。
**验收**:`python -m filemanager` GUI 行为与改造前**完全一致**。

---

## 3. 可切换 LLM 抽象层 `llm/`

不同厂商工具调用格式不同,用抽象层统一。Agent 只依赖 `LLMClient`,不依赖具体厂商。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JTRMinsk/filemanager](https://github.com/JTRMinsk/filemanager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
