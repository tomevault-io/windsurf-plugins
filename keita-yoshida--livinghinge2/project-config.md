---
trigger: always_on
description: レーザー加工用カーフベンディング/リビングヒンジパターンを生成するWebアプリ。
---

# リビングヒンジ ジェネレーター — 開発引き継ぎ資料

レーザー加工用カーフベンディング/リビングヒンジパターンを生成するWebアプリ。
現状は単一HTMLファイル(`index.html`)として完結し、WordPressのuploads配下に置くだけで公開できる。

- 公開URL: https://laser-kakouki.com/wp-content/uploads/2025/12/livinghinge.html
- 作業ファイル名は `index.html`(公開時に `livinghinge.html` へリネーム)
- 現行サイズ: 約66KB / 1,431行
- **実行時の外部依存: ゼロ**(v6でGoogle Fonts読み込みを削除。フォントはOS標準のみ)
- 開発時の依存: 検証ツールのみ(Playwright / Chromium。`tools/*.mjs`)

## 方針(v8で転換)

**仕様の安定と出力データの正確性のためなら、「手書きの単一ファイルであること」「ビルドレスであること」は犠牲にしてよい。**

ただし勘違いしないこと ——
**配布物としての単一HTMLとオフライン動作は放棄しない**。ビルド(`vite-plugin-singlefile` 等)で
1枚のHTMLに畳めば、WordPress uploads運用もCDN障害耐性もそのまま維持できる。
実際に手放すのは「手書きの単一ファイルで開発すること」だけである。

---

## 1. 絶対に壊してはいけない契約(Hard Contract)

1. **DXF出力**: 単位mm実寸、レイヤー名 `CUT`。Y軸は反転して出力(`y' = H - y`、DXFはY上向きのため)。
   現行はR12形式(AC1009)で、2点線分は `LINE`、多点は `POLYLINE`+`VERTEX`+`SEQEND`。
   LightBurn / Illustrator / Fusion 360 / Inkscape で読めることを維持する。
   (バージョンの引き上げは §8 P1-5 で検討中。**mm実寸とレイヤー名は形式を変えても不変**)
2. **SVG出力**: `width="{W}mm" height="{H}mm" viewBox="0 0 W H"`(=1ユーザー単位が1mm)、
   `stroke="#FF0000" stroke-width="0.1"` fill無し。テキスト要素は含めない。
3. **配布物は単一HTML・実行時ゼロ依存**。CDNやWebフォントを実行時に読みに行かないこと
   (CDN障害・オフラインで壊れるため。過去に明示的に不採用を決定済み)。
   ビルドで達成するのは可(v8以降)。**ソースが単一ファイルである必要はもう無い**。
4. **設定共有URL**: `location.hash` にJSONを `encodeURIComponent` して保持。
   後方互換を守ること(古いキー、例: 廃止済み `p.web` が入っていても無視して動くこと。テスト済み)。
   新キーを足す場合は、そのキーが**無い**古いhashでもデフォルトにフォールバックすること
   (v6で `mat` を追加。旧hash読み込み時は `mat` 未定義→デフォルト適用をテスト済み)。
5. ブラウザストレージ(localStorage等)は使わない。状態共有はhashのみ。
6. **出力を変える変更は必ずゴールデン照合を通す**(§6.0)。
   `node tools/golden.mjs check` が通らない変更は、出力仕様を意図的に変えた場合を除いてマージしない。
   意図的な変更なら `capture` で採り直し、**差分を必ずレビューしてから**コミットする。

## 2. ファイル構成

```
index.html            アプリ本体(単一ファイル)
├─ <style>            デザイントークン(:root CSS変数)、レイアウト/パネル/ステージ/モバイル
├─ <body>             ヘッダー / 左パネル(パターン・プリセット・サイズ・材料・調整・DL・ヒント)
│                     / ステージ(表示モード切替・ズーム・viewport・統計チップ)
│                     viewport内: <svg #pv> のみ
└─ <script>
   ├─ state / GLOBAL_DEFS / PARAM_DEFS     状態とスライダー定義
   ├─ 幾何ユーティリティ                    clipHalf / clipRect / polyLen / columnXs / centerYs
   ├─ GEN = {straight,wave,diamond,cross,arc,hex,hexslit,ehex,bone,tri,spiral} ← 全11パターン生成器
   ├─ PATTERNS / PRESETS
   ├─ MATERIALS / flexScoreOf / estMinRadius / genPolysFor   材料プロファイルと半径推定(§6.5)
   ├─ regenerate()                          生成→renderView()→統計→警告→hash更新
   ├─ 2Dプレビュー                          renderPreview / fitView / zoomAt(SVG viewBox操作)
   ├─ 入力系                                pointer(パン/ピンチ)/ wheel / ボタン
   ├─ コントロールUI生成                     ctlHTML / bindCtl / buildGlobalCtls / buildParamCtls / buildPatternCards
   ├─ エクスポート                          makeDXF(polys?,H?) / makeSVG(polys?,W?,H?) / download
   │                                        引数省略時は現在の盤面。テストピースは引数渡しで再利用(§7.5)
   ├─ テストピース                          TEST / testPieceVariants / buildTestSheet / downloadTestSheet
   └─ hash共有 / toast / init

tools/
├─ golden.mjs         出力ゴールデンの採取/照合(§6.0)
└─ smoke.mjs          UIスモークテスト(§6.4)

tests/
├─ README.md          ゴールデンの使い方と採取マトリクス
└─ golden/            94ケース × DXF/SVG = 188ファイル + manifest.json
```

