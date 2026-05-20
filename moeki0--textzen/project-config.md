---
trigger: always_on
description: TextZen は、マークダウン記法によるノート管理のための、以下の特徴を持つデスクトップアプリケーションです：
---

# TextZen アーキテクチャガイド

## 概要

TextZen は、マークダウン記法によるノート管理のための、以下の特徴を持つデスクトップアプリケーションです：

- **シンプルな UI**: 執筆に集中できるミニマルなインターフェース
- **マークダウン中心**: GFM（GitHub Flavored Markdown）をベースとした拡張構文
- **拡張可能**: プラグインによる機能拡張
- **自動保存**: 変更内容の自動保存
- **内部リンク**: `[[title]]` 構文によるノート間リンク
- **カスタマイズ**: 設定とショートカットのカスタマイズ

## アーキテクチャの基本構成

- **Electron + React**: クロスプラットフォーム対応のデスクトップアプリ
- **IPC パターン**: メインプロセスは `ipcMain.handle` でハンドラを公開し、レンダラーは `window.api` で呼び出し
- **状態管理**: React Context API による状態管理 (`FileListContext`, `FocusContext`, `EditorContext`)
- **CodeMirror エディタ**: カスタムプラグインを使用した拡張可能なエディタ
- **設定管理**: electron-store による永続的な設定管理
- **多言語対応**: react-intl によるローカライゼーション
- **テーマ**: Tailwind CSS を使用したスタイリング
- **セキュリティ**: CSP（Content Security Policy）によるリソース制御

## 主要概念と機能

### ノート管理

- **ファイルベース**: 各ノートは独立したマークダウンファイルとして管理
- **内部リンク**: `[[title]]` 構文によるノート間のナビゲーション
- **バックリンク**: 参照元の追跡と表示
- **全文検索**: 高速な全文検索機能

### 拡張マークダウン

- **数式**: KaTeX を使用した数式レンダリング（`$inline$` と `$$block$$`）
- **図表**: Mermaid.js による図表作成と表示
- **テーブル**: マークダウンテーブルの視覚的なプレビュー
- **コンテキスト認識**: カーソル位置に応じた編集体験の最適化
- **デバウンス保存**: 自動保存と性能最適化の両立

### ユーザーインターフェース

- **サイドバー**: ファイル一覧とナビゲーション
- **エディタ**: 拡張マークダウンエディタ
- **フォーカス管理**: コンテキストに応じたフォーカス切替
- **ショートカットキー**: カスタマイズ可能なキーボードショートカット

## 開発ガイドライン

### コード品質と標準

- **型安全性**: TypeScript による厳格な型チェック
- **コードスタイル**: ESLint と Prettier による一貫したコード形式
- **コンポーネント設計**: 責任の明確な分離と再利用性の高いコンポーネント
- **テスト**: Playwright による E2E テスト
- **ドキュメント**: 明確なコメントと README

### テスト戦略

- **セレクタ**: 実装詳細ではなく、役割（role）やテキストに基づいたセレクタ
- **ユーザー行動**: 実際のユーザー操作をシミュレートしたテスト
- **機能テスト**: 個別機能に焦点を当てたテスト組織
- **視覚的検証**: 要素の表示状態の検証

## 実装パターン例

### IPC 通信パターン

メインプロセスとレンダラープロセス間の安全な通信を確立するパターン。

```typescript
// メインプロセス側
import { ipcMain } from 'electron'

export const setupFilesHandler = (): void => {
  ipcMain.handle('getFiles', async () => {
    try {
      return await getFiles()
    } catch (error) {
      console.error('Error getting files:', error)
      return []
    }
  })
}

// プリロードスクリプト側
import { contextBridge, ipcRenderer } from 'electron'

const api = {
  getFiles: (): Promise<Array<FileType>> => ipcRenderer.invoke('getFiles'),
}

// 安全のため contextBridge を使用
if (process.contextIsolated) {
  contextBridge.exposeInMainWorld('api', api)
}

// レンダラー側 (React)
export const FileList = (): JSX.Element => {
  const [files, setFiles] = useState<Array<File>>([])

  useEffect(() => {
    window.api.getFiles()
      .then(files => setFiles(files))
      .catch(error => console.error('Failed to load files:', error))
  }, [])

  return (
    <div className="file-list">
      {files.map(file => (
        <div key={file.id}>{file.title}</div>
      ))}
    </div>
  )
}
```

### Context API による状態管理

アプリケーション状態を管理し、コンポーネント間で共有するパターン。

```typescript
// コンテキスト定義
import { createContext } from 'react'

export type FocusTarget = 'fileList' | 'editor' | 'search' | null

interface FocusContextType {
  focus: FocusTarget
  setFocus: (target: FocusTarget) => void
  toggleFocus: (target: FocusTarget) => void
}

export const FocusContext = createContext<FocusContextType>({
  focus: null,
  setFocus: () => {},
  toggleFocus: () => {}
})

// プロバイダー実装
export default function App(): JSX.Element {
  const [focus, setFocus] = useState<FocusTarget>('fileList')

  const toggleFocus = (target: FocusTarget): void => {
    setFocus(target === focus ? 'editor' : target)
  }

  return (
    <FocusContext.Provider value={{ focus, setFocus, toggleFocus }}>
      {/* アプリコンポーネント */}
    </FocusContext.Provider>
  )
}

// コンシューマー
export default function Sidebar(): JSX.Element {
  const { focus, setFocus } = useContext(FocusContext)

  return (
    <div
      className={`sidebar ${focus === 'fileList' ? 'focused' : ''}`}
      onClick={() => setFocus('fileList')}
    >
      {/* サイドバー内容 */}
    </div>
  )
}
```

### CodeMirror ViewPlugin パターン

エディタの機能を拡張するためのプラグイン実装パターン。

```typescript
import {
  ViewPlugin,
  DecorationSet,
  Decoration,
  EditorView,
  ViewUpdate,
  WidgetType
} from '@codemirror/view'
import { Range } from '@codemirror/state'

// カスタムウィジェット
class CustomWidget extends WidgetType {
  constructor(private readonly content: string) {
    super()
  }

  eq(other: CustomWidget) {
    return this.content === other.content
  }

  toDOM() {
    const element = document.createElement('div')
    element.className = 'custom-widget'
    element.textContent = this.content
    return element
  }
}

// コンテンツ検出関数
function findCustomContent(view: EditorView): Range<Decoration>[] {
  const ranges: Range<Decoration>[] = []
  const content = view.state.doc.toString()

  // パターンマッチング: [[example]]
  const regex = /\[\[(.*?)\]\]/g
  let match

  while ((match = regex.exec(content)) !== null) {
    // ウィジェット作成
    const widget = Decoration.widget({
      widget: new CustomWidget(match[1]),
      side: 1
    })

    ranges.push(widget.range(match.index + match[0].length))
  }

  return ranges
}

// ViewPlugin 定義
export const customViewPlugin = ViewPlugin.fromClass(
  class {
    decorations: DecorationSet

    constructor(view: EditorView) {
      this.decorations = Decoration.set(findCustomContent(view))
    }

    update(update: ViewUpdate) {
      if (update.docChanged || update.viewportChanged) {
        this.decorations = Decoration.set(findCustomContent(update.view))
      }
    }
  },
  {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moeki0/TextZen](https://github.com/moeki0/TextZen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
