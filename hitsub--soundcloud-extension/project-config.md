---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## これは何か

soundcloud.comにジャケット画像コピーボタンとメタデータタグ付きダウンロード機能を追加する、単一ファイルのTampermonkeyユーザースクリプト（`soundcloud-menu-extension.user.js`）。ビルドシステム、package.json、バンドラー、リンター、テストスイートは無い — 1つのIIFEにまとめたプレーンなvanilla JSで、Tampermonkeyが直接読み込む。ユーザー向けの機能一覧と日本語の利用メモは`README.md`を参照。

## 開発ワークフロー

ビルド/lint/testコマンドは無い — このプロジェクトにはそれらが存在しない。変更を確認する手順:
1. `soundcloud-menu-extension.user.js`を直接編集する。
2. Tampermonkeyのダッシュボードで、スクリプトの中身を更新後の内容に貼り替える（ディスクからインストールしている場合はリロード）。
3. soundcloud.com上で手動テストする — どの機能もログイン済みセッションが必要。

このスクリプトを実ブラウザの外で実行・型チェックする手段は無い（このリポジトリに`node`/バンドラーは無い）。開発環境で`node`が使えない場合は、コードを注意深く読み、バイナリオフセットやDOMセレクタを手で追って検証すること。

## アーキテクチャ

### 2つの機能領域と、共通のボタン/フィードバック基盤

1. **タイル/行オーバーレイのジャケットコピーボタン**（`insertTileButtons`/`createTileButton`） — ネイティブのLike/Follow/More（グリッドタイル）、Like/Repost/Share/Copy Link/More（リスト行、プレイリストのトラック行）、Like/Repost/Share/Copy Link/More（トラック自身の単体ページ）の各ボタンの隣、`.sc-button-more`の直前に挿入される。`ACTION_ROW_CONFIGS`は4種類の異なるDOM形状（グリッド「Badges」表示用の`.playableTile__actionWrapper`、「List」表示用の`.soundActions .sc-button-group`、プレイリスト（`/sets/...`）のトラック行用の`.trackItem .soundActions .sc-button-group`、単体ページ用の`.listenEngagement__footer .soundActions .sc-button-group`）を保持している。これはSoundCloudが文脈によって同じトラックを異なる形でレンダリングするため — 特にプレイリスト行は「List」表示の行とまったく同じボタングループのマークアップを再利用しているが、`.sound__body`/`.sound__artwork`ではなく`.trackItem`/`.trackItem__image`で包まれている。これが`permalinkFromScope()`と各configの`resolveCopy`が、1つの形状だけを前提にせず複数の包み方をチェックする必要がある理由。グリッド/リスト/プレイリストのconfigはタイル/行自身のDOMからジャケット画像のURLを取り出す（`copyArtworkFromTile`）。単体ページのconfigは代わりに`copyArtwork()`（api-v2ベースの取得。そこには読み取れる`.sound__body`タイル/行が無いため）を再利用する。グリッド（「Badges」）タイルのconfigだけがFont Awesomeの塗りつぶしクリップボードアイコン（`ICON_CLIPBOARD_SOLID`）を使う（ジャケット画像の真上に乗るため）。それ以外のconfig（リスト、プレイリスト、単体ページ）はすべて素のアウトラインアイコン（`ICON_IDLE`）を使う。
2. **「Download file with metadata」**（`insertDownloadButtons`/`createDownloadButton`） — ネイティブの`.sc-button-download`ボタンの直後、`.moreActions__group`内に挿入される。`downloadFileWithMetadata()`を呼び出す。

両者は`attachCopyHandler()`を共有しており、これがロード中→成功/失敗のアイコン状態遷移（`showFeedback`、`setIcon`）を制御し、エラー時にはローカライズされた失敗理由をトーストで表示する（`showToast`）。各ボタンは自分自身のアイドル時アイコンを`button._idleIcon`で覚えている（`showFeedback`が全ボタン共通で1つのアイコンにハードコーディングしないように — 実際に一度バグになったことがある）。タイルコピーボタンのホバー時ラベルは、ネイティブの`title`属性ではなく`attachSoundCloudTooltip()`を使っている — SoundCloud自身のツールチップが使うのと同じクラス（`tooltip`/`tooltip__arrow`/`tooltip__content`）でツールチップの吹き出しを組み立てるため、サイト自身のCSSを継承し、ネイティブのLike/Repost/Share/Copy Link/Moreのツールチップと同じ見た目になる（ブラウザの素のOSツールチップではなく）。