ポリラインは `[[x,y],...]` のmm座標。原点は左上、y下向き。`cuts`(生成結果)と `framePoly`(外枠)が描画・出力の唯一のソース。

## 3. パターン生成仕様

### 3.1 共通(列ベース: straight / wave / diamond / cross / arc)
- `columnXs(x0,x1,pitch)`: 領域内に対称配置した列中心X群。
- `centerYs(y0,y1,period,offset)`: period = `cut + gap`。偶数列はoffset 0、奇数列はperiod/2ずらし(千鳥)。
- 生成後は必ず `clipRect` で余白内にクリップ(開ポリライン対応の半平面クリップを4回)。
- diamond/crossは形状幅を `pitch - 0.6〜0.8` で自動制限。

### 3.2 hex(ハニカム)
尖頭六角形(pointy-top)。各六角形を左右2本のポリラインで描き、上下頂点の隣接辺を
`gap/2` ずつトリムしてブリッジを残す。colStep=`√3R+hs`、rowStep=`1.5R+hs*0.87`、奇数行は半列オフセット。

### 3.3 spiral(メアンダー)★最重要
**[4,2] 2Dメアンダーパターン**(Dujam Ivanišević / koFAKTORlab 2014年考案)の忠実な実装。
形式化の出典: Zarrinmehr et al. "Kerfing with Generalized 2D Meander-Patterns" (CAADFutures 2017)。

構造の要点(実装を触る前に必ず理解すること):
- 正方形クワッド格子。各クワッドに**同キラリティの矩形スパイラル2本**を対角アンカーで絡ませる。
- カットは格子頂点で合流し「4本枝のスパイラルツリー」になる。頂点は市松2彩色され、
  片方の色(ツリー頂点)からだけ渦が生え、もう片方(エッジ中点相当)は**無傷のソリッド固定点**として残る。
- ウェブ(セル間の枠)は**存在しない**。材料は固定点同士を結ぶ蛇行アームのみ。これが二重曲率の柔軟性の源。

実装詳細(`GEN.spiral`):
- アーム幅 `sw` はセルサイズ `cs` の**奇数分割に自動スナップ**: `N = round(cs/sw)` を奇数化(最小5)、`p = cs/N`。
  → 渦間隔が全域で完全に均一になる数学的条件。
- 基本パスA(アンカー(0,0)、自ピッチ2p)の頂点列:
  `(0,0) → (S-p,0) → (S-p,S-p) → (2p,S-p) → (2p,2p) → (S-3p,2p) → (S-3p,S-3p) → (4p,…) …`
  (k%4でR/U/L/D、インセットは1周ごとに2p増。線長がp以下になったら終了)
- B = Aの180°回転(同一クワッド内で絡む相手)。
- 奇数パリティのクワッドは `rot90: (x,y)→(S-y,x)` を適用(**ミラーではなく回転**。キラリティを全クワッドで統一するため。ここをミラーに変えると本物と異なる構造になる)。
- 偶数クワッドは水平辺上に、奇数クワッドは垂直辺上にカットを持つため、共有辺のカットは重複しない(設計済み)。

### 3.3.5 bone / hexslit / ehex / tri(v7で追加、fequalsf再現)
fequalsf.com の "parametric kerf" テストフォブDXF(#6/#7, #8, #9)を解析して再現したもの。
DXFはR13(AC1012)・SPLINE主体。**スプラインは短い断片の連鎖なので、端点連結(join)して
初めて本当のモチーフ形状が分かる**(断片単位のbbox統計では誤同定する。v7初版はこれで
縦の鼓形と誤実装し、やり直した)。手順は§6.7。

- **bone**(#6/#7): 参照モチーフは **幅1.205×高さ0.133の滑らかな閉曲線** =
  中央レンズ(半幅A≈0.055×全長) → 細い首(≈0.13A) → 両端の丸バルブ(r≈0.7A)。
  半幅プロファイルを円弧+コサイン補間で構成し、`straight` と同じ列/千鳥配置。
  レンズ内部は脱落する(仕様)。パラメータ pitch/cut/gap/**br**(=レンズ半幅)。
- **hexslit**(#8): **フラットトップ・縦0.8スカッシュの六角形**。各セル =
  **完全に閉じた内六角形(芯は脱落する。参照フォブも同じ)** + 外郭線 + セル間の帯 `hs`。
  パラメータ hr/hs/**rw**(リング幅=外郭−内六角)/gap(ブリッジ幅)。
  参照比率: 内/外 ≈ 0.73 → rw標準1.2(hr=5時)。
  - **外郭は左右2本の長いポリライン**で、切れ目は**上辺・下辺の中央だけ**(各セル2箇所)。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keita-yoshida/LivingHinge2](https://github.com/keita-yoshida/LivingHinge2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
