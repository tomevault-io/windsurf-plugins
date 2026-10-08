---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

Claude Codeのスキルコレクションです。SDD（ソフトウェア設計ドキュメント）管理スキル群を中心に、分析・インタビュー・ユーティリティスキルを提供します。

## リポジトリのドキュメント構造

このリポジトリには2種類のドキュメントがあります:

| 種類 | 配置場所 | 対象読者 | 説明 |
|------|----------|----------|------|
| **ユーザー向けドキュメント** | `docs/` | 人間 | 使い方ガイド、スキルカタログ、ワークフロー解説、開発ガイド |
| **スキル内部リソース** | 各スキルディレクトリ | Claude Code | SKILL.md、references/、assets/templates/ |

```text
docs/                              # ユーザー向けドキュメント（人間が読む）
├── getting-started.md             # 導入ガイド
├── skill-catalog.md               # 全スキル一覧と使い方
├── sdd-workflow.md                # SDDワークフロー解説
└── development-guide.md           # スキル開発者向けガイド

{カテゴリ名}/{スキル名}/             # スキル内部リソース（Claude Codeが読む）
├── SKILL.md                       # スキル定義
├── references/                    # スキル実行時にClaudeが参照するリファレンス
└── assets/templates/              # ドキュメント生成用テンプレート
```

## SDDスキル構成

```text
sdd-documentation（オーケストレーター）
    ├── requirement-management   → docs/requirements/*.yaml（要求・ストーリー・理由・用語・関係）
    │                               ＋ generated/（自動生成の一覧・追跡表・関係グラフ）
    ├── task-planning            → GitHub Issue（デフォルト）/ docs/sdd/tasks/（オプション）
    ├── task-executing           → 実装コード（逆順レビュー付き）
    └── sdd-troubleshooting      → 問題分析・修正タスク（承認フロー付き）

非推奨（既存 docs/sdd/ の保守・参照のみ。新規利用は禁止）
    ├── requirements-defining    → requirement-management に統合
    ├── software-designing       → 設計はPR本文とGitHubコメントに記載（永続化しない）
    └── sdd-document-management  → 要求の整合性チェックは reqctl.py validate に統合
```

> **設計の記録先**: 設計は実装時に必ず行うが、PR本文とGitHubコメントにのみ記載し、リポジトリには永続化しない。レジストリにはPRのURLを `design_refs` として残す。
>
> **タスクの出力先**: SDDスキルはタスクを**デフォルトでGitHub Issueとして起票**する（1タスク＝1 Issue、詳細はIssue本文に集約、ラベル `sdd:task` で一覧＝目次）。ステータスはラベル `sdd:status/*`、DONEはIssueのcloseで表現。ユーザーが「ファイルで管理」を明示した場合のみ `docs/sdd/tasks/` にファイル生成する。

## スキルファイル一覧

### sdd/sdd-documentation/
- **SKILL.md** - 統合スキル定義（初期化・整合性チェック・逆順レビュー・エージェントチーム・タスク同期）
- **references/workflow_guide_ja.md** - ワークフローガイド
- **references/agent_teams_guide_ja.md** - エージェントチーム活用ガイド
- **references/task_sync_guide_ja.md** - タスク同期（TodoWrite連携）ガイド
- **references/checklist_ja.md** - チェックリスト

### sdd/requirements-defining/（非推奨 → requirement-management）
- **SKILL.md** - スキル定義ファイル
- **assets/templates/requirements_index_template_ja.md** - 要件目次テンプレート
- **assets/templates/user_story_template_ja.md** - ユーザーストーリーテンプレート
- **assets/templates/nfr_template_ja.md** - 非機能要件テンプレート
- **references/ears_notation_ja.md** - EARS記法の詳細ガイド

