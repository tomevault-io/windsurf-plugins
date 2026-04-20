---
trigger: always_on
description: これは、vanilla JavaScript、HTML5、Tailwind CSSで構築されたシンプルなクライアントサイドのみのToDoリストWebアプリケーションです。データの永続化には`localStorage`を使用し、バックエンドサーバーやビルドプロセスは不要です。
---

# CLAUDE.md - ToDoリストアプリケーションのAIアシスタントガイド

## プロジェクト概要

これは、vanilla JavaScript、HTML5、Tailwind CSSで構築されたシンプルなクライアントサイドのみのToDoリストWebアプリケーションです。データの永続化には`localStorage`を使用し、バックエンドサーバーやビルドプロセスは不要です。

**言語**: 日本語（UIとドキュメント）
**種類**: 教育/デモンストレーションプロジェクト
**ステータス**: 開発中

## クイックスタート

1. ブラウザで`login.html`を開く
2. デフォルト認証情報：ユーザー名 `user`、パスワード `pass`
3. ビルドツール、パッケージマネージャー、サーバーは不要

## アーキテクチャ

### 技術スタック

- **HTML5**: 日本語対応のセマンティックマークアップ（`lang="ja"`）
- **JavaScript ES6**: モダンな構文を使用したVanilla JavaScript
- **Tailwind CSS**: CDN経由（https://cdn.tailwindcss.com）
- **ストレージ**: データ永続化にブラウザの`localStorage`を使用

### アプリケーションタイプ

- **クライアントサイドのみ**: バックエンド、API、データベースなし
- **マルチページアプリケーション（MPA）**: ページ遷移時にリロードする3つの独立したHTMLファイル
- **状態管理**: `localStorage`が唯一の真実の情報源として機能

### データフロー

```
ユーザーアクション → JavaScript関数 → localStorage → DOMの再レンダリング
                                      ↓
                              セッション間で永続化
```

## ファイル構成

```
todo-list/
├── login.html              # ログインページ
├── todolist.html           # ToDo一覧ビュー（メインページ）
├── edit.html               # ToDo作成/編集フォーム
├── js/
│   └── app.js              # すべてのJavaScriptロジック（192行）
├── .github/
│   └── copilot-instructions.md   # GitHub Copilot AIガイド
├── .gemini/
│   └── styleguide.md             # Gemini AIガイド（日本語応答）
├── README.md               # ユーザー向けドキュメント（日本語）
└── CLAUDE.md              # このファイル（Claude AIガイド）
```

### HTMLファイル

すべてのHTMLファイルは共通のパターンを持っています：
- UTF-8エンコーディングと日本語サポート
- レスポンシブなviewportメタタグ
- `<head>`内にTailwind CSS CDN
- `</body>`の直前に`js/app.js`をインポート
- Tailwindユーティリティクラスを使用したクリーンでセマンティックなマークアップ

**login.html**
- シンプルな中央寄せのログインフォーム
- ユーザー名とパスワードの入力欄（両方必須）
- エラーメッセージコンテナ（デフォルトで非表示）
- フォームID: `loginForm`

**todolist.html**
- タイトルとログアウトボタンを含むヘッダー
- `edit.html`へのリンク「新規作成」ボタン
- 空の`<ul id="todoList">`コンテナ（JavaScriptで生成）

**edit.html**
- ToDo作成/編集用の中央寄せフォーム
- タイトル入力（必須）と詳細テキストエリア（任意）
- フォームID: `todoForm`
- クエリパラメータ`?idx=N`は編集モードを示す

## JavaScriptアーキテクチャ（app.js）

### コード構成

`app.js`ファイルは論理的なセクションに分かれています：

1. **認証関数**（5-25行目）
2. **データ管理関数**（27-35行目）
3. **ページ初期化**（37-103行目）
4. **レンダリングとアクション**（105-191行目）

### 主要な関数

#### 認証

```javascript
checkLogin()
```
- `localStorage.getItem('todo_login') === '1'`を検証
- 認証されていない場合、`login.html`にリダイレクト
- 保護されたページの読み込み時に呼び出される

```javascript
login(username, password)
```
- **ハードコードされた認証情報**: `user` / `pass`
- 成功時に`true`、失敗時に`false`を返す
- `localStorage.setItem('todo_login', '1')`を設定
- **セキュリティ注意**: 本番環境には不適切

```javascript
logout()
```
- localStorageから`todo_login`を削除
- `login.html`にリダイレクト

#### データ管理

```javascript
getTodos()
```
- `localStorage.getItem('todos')`からパース済み配列を返す
- データが存在しない場合は空配列`[]`をデフォルトとする
- **場所**: js/app.js:29

