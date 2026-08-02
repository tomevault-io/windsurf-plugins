---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

このファイルは、drawio業務フロー図作成プロジェクトにおけるClaude Codeへのガイドラインです。

<!-- ============================================================
 CLAUDE.md  ── drawio業務フロー図作成プロジェクト
 Last updated: 2025-06-10
=============================================================== -->

## 🎯 プロジェクトの目的
- **成果物**: SOX法対応の業務フロー図（drawio形式）
- **対象**: 内部統制・監査部門向け業務プロセス文書
- **規格**: J-SOX（金融商品取引法）準拠の内部統制文書

---

## 📊 業務フロー図の基本構造

### スイムレーン構成例（左→右）必要に応じて加除する
1. **社外** - 顧客、仕入先、配送業者など
2. **店舗担当者** - 販売員、レジ担当、売場担当など
3. **店舗管理者** - 店長、副店長、売場主任など
4. **本社営業担当者** - 商品企画、仕入担当、販促担当など
5. **本社営業管理者** - 営業部長、商品部長、販促部長など
6. **本社管理部門担当者** - 経理、人事、総務などの実務担当者
7. **本社管理部門承認者** - 経理部長、人事部長、CFOなど
8. **社内システム** - POSシステム、在庫管理システム、ERPなど
9. **社外サービス** - 決済代行、配送業者API、外部サービスなど

### drawio設定
```xml
<!-- ページ設定 -->
<mxGraphModel grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="1654" pageHeight="1169" math="0" shadow="0">
  <!-- A3横向き: 1654x1169px -->
</mxGraphModel>
```

---

## 🔍 SOX統制要素

### リスク分類
| 記号 | リスク種別 | 色 | 説明 |
|------|-----------|-----|------|
| **R1** | 財務リスク | #FFE6E6 | 財務報告の虚偽記載リスク |
| **R2** | 業務リスク | #FFF0E6 | 業務プロセスの不正・誤謬リスク |
| **R3** | コンプライアンスリスク | #E6F0FF | 法令違反リスク |
| **R4** | ITリスク | #F0E6FF | システム関連リスク |

### コントロール分類
| 記号 | コントロール種別 | 色 | 説明 |
|------|-----------------|-----|------|
| **C1** | 予防的統制 | #E6FFE6 | 事前にリスクを防止 |
| **C2** | 発見的統制 | #E6F5FF | 事後的にリスクを発見 |
| **KC** | キーコントロール | #FFD700 | 重要な統制（★マーク付） |

### IT統制分類
| 記号 | IT統制種別 | 色 | 説明 |
|------|-----------|-----|------|
| **IT-G** | IT全般統制 | #D3D3D3 | システム全体の統制 |
| **IT-A** | IT業務処理統制 | #B0E0E6 | 自動化された業務統制 |

---

## 📐 フロー図作成ルール

### 基本要素
1. **プロセス**: 角丸長方形（通常業務）
2. **判断**: ひし形（分岐・承認判断）
3. **文書**: 波型下辺の長方形（帳票・文書）
4. **データ**: 平行四辺形（データ入出力）
5. **システム処理**: 長方形（自動処理）