### sdd/software-designing/（非推奨 → 設計はPR本文へ）
- **SKILL.md** - スキル定義ファイル
- **assets/templates/design_index_template_ja.md** - 設計目次テンプレート
- **assets/templates/component_template_ja.md** - コンポーネント設計テンプレート
- **assets/templates/api_template_ja.md** - API設計テンプレート
- **assets/templates/database_template_ja.md** - データベース設計テンプレート
- **assets/templates/decision_template_ja.md** - 技術的決定テンプレート
- **references/design_patterns_ja.md** - 設計パターンリファレンス
- **references/ears_notation_ja.md** - EARS記法（要件参照用）
- **references/cicd_guide_ja.md** - CI/CDガイド

### sdd/task-planning/
- **SKILL.md** - スキル定義ファイル（デフォルト: GitHub Issue起票 / オプション: ファイル生成）
- **assets/templates/task_issue_template_ja.md** - タスクIssueテンプレート（デフォルト・Issue本文フォーマット/ラベル体系）
- **assets/templates/tasks_index_template_ja.md** - タスク目次テンプレート（ファイルモード）
- **assets/templates/task_detail_template_ja.md** - タスク詳細テンプレート（ファイルモード）
- **references/task_guidelines_ja.md** - タスク管理ガイドライン

### sdd/task-executing/ + agents/task-executing.md
- **agents/task-executing.md** - エージェント定義ファイル（model: sonnet、Agent toolのsubagent_typeとして使用）
- **references/execution_guide_ja.md** - 実行ガイド
- **references/commit_templates_ja.md** - コミットテンプレート
- **references/jules_integration_ja.md** - Jules連携ガイド

> **注**: task-executing はSKILL.mdを持たず、エージェント定義は `agents/task-executing.md` に配置。`sdd/task-executing/references/` のリソースを参照して動作する。

### sdd/sdd-troubleshooting/
- **SKILL.md** - スキル定義ファイル（問題確認・原因分析・承認フロー）
- **assets/templates/analysis_report_template_ja.md** - 分析レポートテンプレート
- **assets/templates/bugfix_task_template_ja.md** - バグ修正タスクテンプレート
- **references/analysis_guide_ja.md** - 分析ガイドライン

### sdd/sdd-document-management/（非推奨 → reqctl.py validate）
- **SKILL.md** - スキル定義ファイル（5機能の定義・承認フロー）
- **assets/templates/consistency_report_template_ja.md** - 整合性レポートテンプレート
- **assets/templates/sync_report_template_ja.md** - 同期レポートテンプレート
- **assets/templates/archive_report_template_ja.md** - アーカイブレポートテンプレート
- **assets/templates/optimize_report_template_ja.md** - 最適化レポートテンプレート
- **references/management_guide_ja.md** - 管理ガイドライン
- **references/consistency_check_ja.md** - 整合性チェックガイド
- **references/sync_check_ja.md** - 同期チェックガイド
- **references/archive_ja.md** - アーカイブガイド
- **references/optimize_ja.md** - 最適化ガイド
- **references/claude_md_sync_ja.md** - CLAUDE.md同期ガイド

### sdd/requirement-management/
- **SKILL.md** - スキル定義ファイル（要求・ストーリー・理由・用語・関係の管理／矛盾の機械検証／承認フロー）
- **scripts/reqctl.py** - 要求レジストリの検証・生成・影響分析ツール（利用側リポジトリの `scripts/` にコピーし、プロジェクトルートから `python3 scripts/reqctl.py` で実行する。コピーしない場合はスキル内の実体パスを指定する）。サブコマンド: `validate`（構造/参照/矛盾/カバレッジ検査、`--strict`で警告もエラー扱い、`--check-tests`でテスト実在確認、`--json`で機械可読出力）/ `generate`（index.md・traceability.md・graph.mdを生成）/ `impact`（変更前の影響範囲）/ `next-id` / `stats`。依存はPyYAMLのみ

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [windschord/claude_skils](https://github.com/windschord/claude_skils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-08 -->
