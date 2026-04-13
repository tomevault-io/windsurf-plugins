---
trigger: always_on
description: ヤドキング・ヤドラン・ヤドンによるマルチエージェントシステム。
---

# ヤドン・エージェント

## 概要

ヤドキング・ヤドラン・ヤドンによるマルチエージェントシステム。
のんびりしているが、確実にタスクをこなす。

Python実装の詳細は[Python Code Style](#python-code-style)セクションを参照してください。

## アーキテクチャ

```
トレーナー（人間）
  │ 直接会話
  ▼
┌──────────────────┐
│   ヤドキング       │  claude --model opus
└──────┬───────────┘
       │ Unix socket: /tmp/yadon-agent-yadoran.sock
       ▼
┌──────────────────┐
│   ヤドラン        │  デスクトップペット + エージェント (PyQt6 + Python)
│                  │  タスク受信 → 3フェーズに分解 → ヤドンに配分
└──┬───┬───┬───┬──┘
   │   │   │   │
   ▼   ▼   ▼   ▼
┌──┐ ┌──┐ ┌──┐ ┌──┐
│Y1│ │Y2│ │..│ │YN│  ヤドン 1-N（PyQt6 + Python、デフォルト4ワーカー）
└──┘ └──┘ └──┘ └──┘  タスク受信 → claude -p (haiku) で実行 → 結果返却
```

GUIデーモン（ヤドラン + ヤドン1-N）は別プロセスで起動され、CLIプロセス（ヤドキング）とは独立して動作。

## 役割対応表

| ポケモン | モデル | 動作モード | 役割 |
|----------|--------|-----------|------|
| ヤドキング | opus | 対話型 | 戦略統括、最終レビュー、ダイレクト対話 |
| ヤドラン | sonnet | バックグラウンド | 3フェーズ分解、並列配分、結果集約 |
| ヤドン×N | haiku | バックグラウンド | 実作業（コーディング、テスト、ドキュメント等） |

## パッケージ構成

```
yadon-agents/
├── pyproject.toml
├── src/yadon_agents/
│   ├── cli.py / commands.py / ascii_art.py / gui_daemon.py
│   ├── domain/           # ドメイン層（型・ポート）
│   ├── agent/            # アプリケーション層（BaseAgent, YadonWorker, YadoranManager）
│   ├── infra/            # インフラ層（protocol.py: Unix socket, claude_runner.py: LLM CLI実行）
│   ├── config/           # 設定（agent.py, ui.py, llm.py）
│   ├── gui/              # GUI層（PyQt6）
│   ├── themes/           # テーマ層（スプライト・メッセージ管理）
│   └── instructions/     # 指示書（yadoking.md, yadoran.md, yadon.md）
├── tests/                # pytest テストスイート（102テスト成功）
├── memory/               # ヤドキングの学習記録
├── logs/                 # ログファイル（自動生成）
└── .claude/              # 設定＆フック
    ├── settings.json     # PreToolUseフック設定
    └── hooks/enforce-role.sh  # 役割別ツール制御
```

## Python Code Style

### 型ヒント（Type Hints）

**すべての関数・メソッドに型ヒント必須:**

```python
# ✅ 良い例
def process_task(task_id: str, timeout: int) -> dict[str, Any]:
    return {"id": task_id, "status": "completed"}

class Agent:
    def __init__(self, name: str) -> None:
        self.name: str = name

# ❌ 禁止（anyの使用）
def process_task(task_id, timeout):  # 型なし
    pass

def handle_event(data: Any) -> Any:  # anyの使用
    pass

# ❌ 禁止（as unknown as T パターン）
result = json.loads(data) as unknown as dict  # TypeScriptパターンは使用禁止
```

**型チェック:** MyPyまたはPyright での `strict` モードで完全クリア必須。

### DDD レイヤー構成

**依存関係: domain/ > agent/ > infra/**

```python
# ✅ 良い例（domain → agent → infra の一方向）
# domain/types.py
class TaskId(NewType):
    pass

class Task(NamedTuple):
    id: TaskId
    name: str

# domain/ports/services.py
class TaskPort(Protocol):
    def execute(self, task: Task) -> Result: ...

# agent/task_manager.py
class TaskManager(BaseAgent):
    def __init__(self, port: TaskPort) -> None:
        self.port = port

# infra/adapters/task_adapter.py
class TaskAdapter(TaskPort):
    def execute(self, task: Task) -> Result:
        return {"status": "done"}

# ❌ 禁止（逆依存）
# infra/ から domain/ をインポート
# agent/ から infra/ の実装クラスを直接インポート
```

### Port & Adapter パターン（依存注入）

**すべてのI/Oはポート定義 → アダプター実装で抽象化:**

```python
# ✅ 良い例
from typing import Protocol

class StoragePort(Protocol):
    def save(self, key: str, value: str) -> None: ...
    def load(self, key: str) -> str: ...

class Agent(BaseAgent):
    def __init__(self, storage: StoragePort) -> None:
        self.storage = storage

class LocalStorageAdapter(StoragePort):
    def save(self, key: str, value: str) -> None:
        with open(f"/tmp/{key}", "w") as f:
            f.write(value)

# ❌ 禁止（具体クラス依存）
class Agent(BaseAgent):
    def __init__(self) -> None:
        self.storage = LocalStorageAdapter()  # 具体化
```

### Unix Socket 通信（JSON over /tmp/yadon-agent-*.sock）

**プロトコル:** [`./.claude/rules/protocol.md`](./.claude/rules/protocol.md) 参照

**実装例:**

```python
import json
import socket

# ✅ 送信側（ポート 5000/unix）
def send_message(sock_path: str, message: dict) -> dict:
    with socket.socket(socket.AF_UNIX, socket.SOCK_STREAM) as sock:
        sock.connect(sock_path)
        sock.sendall(json.dumps(message).encode())
        return json.loads(sock.recv(4096).decode())

# ✅ 受信側
def listen(sock_path: str) -> None:
    try:
        with socket.socket(socket.AF_UNIX, socket.SOCK_STREAM) as sock:
            sock.bind(sock_path)
            sock.listen()
            conn, _ = sock.accept()
            try:
                data = json.loads(conn.recv(4096).decode())
                response = handle_message(data)
                conn.sendall(json.dumps(response).encode())
            finally:
                conn.close()
    finally:
        os.unlink(sock_path)  # cleanup
```

### PyQt6 リソース管理（try-finally 必須）

**Qt オブジェクトのライフサイクル管理:**

```python
# ✅ 良い例（親オブジェクト設定）
class MyWidget(QWidget):
    def __init__(self) -> None:
        super().__init__()
        self.button = QPushButton("Click", self)  # selfを親として指定 → 自動cleanup

# ✅ 良い例（接続時はメモリ管理に注意）
class Agent(BaseAgent):
    def on_bubble(self, message: str) -> None:
        try:
            # リソース確保
            socket = socket.socket()
            socket.connect(...)
        finally:
            # 必ずクローズ
            socket.close()

# ❌ 禁止（cleanup忘れ）
def create_widget() -> None:
    button = QPushButton("Click")  # 親なし → GCに依存（危険）
    button.show()
```

## 設計方針

Python設計に関する判断：

- **Port & Adapter（DI）** — 依存注入パターン、ポート定義

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yida29) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
