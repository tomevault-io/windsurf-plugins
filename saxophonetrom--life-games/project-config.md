---
trigger: always_on
description: Farcaster Mini App として動作する、Conway's Game of Life アートプロジェクト。
---

# Farcaster Infinite Life - プロジェクト CLAUDE.md

## プロジェクト概要

Farcaster Mini App として動作する、Conway's Game of Life アートプロジェクト。
64×64、16色のボードで、ユーザーが「SegmentNFT」（独立作品）を作成し、
共有世界線は「EpochArchiveNFT」（256世代ごと）として別途生成される。

### アーキテクチャ（新案: 2025-12-15〜）

#### SegmentNFT（個人作品）
- **空盤面（世代0）から**ユーザーが注入してn世代進めた「独立作品」
- 共有盤面の状態を参照しない
- **即確定mint**（Pending→Finalizeパイプライン不要）
- Mini Appがクライアント側でGIF生成・即時表示

#### EpochArchiveNFT（共有世界線）
- SegmentNFT mint時に発生する**Contributionログ**を時系列順に適用
- **256世代ごと**にMP4アーカイブとしてmint
- 貢献者（Contributors）は該当Epochを無料mint可能

#### Contribution（貢献ログ）
- SegmentNFTのmint時に自動生成
- `tokenId`, `fid`, `nGenerations`, `cellsEncoded`, `blockNumber`
- 共有世界線は Contribution を順番に適用して構築

### コンセプト
- **SegmentNFT**: 空盤面から始まる個人作品（即確定mint）
- **EpochArchiveNFT**: 全員を繋いだ共有世界線のアーカイブ（256世代ごと）
- **Farcaster連携**: FIDをオンチェーン記録

---

## フォルダ構成（モノレポ）

```
200_LifeGames/
├── CLAUDE.md                    # このファイル（プロジェクトルール）
├── .gitignore                   # Git除外設定
├── .env.example                 # 環境変数テンプレート（Git管理）
├── package.json                 # ルートpackage.json（スクリプト用）
├── pnpm-workspace.yaml          # pnpmワークスペース設定
│
├── packages/
│   ├── webapp/                  # Next.js アプリ（Vercelデプロイ対象）
│   │   ├── src/
│   │   │   ├── app/             # App Router
│   │   │   │   ├── layout.tsx
│   │   │   │   ├── page.tsx     # ホーム画面（ボード表示）
│   │   │   │   ├── buy/         # セグメント作成画面
│   │   │   │   ├── segment/[id] # セグメント詳細画面
│   │   │   │   ├── epoch/[id]   # エポック詳細画面
│   │   │   │   └── api/         # API Routes
│   │   │   │       └── token/[id]/route.ts    # NFTメタデータ
│   │   │   ├── components/      # 共通コンポーネント
│   │   │   ├── hooks/           # カスタムフック
│   │   │   ├── lib/             # ユーティリティ
│   │   │   │   ├── life-engine.ts    # Life シミュレーション（決定論的）
│   │   │   │   ├── color-rules.ts    # 16色カラールール
│   │   │   │   ├── state-encoding.ts # StateV1 エンコーディング
│   │   │   │   └── gif-renderer.ts   # クライアント側GIF生成
│   │   │   ├── providers/       # Context Providers
│   │   │   └── types/           # TypeScript型定義
│   │   ├── public/              # 静的ファイル
│   │   └── package.json
│   │
│   └── contracts/               # Foundry プロジェクト
│       ├── foundry.toml
│       ├── src/
│       │   ├── SegmentNFT.sol   # 個人作品NFT（即確定mint）
│       │   └── EpochArchiveNFT.sol  # 共有世界線アーカイブNFT
│       ├── test/
│       ├── script/
│       └── lib/                 # forge dependencies (Git除外)
│
├── .github/
│   └── workflows/
│       └── epoch-generator.yml  # Epoch生成用 Actions（256世代ごと）
│
├── _backup_life_league/         # 旧プロジェクトバックアップ（Git除外）
│
└── docs/
    ├── requirements/
    │   ├── art/
    │   │   ├── farcaster_infinite_life_mvp_spec.md  # MVP仕様書（旧案）
    │   │   ├── detailed_design.md                   # 詳細設計書（旧案）
    │   │   └── spec_diff_new_plan_segment_independent_epoch_archive_ja.md  # 新仕様差分【必読】
    │   └── battle/              # 旧プロジェクト要件（参照用）
    └── TODO.md                  # 開発TODO
```