### 統制ポイントの表記（付箋形式）
```xml
<!-- プロセスボックスの例 -->
<mxCell id="process-001" value="販売処理&lt;br&gt;(P-001)" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#DAEEFF;strokeColor=#000000;" parent="4" vertex="1">
  <mxGeometry x="25" y="140" width="120" height="60" as="geometry" />
</mxCell>

<!-- リスク付箋の例 -->
<mxCell id="risk-001" value="R-001&lt;br&gt;金額誤り" style="shape=note;whiteSpace=wrap;html=1;backgroundOutline=1;darkOpacity=0.05;fillColor=#FFE6E6;strokeColor=#B85450;size=15;fontSize=10;" parent="4" vertex="1">
  <mxGeometry x="150" y="130" width="60" height="40" as="geometry" />
</mxCell>

<!-- 通常統制付箋の例 -->
<mxCell id="control-001" value="C-001&lt;br&gt;金額確認" style="shape=note;whiteSpace=wrap;html=1;backgroundOutline=1;darkOpacity=0.05;fillColor=#E6FFE6;strokeColor=#82B366;size=15;fontSize=10;" parent="4" vertex="1">
  <mxGeometry x="150" y="175" width="60" height="40" as="geometry" />
</mxCell>

<!-- キーコントロール付箋の例 -->
<mxCell id="control-002" value="C-002 ★&lt;br&gt;現金実査" style="shape=note;whiteSpace=wrap;html=1;backgroundOutline=1;darkOpacity=0.05;fillColor=#FFD700;strokeColor=#C8AB37;size=15;fontSize=10;fontStyle=1;" parent="4" vertex="1">
  <mxGeometry x="-40" y="250" width="60" height="40" as="geometry" />
</mxCell>

<!-- IT統制付箋の例 -->
<mxCell id="control-008" value="C-008&lt;br&gt;自動集計&lt;br&gt;[IT統制]" style="shape=note;whiteSpace=wrap;html=1;backgroundOutline=1;darkOpacity=0.05;fillColor=#B0E0E6;strokeColor=#5D7F99;size=15;fontSize=10;" parent="6" vertex="1">
  <mxGeometry x="-40" y="560" width="60" height="40" as="geometry" />
</mxCell>
```

### 接続線のルール
- **実線**: 通常のフロー
- **破線**: 情報の参照・照会
- **太線**: キーコントロールを含むフロー
- **赤線**: リスクの高いフロー

---

## 📝 レジェンド構成

### 必須レジェンド項目
```xml
<mxCell value="【凡例】" style="text;html=1;fontSize=14;fontStyle=1;" vertex="1" parent="1">
  <mxGeometry x="20" y="20" width="100" height="30" as="geometry"/>
</mxCell>

<!-- プロセス記号 -->
<mxCell value="通常プロセス" style="shape=process;fillColor=#E6F3FF;" vertex="1" parent="1">
  <mxGeometry x="20" y="60" width="100" height="40" as="geometry"/>
</mxCell>

<!-- リスク記号 -->
<mxCell value="R1: 財務リスク" style="shape=hexagon;fillColor=#FFE6E6;" vertex="1" parent="1">
  <mxGeometry x="20" y="110" width="100" height="40" as="geometry"/>
</mxCell>

<!-- コントロール記号 -->
<mxCell value="C1: 予防的統制" style="shape=parallelogram;fillColor=#E6FFE6;" vertex="1" parent="1">
  <mxGeometry x="20" y="160" width="100" height="40" as="geometry"/>
</mxCell>

<!-- キーコントロール -->
<mxCell value="KC: キーコントロール ★" style="shape=process;fillColor=#FFD700;strokeWidth=2;" vertex="1" parent="1">
  <mxGeometry x="20" y="210" width="150" height="40" as="geometry"/>
</mxCell>

<!-- IT統制 -->
<mxCell value="IT-A: 自動統制" style="shape=rectangle;fillColor=#B0E0E6;" vertex="1" parent="1">
  <mxGeometry x="20" y="260" width="100" height="40" as="geometry"/>
</mxCell>
```

---

## 🎨 配色ガイドライン

### スイムレーン背景色
- **社外**: #F5F5F5（薄いグレー）
- **フロント**: #E6F3FF（薄い青）
- **ミドルオフィス**: #F0F8E6（薄い緑）
- **バックオフィス**: #FFF0E6（薄いオレンジ）
- **システム**: #F0E6FF（薄い紫）

### プロセス要素の配色
- **通常プロセス**: 各スイムレーンの背景色を少し濃くした色
  - フロント系: #DAEEFF
  - ミドル系: #E6F0DC
  - バック系: #FFE8D8
  - システム系: #E8D8FF
- **承認・判断**: #FFE6E6（薄い赤）

### 付箋（リスク・コントロール）の配色
- **リスク**: #FFE6E6（薄い赤）、枠線 #B85450
- **通常統制**: #E6FFE6（薄い緑）、枠線 #82B366
- **キーコントロール**: #FFD700（金色）、枠線 #C8AB37
- **IT統制**: #B0E0E6（薄い青）、枠線 #5D7F99

### テキストルール
- **フォント**: メイリオ または ヒラギノ角ゴ

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomiyamaluca/CC_Internal_Control](https://github.com/tomiyamaluca/CC_Internal_Control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
