---
trigger: always_on
description: このプロジェクトは **Minecraft Forge 1.10.2 で作成された Mod を 1.20.1 へ移植する作業** です。
---

# GitHub Copilot Agent Instructions
# Minecraft Forge Mod 移植プロジェクト（1.10.2 → 1.20.1）

---

## プロジェクト概要

このプロジェクトは **Minecraft Forge 1.10.2 で作成された Mod を 1.20.1 へ移植する作業** です。
バージョン間のギャップは約 10 バージョン分に及び、Forge・Minecraft 双方で多数の破壊的変更が存在します。
また、このプロジェクトは **進行途中であり、既存コードには未完成・不整合・暫定実装が含まれています。**

### 移植の最重要原則

> **「コンパイルが通ること」「旧 API が残っていないこと」は移植完了の条件ではない。**
> **移植元（1.10.2）と同一の挙動が再現できていることが唯一のゴールである。**

Copilot は常に「移植元ではどう動くか」を基準として作業してください。
移植元の挙動が確認できない場合は、実装を完了とみなさず、必ず未確認であることを明示してください。

---

## 0. 作業フロー（すべての編集作業で必ず従う）

### PHASE 1: 移植元の挙動を確認する（編集前・必須）

対象ファイルを編集する前に、**移植元（1.10.2）の対応するコードを必ず参照し、以下を文書化してください。**
移植元コードが手元にない場合は、その旨を報告して作業を止めてください。

```
📖 移植元の挙動確認: <クラス名#メソッド名>

【移植元コード（1.10.2）の要約】
  - 何をするメソッド/クラスか:
  - 入力: <引数・前提条件>
  - 出力・副作用: <戻り値・状態変化・イベント発火・NBT 操作など>
  - 呼び出し元・呼び出し条件: <どのタイミングで呼ばれるか>
  - 特記事項: <エッジケース・null 条件・サーバー/クライアントの違いなど>
```

この確認なしに実装を始めることを禁止します。

### PHASE 2: ファイルスキャン（編集前・必須）

以下のパターンをすべて検索し、件数と内容を報告してください：

```
【未実装スキャン】
  // TODO  // FIXME  // HACK  // PORT?  // BEHAVIOR?  // RENDER?  // TODO(Copilot)
  throw new UnsupportedOperationException
  インターフェース実装メソッドで本体が {} のみ

【旧API残存スキャン（描画系）】
  GL11.  GL13.  GL14.  GL20.
  GlStateManager.enableAlpha    GlStateManager.disableAlpha
  GlStateManager.enableLighting GlStateManager.disableLighting
  GlStateManager.color(         GlStateManager.translate(
  TileEntitySpecialRenderer     WorldRenderer（旧）
  RenderHelper.

【旧API残存スキャン（ロジック系）】
  world.isRemote        GameRegistry.register
  NBTTagCompound        NBTTagList        IBlockState
  EnumFacing            EntityPlayer      TileEntity       ITickable
  TextComponentString   TextComponentTranslation
  NetworkRegistry.INSTANCE.newSimpleChannel
  DamageSource.GENERIC  DamageSource.MAGIC  （静的フィールド参照）
  entity.attackEntityFrom(    entity.remove()    world.spawnEntity(
  new Configuration(

【クライアント/サーバー混在スキャン】
  @OnlyIn なしで Minecraft.getInstance() が呼ばれていないか
  DistExecutor で適切に分岐されているか
```

### PHASE 3: 実装する

移植元の挙動確認（PHASE 1）を参照しながら実装してください。
実装中に移植元との差異が生じる場合は、その理由を `// [PORT]` コメントで明記してください。

### PHASE 4: 再現度を自己評価する（実装後・必須）

**実装が完了したと判断した時点で、必ず以下のフォーマットで再現度を自己評価して報告してください。**
「おそらく動く」「たぶん大丈夫」は認めません。根拠のない項目は `未確認` と記載してください。

```
📊 再現度レポート: <クラス名 / 機能名>

【挙動の再現状況】
  ✅ 再現確認済み  : <具体的な挙動と確認根拠>
  ⚠️  要検証      : <何が確認できていないか、どう確認すべきか>
  ❌ 再現できていない: <何が違うか、なぜ違うか>
  ❓ 移植元挙動不明 : <何がわからないか>

【再現度スコア】（確認済み項目 / 全項目）
  ロジック:   ___ / ___
  副作用:     ___ / ___
  エッジケース: ___ / ___
  描画:       ___ / ___  （該当する場合）

【未解決の問題】
  - <問題1>: <推奨アクション>
  - <問題2>: <推奨アクション>
```

スコアが低い・未確認項目がある場合は、完了とみなさず `// [REPRO?]` コメントを付与してください。

---

## 1. 再現度検証の具体的な観点

### 1-1. ロジックの再現

以下の観点で移植元との一致を確認してください：

