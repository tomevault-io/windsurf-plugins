---
trigger: always_on
description: <!-- FILE: 00_master_rules.mdc START -->
---

# ⚠️ 重要: このファイルは自動生成されています
# ルールを修正する場合は .cursor/rules ディレクトリ内の .mdc ファイルを編集してください
# 直接このファイルを編集しないでください - 変更は上書きされます

<!-- FILE: 00_master_rules.mdc START -->
# ==========================================================
# Agent Template - ドメイン特化エージェント生成フレームワーク
# ==========================================================

framework_overview:
  name: "Agent Template"
  description: "高品質な業務特化エージェントを生成するテンプレートベースフレームワーク。"
  focus_areas:
    - "業界標準準拠"
    - "実務即応性"
    - "手作り品質"


# =========================
# 設計思想
# =========================

design_principles:
  - id: "industry_standard_alignment"
    label: "業界標準準拠"
    description: "BABOK、AMA Marketing BOKなどの認知されたフレームワークに準拠する。"
  - id: "operational_readiness"
    label: "実務即応性"
    description: "現場で直ちに使える詳細なワークフローを提供する。"
  - id: "crafted_quality"
    label: "手作り品質"
    description: "専門知識に基づいた高品質テンプレートを維持する。"


# =========================
# エージェント作成ワークフロー
# =========================

agent_creation_tasks:
  description: "ヒアリングで蓄積した要件と合意事項を基にエージェント作成工程を管理する進捗タスクリスト。ユーザーと共有し、必要に応じて項目を追加・削除する。初回の作成支援時のみ本テンプレートリポジトリの Flow 階層（例: Flow/{{agent_name}}_{{agent_info.domain}}_agent_build/{{meta.year_month}}/{{meta.today}}_agent_creation_tasklist.md）に保存し、要件定義ファイルと逐次同期する。外部タスク機能を併用する場合でも、要件定義ファイルは必ず create_markdown_file で生成・更新することが必須。生成後のエージェントが独立運用する際は、このタスクリストは使用せず、各リポジトリ固有のフローで管理する。"
  default_items:
    - id: "update_tasklist_file"
      detail: "作業開始直後に Flow/{{agent_name}}_{{agent_info.domain}}_agent_build/{{meta.year_month}}/{{meta.today}}_agent_creation_tasklist.md を create_markdown_file で生成し、以降の工程管理と承認事項を記録する。生成場所はテンプレートリポジトリ側に限定し、派生エージェント配下には置かない。"
    - id: "create_requirements_file"
      detail: "タスクリスト生成と同時に {{patterns.flow_day_dir}}/draft_requirements.md（例: Flow/{{meta.year_month}}/{{meta.today}}/{{meta.agent_dir}}/draft_requirements.md）を create_markdown_file で作成し、ヒアリング結果と合意事項の記録先を確保する。外部タスク機能利用時も必須とし、以降の作業で常に参照・更新する。"
    - id: "sync_tasklist_with_requirements"
      detail: "各タスク完了時に Flow/{{agent_name}}_{{agent_info.domain}}_agent_build/{{meta.year_month}}/{{meta.today}}_agent_creation_tasklist.md と {{patterns.draft_requirements}} の双方へ反映内容・判断・未決事項を追記し、ステータスと要件メモの差分を残さないよう同期する。"
    - id: "configure_root_path"
      detail: "`{domain}_paths.mdc` の `root: \"{{templates.root_dir}}/{{agent_name}}\"` を、実際にエージェントフォルダを配置するフルパス（例: `/Users/you/workspace/{{agent_info.domain}}_agent`）へ書き換える。タスクリストに設定したパスを記録し、# ---- 0. ルートディレクトリ コメントに同じ値を残す。"
    - id: "collect_requirements"
      detail: "ヒアリングで得た要件を {{patterns.draft_requirements}} に整理し、タスクリストと同期を取る。"
    - id: "confirm_initial_structure"
      detail: "Flow / Stock / Archived を初期構成として説明し、エージェント要件とフォルダ設計の双方について承認を得る。承認が得られない場合は後続工程へ進まない。"
    - id: "directory_alignment"
      detail: "ディレクトリ構成と {domain}_paths.mdc の整合を確認・調整する。"
    - id: "draft_domain_rules"
      detail: "01_{domain}_initialization.mdc を起点に番号順で NN_{domain}_*.mdc を整備し、質問・テンプレート・アクション・保存先の候補を確定させる。この段階では {domain}_paths.mdc / 00_master_rules.mdc に手を入れず、必要なキーとトリガーをメモしておく。"
    - id: "rule_customization"
      detail: "01番以降のルール整備を終えてから 00_master_rules.mdc と {domain}_paths.mdc を更新し、マスタートリガーとパス定義を反映する。"
    - id: "finalize_master_and_paths"
      detail: "すべての 01〜NN ルールで保存パターンとトリガーが確定した最終局面で {domain}_paths.mdc と 00_master_rules.mdc を一括で整備し、Flow/{{agent_name}}_{{agent_info.domain}}_agent_build タスクリストに完了記録を残す。"
    - id: "cleanup_sample_rules"
      detail: "テンプレートコピー後に残ったサンプルルールファイル（01_sample_*.mdc など）を削除する。"
    - id: "post_completion_choice"
      detail: "ブラッシュアップ継続か、生成フォルダ単体のプライベートGit化かを確認する。"
