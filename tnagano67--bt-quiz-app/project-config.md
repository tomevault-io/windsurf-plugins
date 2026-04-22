---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

BT管理システム — 教育機関向け小テスト・成績管理システム。元は GAS + Google スプレッドシートで構築されていたものを、モダンスタックで再構築。学習・練習目的。複数科目対応で、生徒機能（科目選択・小テスト受験・履歴・成績チャート）と教員機能（科目/生徒/問題/グレード定義のCRUD・成績ダッシュボード・CSVインポート/エクスポート）を実装済み。

実装フェーズの履歴・DBスキーマ(SQL)・今後の候補は `PLAN.md` を参照。

## 技術スタック

- **Next.js 16** App Router、`src/` ディレクトリ構成、TypeScript（strict mode）
- **React 19**
- **Tailwind CSS 4**（`@import "tailwindcss"` 構文、`@theme inline` でカスタムプロパティ定義 — `tailwind.config` ファイルなし）
- **Supabase**（`@supabase/ssr` でSSR対応認証、`@supabase/supabase-js` でDB操作）
- **Chart.js** + react-chartjs-2（成績チャート、`next/dynamic` で遅延読み込み）
- **date-fns** + date-fns-tz（Asia/Tokyo タイムゾーン対応）
- **ESLint 9** flat config（`eslint.config.mjs`、`eslint-config-next/core-web-vitals` + `/typescript`）
- **Vitest** でユニットテスト
- **Playwright** でE2Eテスト
- **Vercel** でデプロイ

## コマンド

```bash
npm run dev          # 開発サーバー起動 (localhost:3000)
npm run build        # 本番ビルド（Turbopack）
npm run start        # 本番サーバー起動
npm run lint         # ESLint 実行
npm run test         # Vitest テスト一度実行
npm run test:watch   # Vitest ウォッチモード
npx vitest run src/lib/csv-utils.test.ts  # 単一テストファイル実行
npm run test:e2e     # Playwright E2Eテスト実行
npm run test:e2e:ui  # Playwright UIモード（ブラウザ付きデバッグ）
npm run test:e2e:headed  # ブラウザ表示付きで実行
```

ユニットテストファイルはソースファイルの隣に `*.test.ts`（または `*.test.tsx`）で配置（コロケーション方式）。テスト用モックユーティリティは `src/test-utils/` に配置。E2Eテストは `e2e/` ディレクトリに配置。

## アーキテクチャ

### ルーティング（App Router）

- `/` — ロール判定: 教員→`/teacher`、生徒→`/student`、未登録→メッセージ表示、未認証→`/login`
- `/login` — Google OAuth ログイン画面
- `/auth/callback` — OAuth コールバック（Route Handler）→ `/` へリダイレクトしてロール判定
- `/student` — 生徒ホーム（科目カードグリッド表示・各科目の進捗）
- `/student/quiz` — 小テスト（Client Component、`Suspense` ラッパー、`?subject={subjectId}` 必須）
- `/student/history` — 学習履歴・再受験
- `/teacher` — 教員ダッシュボード（概要統計・グレード分布チャート・合格率推移チャート・科目フィルタ）
- `/teacher/students` — 生徒一覧（URLパラメータでフィルター・科目フィルタ）、CSVインポート
- `/teacher/students/new` — 生徒登録
- `/teacher/students/[studentId]` — 生徒詳細（30日統計・チャート）
- `/teacher/students/[studentId]/edit` — 生徒編集
- `/teacher/subjects` — 科目一覧・登録・編集・削除
- `/teacher/subjects/new` — 科目登録
- `/teacher/subjects/[subjectId]/edit` — 科目編集
- `/teacher/questions` — 問題一覧（科目フィルタ）、CSVインポート
- `/teacher/questions/new` — 問題登録（科目選択）
- `/teacher/questions/[questionId]/edit` — 問題編集
- `/teacher/grades` — グレード定義一覧（科目フィルタ）
- `/teacher/grades/new` — グレード定義登録（科目選択）
- `/teacher/grades/[gradeId]/edit` — グレード定義編集
- `/teacher/teachers` — 教員一覧、CSVインポート
- `/teacher/teachers/new` — 教員登録
- `/teacher/teachers/[teacherId]/edit` — 教員編集
- `/teacher/export` — 成績CSVエクスポート（Client Component、フィルター・件数確認・ダウンロード）
- `/api/teacher/export` — CSVダウンロード用 Route Handler（BOM付きUTF-8）