---

## Vercelデプロイ設定

| 項目 | 値 |
|------|-----|
| Root Directory | `packages/webapp` |
| Build Command | `pnpm build` |
| Output Directory | `.next` |

---

## 要件定義ドキュメント参照【必読】

実装前に必ず以下のドキュメントを確認すること：

| ドキュメント | パス | 内容 |
|-------------|------|------|
| **新仕様差分** | `docs/requirements/art/spec_diff_new_plan_segment_independent_epoch_archive_ja.md` | **最優先で参照**。新アーキテクチャの詳細 |
| ~~MVP仕様書~~ | `docs/requirements/art/farcaster_infinite_life_mvp_spec.md` | 旧案（参考用） |
| ~~詳細設計書~~ | `docs/requirements/art/detailed_design.md` | 旧案（参考用） |

---

## セキュリティルール【絶対厳守】

### 秘密情報のハードコード厳禁

以下の情報は**絶対に**ソースコードにハードコードしてはならない：

- `EPOCH_MINTER_PRIVATE_KEY` - Epoch mint用秘密鍵
- `GITHUB_TOKEN` - GitHub Actions用
- `ALCHEMY_API_KEY` - Alchemy APIキー
- `BASESCAN_API_KEY` - Basescan APIキー
- その他すべてのAPIキー、秘密鍵、トークン

### 環境変数の管理

```
✅ 正しい方法:
   - .env.local に記載（Git除外）
   - Vercelの環境変数設定で管理
   - GitHub Secretsで管理（Actions用）
   - process.env.XXX で参照

❌ 禁止行為:
   - ソースコードへの直接記載
   - .env.example に実際の値を記載
   - コミットメッセージに秘密情報を含める
   - console.log で秘密情報を出力
```

### コミット前チェック

1. `git diff --staged` で秘密情報が含まれていないか
2. `.env` ファイルがステージングされていないか
3. ログ出力に秘密情報が含まれていないか

---

## 重要な仕様【新案】

### ボードパラメータ

| パラメータ | 値 |
|-----------|-----|
| サイズ | 64×64（4,096セル） |
| パレット | 16色（インデックス 0〜15） |
| トポロジー | Moore近傍（8方向） |
| ルール | **HighLife (B36/S23)** - Birth: 3 or 6, Survival: 2-3 |

### SegmentNFT（個人作品）

| パラメータ | 値 |
|-----------|-----|
| nGenerations | 10〜30 |
| 注入セル数上限 | `floor(nGenerations / 2)` |
| セルエンコーディング | 3バイト/セル（x, y, colorIndex） |
| **起点** | **空盤面（世代0）** ← 共有盤面ではない |
| **mint方式** | **即確定**（Pending/Finalize不要） |

### SegmentNFT保持データ（最小）

| フィールド | 説明 |
|-----------|------|
| `fid` | 購入者Farcaster ID |
| `nGenerations` | 世代数 |
| `cellsHash` | keccak256(cellsEncoded) |
| `rulesetHash` | ルールセットハッシュ（将来互換） |
| `mintedAt` | mint時のblock番号 |

### SegmentNFTイベント

```solidity
event SegmentMinted(
  uint256 indexed tokenId,
  address indexed minter,
  uint256 indexed fid,
  uint8 nGenerations,
  bytes32 cellsHash
);

event SegmentCells(
  uint256 indexed tokenId,
  bytes cellsEncoded
);
```

### EpochArchiveNFT（共有世界線）

| パラメータ | 値 |
|-----------|-----|
| 世代範囲 | 256世代ごと（epoch 0: gen 1-256, epoch 1: gen 257-512, ...） |
| 成果物 | MP4動画 |
| 貢献者 | Contributionを通じて寄与したSegment所有者 |

### Epoch保持データ

| フィールド | 説明 |
|-----------|------|
| `absStartGen` | 絶対開始世代 |
| `absEndGen` | 絶対終了世代 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SaxophoneTrom) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
