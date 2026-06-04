---
trigger: always_on
description: AIエージェントをツールではなく「自律的な人」として扱うフレームワーク。
---

# AnimaWorks — Digital Anima Framework

AIエージェントをツールではなく「自律的な人」として扱うフレームワーク。
各Animaは固有のアイデンティティ・記憶・判断基準を持ち、ハートビートやcronで自律行動する。

## 設計原則

- **カプセル化**: Anima内部の思考・記憶は外部から不可視。外部とはテキスト会話のみ
- **RAG記憶**: PrimingレイヤーがRAGで自動検索した関連記憶をシステムプロンプトに注入する。加えてエージェントは自律的に記憶を検索可能。サイズ無制限
- **自律性**: ハートビート(定期巡回)とcron(定時タスク)で人間の指示なしに行動
- **プロセス分離**: ProcessSupervisorが各AnimaをUnixソケット付き独立子プロセスとして起動・監視する
- **海馬モデル**: PrimingEngineがプロンプト構築における唯一のアクティビティリーダー。builder.pyはActivityLoggerを直接読まない

## ディレクトリ構成

```
core/                        # フレームワーク本体
├── anima.py, agent.py, lifecycle.py  # コアエンティティ・オーケストレーター
├── anima_factory.py, init.py         # 初期化・Anima生成
├── schemas.py, paths.py              # データモデル・パス定数
├── messenger.py, logging_config.py   # 通信・ログ
├── background.py                     # バックグラウンドタスク
├── asset_reconciler.py               # アセット自動生成
├── org_sync.py                       # 組織同期
├── outbound.py                       # 統一アウトバウンドルーティング（Slack/Chatwork/内部自動判定）
├── schedule_parser.py                # cron.md/heartbeat.mdパーサー
├── voice/                            # 音声チャットサブシステム
│   ├── stt.py, tts_base.py, tts_factory.py
│   ├── tts_voicevox.py, tts_elevenlabs.py, tts_sbv2.py
│   ├── sentence_splitter.py
│   └── session.py
├── memory/                           # 記憶サブシステム
│   ├── manager.py, conversation.py, shortterm.py
│   ├── priming.py, consolidation.py, forgetting.py
│   ├── activity.py, streaming_journal.py
│   └── rag/                          # RAGエンジン（ChromaDB + sentence-transformers）
│       ├── indexer.py, retriever.py, graph.py
│       ├── store.py, singleton.py, watcher.py
├── supervisor/                       # プロセス監視
│   ├── manager.py, ipc.py, runner.py, process_handle.py
├── notification/                     # 人間通知
│   ├── notifier.py
│   └── channels/                     # 通知チャネル（slack, chatwork, line, telegram, ntfy）
├── auth/                             # 認証
├── tooling/                          # ツール基盤
│   ├── handler.py                    # ToolHandler（権限チェック・ディスパッチ）
│   ├── schemas.py                    # ツールスキーマ定義
│   ├── guide.py                      # 動的ツールガイド生成
│   └── dispatch.py                   # ExternalToolDispatcher（外部ツール振り分け）
├── config/                           # 設定管理
│   ├── models.py                     # Pydanticモデル・load/save
│   ├── cli.py, migrate.py
├── prompt/                           # プロンプト・コンテキスト管理
│   ├── builder.py                    # システムプロンプト構築（6グループ構造）
│   └── context.py                    # コンテキストウィンドウ追跡
├── execution/                        # 実行エンジン
│   ├── base.py                       # BaseExecutor・ExecutionResult
│   ├── agent_sdk.py                  # S: Claude Agent SDK
│   ├── codex_sdk.py                  # C: Codex CLI
│   ├── cursor_agent.py               # D: Cursor Agent CLI
│   ├── gemini_cli.py                 # G: Gemini CLI
│   ├── anthropic_fallback.py         # A（内部分岐）: Anthropic SDK直接
│   ├── litellm_loop.py              # A: LiteLLM + tool_useループ
│   ├── assisted.py                   # B: 1ショット（記憶I/O代行）
│   └── _session.py                   # セッション継続・チェイニング
└── tools/                            # 外部ツール実装
    ├── web_search.py, x_search.py
    ├── slack.py, chatwork.py, gmail.py
    ├── github.py, aws_collector.py
    ├── image_gen.py
    ├── transcribe.py, local_llm.py
    └── _base.py, _cache.py, _retry.py
cli/                                  # CLIパッケージ
├── parser.py                         # argparse定義 + cli_main()
└── commands/                         # サブコマンド実装
server/                               # FastAPI単体サーバー + Web UI
├── app.py, websocket.py
├── routes/                           # APIルート（ドメイン別分割）
└── static/                           # フロントエンド
    ├── index.html
    ├── modules/                      # JS モジュール
    ├── styles/                       # CSS
    └── workspace/                    # インタラクティブWorkspace（3Dオフィス・会話画面）
templates/                            # 初期化テンプレート
├── ja/, en/                          # ロケール別テンプレート
│   ├── prompts/                      # プロンプトテンプレート
│   ├── anima_templates/              # Animaスケルトン
│   ├── roles/                        # ロールテンプレート
│   ├── common_knowledge/             # 共有知識テンプレート
│   └── common_skills/                # 共通スキルテンプレート
├── _shared/                          # ロケール共通（company等）
main.py                               # CLIエントリポイント
```

### Chat UI レイアウト注意点（2026-02-27）

`/#chat` の入力欄下部に意図しない隙間が出る場合は、以下を優先確認する。

- `server/static/styles/responsive.css` の `.chat-page-layout` は `height: 100%` を基準にする（`calc(100svh - 140px)` だと親余白と二重計算になりやすい）。
- `server/static/styles/layout.css` で `.main-content:has(.chat-page-layout)` を使い、チャット時のみ下余白を制御する（現在は `padding-bottom: 10px`）。
- `server/static/pages/chat.js` の `#chatPageForm` にインライン `padding` を持たせない（CSS一元管理）。
- 入力欄の空き領域クリックでフォーカスできるよう、`.chat-input-wrap` に `pointerdown/click -> #chatPageInput.focus()`（ボタン類は除外）を入れる。
- キャッシュ影響を避けるため、`server/static/index.html` で `chat.css` / `layout.css` / `responsive.css` / `modules/app.js` のクエリバージョンを更新して確認する。

### 非活性ディレクトリ（検索不要）

- `archive/` — 旧設計・非推奨コード（gateway, worker, broker 含む）。参照のみ
- `site-docs/` — 生成済みドキュメント
- `.venv/`, `__pycache__/`, `animaworks.egg-info/` — ビルド成果物

## ランタイムデータ（リポジトリ外）

`~/.animaworks/` に配置。`animaworks init` で生成:

```
~/.animaworks/
├── config.json               # 統合設定
├── auth.json                 # UI認証設定

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xuiltul/animaworks](https://github.com/xuiltul/animaworks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