各ルートに `loading.tsx`（スケルトンUI）と `error.tsx`（リトライボタン付き、`useEffect` で `error.digest` をコンソールにログ出力）を配置済み。

### ロール分離とカラーテーマ

- **生徒側**: 青系（`blue-600` 等）、`max-w-3xl` コンテナ
- **教員側**: ティール系（`teal-700` 等）、`max-w-5xl` コンテナ（テーブル用に広め）
- 各ロールは独立した `layout.tsx` と専用ヘッダー（`Header.tsx` / `TeacherHeader.tsx`）を持つ
- **ヘッダーナビゲーション**: アクティブタブに下ボーダー表示（生徒: `border-blue-600`、教員: `border-teal-600`）。`TeacherHeader` はモバイル（`md` 未満）でハンバーガーメニュー、デスクトップでタブ表示
- **モバイルテーブル対応**: `StudentTable` は日付スコア列を `hidden lg:table-cell` でデスクトップのみ表示。`QuestionTable` は選択肢1〜4列を `hidden md:table-cell` でデスクトップのみ表示

### Server Component / Client Component の使い分け

- **Server Component**: ホーム画面、履歴、教員一覧/詳細ページ、テーブル系コンポーネント（`StudentTable`、`QuestionTable`、`GradeTable`、`SubjectTable`）、表示系コンポーネント（`SubjectCard`、`QuizQuestion`、`QuizResult`、`HistoryItem`、`StatisticsCard`）
- **Client Component** (`"use client"`): クイズページ、ヘッダー（`usePathname`）、`ScoreChart`/`GradeDistributionChart`/`PassRateTrendChart`（Chart.js、各ページから `next/dynamic` で遅延読み込み、`useMemo` で `data`/`options` メモ化）、`StudentFilter`（URL パラメータ操作）、フォーム系（`QuestionForm`/`StudentForm`/`GradeForm`/`SubjectForm`/`TeacherForm`）、CSVインポート系（`CsvImport`/`StudentCsvImport`）、`Pagination`、`ConfirmDialog`（削除確認モーダル）、各 `*DeleteButton`（5種）、ログイン画面、エクスポートページ

### 主要な lib モジュール

- `src/lib/supabase/` — Supabase クライアント: `client.ts`（ブラウザ用）、`server.ts`（SSR用・`cookies()` 使用）、`middleware.ts`（セッション更新・認証ガード）
- `src/lib/types/database.ts` — DB テーブルの型: `Subject`, `Student`, `StudentSubjectProgress`, `Teacher`, `GradeDefinition`, `Question`, `QuizRecord`
- `src/lib/quiz-logic.ts` — `shuffleArray`（Fisher-Yates）、`shuffleChoices`、`gradeQuiz`、`verifyScore`（サーバー側再採点）
- `src/lib/grade-logic.ts` — `calculateGradeAdvancement`（連続日数・昇級判定）
- `src/lib/date-utils.ts` — `getTodayJST`、`getRecentDates`、`isTakenToday` 等（Asia/Tokyo）
- `src/lib/csv-utils.ts` — `parseCsvRows`（RFC 4180 準拠CSVパーサー、クォート内改行対応）、`generateCsvText`（2D配列→CSV文字列変換）
- `src/lib/validation.ts` — `validateQuestionInput`、`validateStudentInput`、`validateTeacherInput`、`validateSubjectInput`（CSVインポート等の入力値検証を一元化）
- `src/lib/export-utils.ts` — `getGradeFilter`（Map O(1) ルックアップ）、`calculateStudentStats`、`formatStudentExportRow`、`formatRecordExportRow`（CSVエクスポートのロジック）
- `middleware.ts`（ルート） — `/student/*` と `/teacher/*` ルートの認証ガード

### Server Actions

- `src/app/student/quiz/actions.ts` — 小テスト結果保存（認証・本人確認・日次制限・サーバー側再採点・進級計算・DB保存）。進捗は `student_subject_progress` テーブルで科目別に管理
- `src/app/teacher/subjects/actions.ts` — 科目のCRUD。作成時に全生徒分の `student_subject_progress` を自動作成（1000件バッチ）。削除時に参照チェック

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tnagano67) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
