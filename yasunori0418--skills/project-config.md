---
trigger: always_on
description: 複数の AI エージェント（Claude Code / Codex 等）共通の作業ガイド。
---

# AGENTS.md

複数の AI エージェント（Claude Code / Codex 等）共通の作業ガイド。
`CLAUDE.md` は本ファイルへの symlink であり、**編集は常に `AGENTS.md` に対して行う**
（二重管理・乖離を防ぐため）。利用者向けの説明は [README.md](./README.md) を参照。

## このリポジトリは何か

AI エージェント向けスキルを管理するリポジトリ。スキルは 3 つのレイヤーを併用する。
加えて Claude Code 限定で、スキルに紐づく任意のサブエージェント層を持てる。

| レイヤー | 担当 | 実体 |
| --- | --- | --- |
| スキルの中身 | [agentskills.io](https://agentskills.io/specification) 標準 | `skills/<category>/<skill-name>/SKILL.md` |
| 配布・パッケージング | Claude Code plugin | per-category `skills/<category>/.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json`（16 プラグイン） |
| Codex 連携 | 本リポジトリの慣習 | per-skill `agents/openai.yaml` |
| claude-code サブエージェント（任意） | Claude Code plugin | per-skill `agents/<name>.md` |
| 運用 hook | Claude Code plugin | skill 非依存: hook 単位プラグイン `hooks/<plugin>/hooks/hooks.json` / skill 連動: `skills/<category>/hooks/hooks.json` |

## プラグイン構成（カテゴリ別 N plugins）

1 マーケットプレイス（`.claude-plugin/marketplace.json`）に 16 プラグイン
（カテゴリ 8 + hook 7 + バンドル 1）を列挙し、利用者はカテゴリ単位・hook 単位・
バンドル単位で install できる。

- **カテゴリプラグイン**（8 個）: `skills/<category>/` が各プラグイン root。
  `skills/<category>/.claude-plugin/plugin.json`（`name: "<category>-skills"`）を置き、
  marketplace の `source` は `./skills/<category>`。`skills` / `agents` 参照は
  **category root からの相対**（`./<skill-name>` / `./<skill-name>/agents/<name>.md`）。
- **hook プラグイン**（7 個）: skill 非依存の guard/通知 hook を **hook 単位**で
  独立プラグイン化し、利用者が個別に install / on-off できる。各プラグイン root は
  `hooks/<plugin>/`（`yasunori0418-askuserquestion-hooks` /
  `yasunori0418-webfetch-github-guard-hooks` / `yasunori0418-sudo-guard-hooks` /
  `yasunori0418-broad-scan-guard-hooks` / `yasunori0418-notify-stop-hooks` /
  `yasunori0418-task-boundary-hooks` / `yasunori0418-teammate-leak-guard-hooks`）。root 直下に
  `.claude-plugin/plugin.json` と `hooks/hooks.json` を置き、実体は `hooks/<name>/main.sh`。
  marketplace の `source` は `./hooks/<plugin>`。skill は含まない。
- **バンドルプラグイン**（1 個）: 用途別にスキルを束ねる `bundles/<bundle>/` root の
  プラグイン（`shift-left-process`）。中身は各カテゴリ配下のスキル実体への
  **symlink 集約**で、二重管理しない（marketplace 内 symlink の実体解決は公式サポート機構）。
  同名スキルが複数 install 済みプラグインに含まれた場合の挙動は公式未定義のため、
  バンドルと重複するカテゴリプラグインの併用不可を説明文に明記する。
- **hook の所属**: skill に依存しない guard/通知 hook は上記の hook 単位プラグインへ置く
  （関心事ごとに分離し、まとめて有効化しない）。skill とペアで機能する hook
  （例 `git-guard` は rebase-flow/reset-flow の arm marker とペア）は、その skill の
  カテゴリプラグインに `skills/<category>/hooks/` として同居させる。

## ディレクトリ規約

- スキルは `skills/<category>/<skill-name>/` 形式でカテゴリ配下に置く。
- `SKILL.md` の frontmatter `name` は **親ディレクトリ名（`<skill-name>`）と一致**させる。
- スキルに紐づく claude-code サブエージェントは `skills/<category>/<skill-name>/agents/<name>.md` に置く。
  `agents/` は「そのスキルのエージェント連携置き場」で、Codex 用 `openai.yaml` と
  Claude 用 `*.md` が同居する。`.md` の frontmatter `name` はファイル名 stem と一致させる。
- 公開スキルは `skills/` 配下に隔離する。インフラ（`flake.nix` / `dev/` / `pkgs/` /
  `schema/` / `scripts/` / `hooks/` / `bundles/` / `.github/` / `.claude-plugin/`）は
  リポジトリ直下。
- plugin hooks は `hooks.json` に定義し、実体は `<name>/main.sh`
  （テストは `<name>/tests/*.test.sh`、`checks.hooks` が実行）。skill 非依存 hook は
  hook 単位プラグイン `hooks/<plugin>/hooks/` 配下、skill 連動 hook は
  `skills/<category>/hooks/` 配下。スクリプト参照は各プラグイン root を
  起点に `${CLAUDE_PLUGIN_ROOT}/hooks/<name>/main.sh` の形で書く。
- 追加の Nix パッケージは callPackage パターンで `pkgs/<pkg>.nix` に置き、`flake.nix` から
  `pkgs.callPackage ./pkgs/<pkg>.nix { }` で取り込む。
  （カテゴリ `skills/nix/`（Nix 系スキル置き場）との衝突を避けるため、インフラ側は `pkgs/`。）

## スキルを作成・編集するときのルール

1. `skills/<category>/<skill-name>/` ディレクトリを新規作成し `SKILL.md` を置く（雛形ディレクトリは
   置いていない。frontmatter は次項に従う）。
2. `SKILL.md` の frontmatter は agentskills.io 標準に従う。
   - 必須: `name`（1-64文字 / 小文字英数とハイフン / 先頭末尾・連続ハイフン不可 / ディレクトリ名と一致）、
     `description`（1-1024文字 / 何をする・いつ使うか）。
   - 任意: `license` / `compatibility`（≤500文字）/ `metadata`（string→string）/
     `allowed-tools`（スペース区切り文字列）。
   - Claude Code 拡張 `disable-model-invocation` / `argument-hint` / `user-invocable` も許可。
   - 検証は公式 `skills-ref`（`pkgs/skills-ref.nix` でビルド）に委譲する。独自の
     frontmatter スキーマは持たない。upstream は Claude Code 拡張フィールドを
     許可しないため、`pkgs/skills-ref.nix` の `postPatch` で `ALLOWED_FIELDS` に
     上記 3 フィールドを追加している（rev 更新時はパッチの追従を確認する）。
3. 本文は ~500 行以内に収め、詳細は `references/` に分割する（progressive disclosure）。
4. Codex 連携が必要なら `agents/openai.yaml` を置く（`interface.display_name` /
   `interface.short_description` 必須。スキーマ: `schema/openai-agent.schema.json`）。不要なら削除。
5. Claude Code のワーカーサブエージェントが必要なら `agents/<name>.md` を置く
   （frontmatter: `name`（ファイル名 stem と一致）/ `description` 必須、任意で
   `tools` / `model` / `color` / `hooks`（agent 専用の PreToolUse 等。スクリプトは
   `${CLAUDE_PLUGIN_ROOT}` 起点で参照する））。**claude-code 専用**で Codex 側に
   等価物は無い（`agents/openai.yaml` とは別物）。不要なら置かない。
6. **既存カテゴリにスキルを足したら、そのカテゴリの
   `skills/<category>/.claude-plugin/plugin.json` の `skills` 配列に `./<skill-name>` を
   追記する**（category root 相対）。カテゴリ別配置のため Claude Code のデフォルト探索
   （`skills/<name>/`）に乗らず、明示登録が必要。**サブエージェント `.md` を追加したら
   同 plugin.json の `agents` 配列に `"./<skill-name>/agents/<name>.md"` を追記する**。
7. **新しいカテゴリを追加したら新しいプラグインを立てる**:
   - `skills/<category>/.claude-plugin/plugin.json` を作成（`name: "<category>-skills"`、
     `skills` / `agents` は category root 相対、`version` / `author` / `license` は既存に揃える）。
   - `.claude-plugin/marketplace.json` の `plugins` 配列へ
     `{ "name": "<category>-skills", "source": "./skills/<category>", … }` を追記

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yasunori0418/skills](https://github.com/yasunori0418/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