```javascript
saveTodos(todos)
```
- 配列を文字列化して`localStorage.setItem('todos', ...)`に保存
- **場所**: js/app.js:33

#### ページ固有のロジック

アプリは単一のDOMContentLoadedイベントリスナーを使用し、特定のDOM要素の存在に基づいて実行するページロジックを決定します：

**ログインページ**（50-64行目）
- `#loginForm`を検出
- フォーム送信を処理
- ログイン失敗時にエラーメッセージを表示
- 認証チェックをスキップして早期に戻る

**編集ページ**（72-96行目）
- `#todoForm`を検出
- 編集モード用の`?idx=`クエリパラメータを読み取り
- 既存のToDoを編集する場合、フォームフィールドを事前入力
- 更新時に`checked`プロパティを保持
- 保存後、`todolist.html`にリダイレクト

**ToDo一覧ページ**（99-102行目）
- `#todoList`を検出
- `renderTodos()`を呼び出してリストを生成

#### レンダリングとアクション

```javascript
renderTodos()
```
- `#todoList` DOMをクリアして再構築
- プログラム的に要素を作成（ユーザーデータにinnerHTMLを使用しない）
- **セキュリティ**: XSS防止のため`innerHTML`の代わりに`textContent`を使用
- チェックボックスと削除ボタンにイベントリスナーを追加
- 空の場合は「ToDoがありません。」を表示
- **場所**: js/app.js:109

```javascript
toggleCheck(idx)
```
- インデックスのToDoの`checked`ブール値を切り替え
- 保存して再レンダリング
- **場所**: js/app.js:176

```javascript
deleteTodo(idx)
```
- 確認ダイアログを表示：「本当に削除しますか？」
- `Array.splice(idx, 1)`を使用して削除
- 保存して再レンダリング
- **場所**: js/app.js:185

## データ構造

### localStorageキー

| キー | 型 | 説明 |
|-----|------|-------------|
| `todo_login` | String | 認証フラグ（`'1'` = ログイン済み） |
| `todos` | JSON String | ToDoオブジェクトのシリアライズされた配列 |

### ToDoオブジェクトスキーマ

```javascript
{
  title: String,    // 必須。ToDoタイトル/名前
  detail: String,   // 任意。追加の詳細
  checked: Boolean  // 完了ステータス（デフォルト: false）
}
```

**例**:
```javascript
[
  {
    title: "買い物に行く",
    detail: "牛乳、卵、パンを買う",
    checked: false
  },
  {
    title: "レポート提出",
    detail: "",
    checked: true
  }
]
```

## コーディング規約

### JavaScriptスタイル

- **ES6構文**: アロー関数、テンプレートリテラル、分割代入
- **セミコロンなし**: 任意（このプロジェクトでは一貫して使用）
- **DOM操作**:
  - 要素選択には`document.getElementById()`を使用
  - `document.createElement()`で要素を作成
  - ユーザー生成コンテンツには`textContent`を使用（`innerHTML`は使用しない）
- **イベント処理**:
  - インライン`onclick`属性よりも`addEventListener()`を優先
  - DOMContentLoadedコールバックまたはレンダリング中にリスナーを追加
- **エラー処理**: 最小限（ブラウザのデフォルトに依存）

### セキュリティプラクティス

✅ **コードが適切に行っていること**:
- ユーザーデータをレンダリングする際にXSSを防ぐために`textContent`を使用
- `eval()`や危険なパターンを使用していない

⚠️ **既知の制限事項**（デモ/学習プロジェクトとして許容）:
- ソースコード内にハードコードされた認証情報
- CSRF保護なし（クライアントサイドのみなので該当なし）
- 入力のサニタイゼーションなし（textContentに依存）
- 認証状態がプレーンなlocalStorageに保存

### HTML/CSSパターン

- **Tailwindクラス**: ユーティリティファーストのアプローチ
- **レスポンシブ**: Tailwindレスポンシブユーティリティとflexboxを使用
- **アクセシビリティ**: 基本的なフォームラベルのみ、改善の余地あり
- **クラス命名**: Tailwindユーティリティのみ（カスタムCSSなし）

## 開発ワークフロー

### 変更を加える

1. **HTML/JSファイルを直接編集** エディタで編集
2. **ブラウザをリフレッシュ** 変更を確認（ビルドステップなし）
3. **手動テスト** ブラウザDevToolsで実施
4. **明確なメッセージでコミット** （下記のGit規約を参照）

### デバッグ

- **コンソールログ**: `console.log()`文を追加
- **DevTools**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TW-Ogawa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