agent_creation_workflow:
  overview:
    phase_sequence:
      - order: 1
        name: "Discovery"
        purpose: "ヒアリングと要件定義を実施し、合意内容をドキュメント化する。"
      - order: 2
        name: "Build"
        purpose: "スクリプトとテンプレートを用いてルール・パス・ドラフトを整備する。"
      - order: 3
        name: "Validation"
        purpose: "`python3 scripts/validate_rules.py` で構文・パスの整合を確認し、エラーを修正したうえで結果をログ化する。"
      - order: 4
        name: "Release"
        purpose: "バリデーション結果を反映した状態でエージェントを運用に移行するか、追加改善を計画するかを決定する。"
    final_adjustment_note: ".cursor/rules/ で最終調整を行い、Flow→Stock移行を準備する。Flow / Stock / Archived 以外の構成が必要な場合は、この段階でディレクトリと `{domain}_paths.mdc` の同期を確認し、合意した階層に揃える。"
  scenarios:
    - id: "scenario_standard"
      condition: "初回ヒアリングからエージェント生成までを一気通貫で進める標準フロー"
      steps:
        - order: 1
          name: "ヒアリング準備"
          tasks:
            - "作業開始直後に create_markdown_file を用いて Flow/{{agent_name}}_{{agent_info.domain}}_agent_build/{{meta.year_month}}/{{meta.today}}_agent_creation_tasklist.md と {{patterns.draft_requirements}} を生成し、初回ヒアリング内容と承認事項の記録基盤を整える。"
            - "{{patterns.flow_day_dir}}/agent_discovery_brief.md を作成し、課題・対象ユーザー・期待成果を整理する。"
            - "既存資料があれば gather_existing_info を実行し、参照元リンクと取得日時を記録する。"
            - "要件定義と並行して関連ドメインの業界標準フレームワークを Web検索・文献調査で必ず確認し、出典と日時を {{patterns.draft_requirements}} に記録する。"
        - order: 2
          name: "要件定義セッション"
      tasks:
        - "目的・KPI・成功条件を質問し記録する。"
        - "想定利用シナリオと成果物フォーマットを確認する。"
        - "利用予定データソース・外部システム・権限条件を確認する。"
        - "不確定事項を TODO: 形式で列挙し、合意時刻（例: {{meta.timestamp}}）を明記する。"
        - "ヒアリング内容を補完するため、業界標準フレームワークや最新トレンドを Web検索・文献調査で再確認し、出典と取得日時を {{patterns.draft_requirements}} に記録する。"
        - order: 3
          name: "仮設計ドキュメント化"
          tasks:
            - "{{patterns.draft_requirements}} に要件をまとめる。"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ma-san229) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
