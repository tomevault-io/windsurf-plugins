---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

技術書を読みながら、気になった箇所を選択して AI に質問できる PDF リーダー。利用者は 1 人。
React 19 (SPA) + Hono (Worker) + D1 + R2 を単一の Cloudflare Workers プロジェクトにまとめ、
`@cloudflare/vite-plugin` で SPA と Worker を同一の `vp dev` で動かす。

**公開先は `https://<worker 名>.<アカウント>.workers.dev`**（`wrangler.jsonc` の `name` と
Cloudflare アカウントのサブドメインで決まる）で、API はログインの内側にある
（下記「ログインとセッション」）。ローカルだけで動かしていた頃の前提（ログイン不要）は
もう成り立たない。

## コマンド

`vp`（Vite+）に統一。生の `vite` / `vitest` は直接叩かない。

```bash
vp dev                    # SPA + Worker を同時起動 (http://localhost:5173)
pnpm run db:migrate:local # D1 マイグレーション適用（初回 / migrations 追加時のみ、自動適用はしない）

pnpm test                 # フロント単体 (jsdom)
pnpm run test:worker      # Worker 単体 (@cloudflare/vitest-pool-workers)
pnpm run test:e2e         # E2E (Playwright)。サーバーは自動起動するので vp dev は不要

vp check                  # フォーマット + lint + 型チェック（--fix で自動修正）
vp exec wrangler types    # wrangler.jsonc の bindings/main 変更後に Env 型を再生成
```

単体テストを1ファイルだけ走らせる: `vp exec vitest run src/front/lib/sseParser.test.ts`
1 つの project だけ走らせる: `pnpm run test:e2e --project=mobile`（`desktop` / `tablet` / `mobile`）
E2E を1件だけ走らせる: `pnpm run test:e2e -g "テスト名の一部"`（`--` を挟むと pnpm が
それをそのまま playwright へ渡し、`-g` が効かないまま全件走る）

`git push` 時に lefthook の `pre-push` が `vp check` + `vp build` を実行し、失敗すると push はブロックされる。

### worktree を作ったら最初に `.dev.vars` を用意する

`.dev.vars` は gitignore 済み（`.gitignore:4`）で **worktree には複製されない**。無いまま
`vp dev`（`pnpm run test:e2e` の自動起動を含む）を動かすと、`@cloudflare/vite-plugin` が
commit 済みの `worker-configuration.d.ts` を再生成し、`DEEPSEEK_API_KEY` の宣言が消えた差分が
毎回出る。worktree を切ったら実装を始める前に用意する:

```bash
cp .dev.vars.example .dev.vars
```

`.dev.vars.example` が実際に読む鍵をそのまま並べてあるので、コピーすればそのまま動く
（`DEEPSEEK_API_KEY` はダミー、ログインは `demo` / `demo`）。

型の差分は値ではなく鍵の**存在**で決まるので、ダミー値で消える。現在の E2E は DeepSeek へ
問い合わせないので、実キーが要るのは手で回答の生成を確かめるときだけ。そのときはメインクローンの
`.dev.vars` からコピーする。**チャット送信を E2E に足すなら実キーが要る**——ダミー値では認証が
通らず、トークンが 1 つも届かないまま 60 秒のタイムアウトまで粘って落ちる。

**`AUTH_*` が無いと API は全部 401 になる**ので、`.dev.vars` を用意しないと E2E も画面も
何も動かない（上記「ログインとセッション」の「秘密が無ければ閉じる」）。

### `useEffect` の扱い

`vite.config.ts` の `no-restricted-imports` が `useEffect` の import を禁止している。
このアプリは canvas 描画・DOM 購読・pdf.js の命令的 API が本質なので使う場面が多いが、
ルールは**残したまま**、使う側が import 行に
`// oxlint-disable-next-line no-restricted-imports -- <理由>` を付けて理由を明記する運用にしている。
新しく足すときも同じように理由を書くこと。

現在 12 ファイルに理由コメントがあり、内訳は次の 5 つしかない。新しく足す `useEffect` も
このどれかに当てはまるはずで、当てはまらないなら書き方を疑うこと:

| 用途                                                    | ファイル                                                                                                                                                                                                                  |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| pdf.js という命令的ライブラリの呼び出しと後始末         | `PdfPage.tsx`（`RenderTask` / `TextLayer`）、`usePdfDocument.ts`（バイナリ取得とドキュメント構築）、`usePdfOutline.ts`（`getOutline` と dest 解決）、`usePageBaseSize.ts`（`getViewport({scale: 1})` でページの素の寸法） |
| `document` / `window` / `ResizeObserver` の購読         | `useKeyboardShortcuts.ts`、`SettingsMenu.tsx`、`SelectionPopover.tsx`、`PdfViewer.tsx`、`useSettledSelection.ts`（`document` の `selectionchange` と `window` の pointer 系）                                             |
| 非 passive なジェスチャの購読（ブラウザの既定を止める） | `PdfViewer.tsx`（ctrlKey wheel のピンチ、touch と Safari の gesture イベント）                                                                                                                                            |
| DOM への命令的な書き込み（スクロール位置）              | `ChatMessageList.tsx`（最下部へ追随）、`PdfViewer.tsx`（ページ遷移時のリセット）                                                                                                                                          |
| URL とサーバという React の外の状態への同期             | `useReadingLocation.ts`、`useReadingStateSync.ts`（読書位置の保存と離脱時の書き残し）                                                                                                                                     |

**画面幅の購読には `useEffect` を使わない**。`useIsNarrow`（`src/front/hooks/useIsNarrow.ts`）が
`useSyncExternalStore` で `matchMedia` を購読する。購読するのは幅そのものではなく
メディアクエリの真偽なので、再レンダーはレイアウトが切り替わるときだけ起きる。

**データ取得は理由にならない**。一覧・本・ハイライト・引用箇所のページ解決は SWR へ
移してある（下記「状態管理とルーティング」）。

**SWR が持っているものを atom へ写すのも理由にならない**。写した瞬間に同じデータが
2 箇所に載り、更新のたびに 1 レンダー遅れる。読み手が少ないなら props で配る
（`AppPage` → `PdfViewer` / `ChatArea` の `book` がその形）。

これと紛らわしいものが 3 つある。どれも `useReadingLocation.ts` で、SWR が解いた値を
atom に一度だけ書く——`useSWRImmutable` が解いた「引用箇所のページ番号」を
`currentPageAtom` に、`useBook` が返した本の中から URL の `?selection=` が名指した
ハイライトを `activeSelectionAtom` に（`openChat` 経由。下記「リーダーの URL は
`useReadingLocation` が単独で書く」）、同じく `useBook` が返した本の `readingState` を
`currentPageAtom` / `activeSelectionAtom` / `outlineOpenAtom` / `chatPanelOpenAtom` に
（下記「読んでいた場所は本と一緒に運ぶ」）。これは写しではない: どの atom も「読者が今どこを
見ているか」というクライアント状態で、キーボード・ページ送りボタン・目次・URL・一覧の
クリックも書き込む。取得結果はその状態を**一度だけ動かすきっかけ**であって、サーバのデータを
atom に常駐させているわけではない。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skanehira/chatbook](https://github.com/skanehira/chatbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