3つ目の、これらとは無関係な処理 — `insertPurchaseLinkDomains()` — はそもそもボタンではないので`attachCopyHandler()`を経由しない。あらゆる`.soundActions__purchaseLink`（「OUT NOW」形式の外部購入リンク）を見つけ、そのリンクの遷移先ドメインを表示するプレーンなテキストspanを追加する。`extractLinkDomain()`は、リダイレクト/ゲートサービス（例: `gate.sc?url=<エンコードされた本来のURL>`）を`url`クエリパラメータの有無で判定して解きほぐし、ゲート自身のホスト名の代わりにそちらのホスト名を使う（無ければhref自身のホスト名にフォールバックする）。

### SoundCloudはReact SPA — あらゆるものが継続的に再挿入される

`document.body`に対する単一の`MutationObserver`が、DOMの変更のたびに`insertTileButtons`/`insertDownloadButtons`/`insertPurchaseLinkDomains`を再実行する。理由:
- トラック一覧はユーザーがスクロールするたびに、より多くのタイル/行を遅延読み込みする。
- 「More」ドロップダウンメニューは、開かれるたびに新しくDOMへポータルされる（開いたタイル/行の近くにネストされるわけではない）。

ドロップダウンはトリガーから離れた場所にポータルされるため、`resolveTrackPermalink()`はトリガーボタンの`aria-owns`属性（ドロップダウンの`id`と一致する）を使って、ドロップダウンを自身のタイル/行に対応付け直し、そのタイル/行のジャケット画像リンクまたはタイトルリンクからトラックのpermalinkを読み取る。トリガー/スコープが見つからない場合（つまりそのドロップダウンがトラック自身の単体ページに属する場合）は`location.href`にフォールバックする。これはその場合には正しい答えになる。

### SoundCloudのページHTMLを直接fetchしない — 代わりにapi-v2のAJAXエンドポイントを使う

`fetch(someTrackPageUrl)`は断続的に`m.soundcloud.com`へリダイレクトされCORSでブロックされる — これはSoundCloudのbot対策（DataDome）が、今まさに見ているページであってもスクリプト発のHTML fetchにフラグを立てているため。トラックデータの取得（`fetchTrackData`、`GET api-v2.soundcloud.com/resolve?url=...`経由）とダウンロード処理（`fetchDownloadFile`、`GET api-v2.soundcloud.com/tracks/{id}/download`経由）は両方とも、代わりにこれを引き起こさないapi-v2のJSONエンドポイントを経由する。

これらの呼び出しの認証: `client_id`と`app_version`は、現在のページのライブなグローバル変数（`window.__sc_hydration`の`apiClient`エントリ、`window.__sc_version` — セッション単位でトラックごとではないため、SPAナビゲーションに関わらず常に最新）から`getSessionCredentials()`経由で取得する。それに加えて`Authorization: OAuth <token>`ヘッダー（`authHeaders()`）が必要で、これはhttpOnlyではない`oauth_token`クッキーから取得する — `client_id` + クッキーだけでは401が返る。

### ジャケット画像URLの解決とクリップボードの癖

- `getHighResUrl()`はトラックのサムネイルサイズのURLを最高解像度の`-original`に置き換える。**2種類の異なるサフィックス表記**にマッチさせる必要がある: Webアプリ自身が描画するDOMは`-t{width}x{height}`（例: `-t500x500`）を使うが、api-v2の`/resolve`レスポンスの`artwork_url`フィールドはSoundCloudの古い`-large`表記を使う。どちらか一方でも見逃すと、アップグレードが静かにスキップされ、小さくPNGでないことが多い画像にフォールバックしてしまう。
- Chromeのクリップボード APIが`navigator.clipboard.write()`で保証しているのは`image/png`のみ — 一部のジャケット画像（特に`-original`版が存在しないもの）は`image/jpeg`で配信されており、書き込みがそのまま拒否されることがある（`NotAllowedError: ... Type image/jpeg not supported on write`）。`copyArtworkFromBaseUrl()`は、PNGでないblobを書き込み前に`OffscreenCanvas`/`createImageBitmap`経由で変換する。この変換は意図的に、ダウンロード＆タグ付け機能側のジャケット画像取得（`fetchArtworkBuffer`）には適用していない — そちらは別のコードパスであり、クリップボードのフォーマット制約を受けないため。

### メタデータタグの書き込み（WAV / MP3 / FLAC）

各フォーマットの「既に設定されていればそのまま、無ければタイトル/アーティスト/アルバム/ジャンル（/ジャケット画像）を埋める」というロジックは、フォーマットごとに手書きしたバイナリパース処理であり、コンテナフォーマット同士に関連が無いためフォーマット間で共有する抽象化は無い:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hitsub/soundcloud-extension](https://github.com/hitsub/soundcloud-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