| 観点 | 確認内容 |
|------|----------|
| **戻り値** | 同じ入力に対して同じ値を返すか |
| **副作用** | 状態変化・NBT 書き込み・イベント発火のタイミングと内容が一致するか |
| **条件分岐** | サーバー/クライアント・null チェック・範囲チェックの条件が同一か |
| **数値** | ダメージ・速度・距離・確率などの定数が移植元と一致するか |
| **NBT キー** | 読み書きするキー名・型が移植元と一致するか（変更はセーブ破壊につながる） |
| **エッジケース** | 空スタック・null エンティティ・未ロードチャンクなどの境界値処理 |

### 1-2. 描画の再現

描画コードは GameTest で自動検証できないため、以下の観点を **`docs/visual_checklist.md`** に記録してください：

```
【ブロック描画】
  □ 通常状態のテクスチャが正しく表示される
  □ BlockEntity のアニメーションが移植元と一致して動作する
  □ 状態変化（ON/OFF など）に応じたモデル切り替えが動作する

【エンティティ描画】
  □ モデルの形状・サイズが移植元と一致する
  □ アニメーション（歩き・攻撃・アイドルなど）が一致する
  □ カスタムレイヤー（装備・エフェクトなど）が正しく表示される
  □ 視点距離による LOD（詳細度）が意図通り動作する

【GUI】
  □ レイアウト・座標が移植元と一致する
  □ スロット・ボタンの動作が一致する
  □ テキスト・フォントが正しく表示される

【パーティクル・エフェクト】
  □ スポーン位置・頻度・サイズが移植元と一致する
```

描画の再現が確認できていない箇所には：

```java
// [RENDER?] 目視検証必須: <移植元の見た目の説明> / <何が変わった可能性があるか>
```

### 1-3. よくある「再現度が低いパターン」

以下は特に再現漏れが起きやすいケースです。実装後に必ず確認してください：

#### 数値・定数の暗黙的変化

```java
// 移植元では整数ダメージ（1.10.2 は半ハート単位）だったが
// 1.20.1 では浮動小数点で扱う → 端数処理の違いに注意
// [PORT] 1.10.2 -> 1.20.1: ダメージ計算の精度変化を要確認
entity.hurt(level.damageSources().magic(), damage);
```

#### DamageSource の変更（1.20〜）

```java
// ❌ 静的フィールドが廃止
entity.attackEntityFrom(DamageSource.MAGIC, 5.0f);

// ✅ レベルのファクトリから生成
entity.hurt(level.damageSources().magic(), 5.0f);
// カスタムダメージは DamageType をレジストリに登録
```

#### イベントのキャンセル・タイミング変化

| イベント | 変化・注意点 |
|----------|-------------|
| `LivingDeathEvent` | キャンセル後のドロップ・経験値挙動が変わった |
| `PlayerInteractEvent` | LEFT/RIGHT サブイベント構成が変更 |
| `TickEvent.LevelTickEvent` | 旧 `WorldTickEvent`。`world` → `level` フィールド |
| `ChunkEvent.Load` | 非同期処理によりタイミングが変化 |

#### NBT のサイレント失敗

```java
// 型不一致で例外が出ず 0 / false / "" が返る → 移植元と動作が変わる
// ✅ 型を指定した安全な確認
if (nbt.contains("myKey", Tag.TAG_INT)) {
    int val = nbt.getInt("myKey");
}
```

#### `ItemStack` の null 扱い変化

```java
// ❌ 1.10.2 では null チェックが必要だったが 1.20.1 では isEmpty() を使う
if (stack == null) { ... }          // ❌ 禁止
if (stack.isEmpty()) { ... }        // ✅
```

#### エンティティライフサイクルの変化

```java
entity.remove();     // ❌ → entity.discard(); ✅
world.spawnEntity(); // ❌ → level.addFreshEntity(); ✅
```

---

## 2. コメント記法（トレーサビリティ）

| 記法 | 意味 |
|------|------|
| `// [PORT] 1.10.2 -> 1.20.1: <理由>` | 移植変更の記録 |
| `// [PORT] 1.10.2 -> 1.13 -> 1.17 -> 1.20.1: <理由>` | 中間バージョン経由の変更 |
| `// [PORT?] <不明な点>` | 移植元の挙動が不明で要確認 |
| `// [BEHAVIOR?] <差分の説明>` | コンパイルは通るが動作等価性が不明 |
| `// [RENDER?] <確認すべき内容>` | 描画の目視確認が必要 |
| `// [REPRO?] <再現できていない内容>` | 再現度レポートで未解決のまま残った箇所 |
| `// TODO(Copilot): 暫定実装。<理由>` | Copilot が生成した暫定コード |

---

## 3. バージョン間差分の吸収（1.10.2 → 1.20.1）

### 3-1. 命名規則の大変更（MCP → Mojmap、1.17〜）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kousakirai/ShinColle-Reforge](https://github.com/kousakirai/ShinColle-Reforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
