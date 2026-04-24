---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vimprove 是一个交互式 Vim 学习网站。核心功能是通过浏览器中的"迷你 Vim 编辑器 + 关卡式练习"来教用户实际操作 Vim 命令。

**当前状态**: ✅ 重构完成。项目已从单文件原型（`tmp/vimprove.html`）重构为模块化的 React + TypeScript 架构。

**课程范围**: 已完成 Chapter 1-6（基础、进阶编辑、行内 find/till、文本对象、搜索/重构），共 28 节课。

**版本管理**: 版本号在 `src/version.ts` 和 `package.json` 中维护，CHANGELOG 见 `README.md`

## Development Commands

```bash
npm run dev      # Start dev server at http://localhost:3000
npm run build    # Build for production
npm run preview  # Preview production build
npm run lint     # Run ESLint
npx vitest run --pool=threads # deprecated
# Note: please refer to section "Test Workflow" and use ./utils/vitest-quickcheck.sh to improve efficiency.
```

## Test Workflow

- 默认并行：`npx vitest run --pool=threads`（避免直接跑无过滤的 `npx vitest run`，输出过长会淹没上下文）
- 测试输出简化： **重要** 使用`grep`过滤最终结论，避免无用信息淹没上下文。如 `npx vitest run --pool=threads [-t <pattern>] 2>&1 | grep -EA20 "Failed Tests|Test Files"`
- 测试输出长度：`grep -EA20`是使用的20行，所获信息不足时，可以根据实际情况适当增加，建议不超过50行
- Parity 单测输出节流：`npx vitest run --pool=threads -t "<pattern>" src/core/tests/exhaustiveTest.*.test.ts 2>&1 | grep -EA20 "Failed Tests|Test Files"`，避免海量 skip 日志淹没上下文
- exhaustive parity 分片：`src/core/tests/exhaustiveTest.{0..7}.test.ts` 使用 `getShardCases` 按索引取模分片（总 shard=8），并行示例：`npx vitest run --pool=threads src/core/tests/exhaustiveTest.*.test.ts --maxWorkers=8`
- 快速检查脚本：`bash utils/vitest-quickcheck.sh [<test_glob>]`（tap-flat + bail，默认跑全部，可传入路径/模式，成功输出 ok ✅，失败时列出前 5 条 not ok）
- 深入排查（exhaustive parity）：
  - 生成 JSON 报告：`npx vitest run --pool=threads --reporter=json --outputFile tmp/vimParity-report.json src/core/tests/exhaustiveTest.*.test.ts`
  - 查看摘要/聚合或按子串过滤：`python utils/vimParity-report-viewer.py tmp/vimParity-report.json ["keyword"...]`（keyword 为测试名片段，支持多个并且大小写不敏感；工具仅用于 ParityExhaustive）
    - 支持附加参数：`--feature paste-after-op`、`--limit 5`、`--sort name|feature|line`、`--details`（输出完整断言）
  - 反复调试单用例：`npx vitest run --pool=threads -t "<pattern>" src/core/tests/exhaustiveTest.*.test.ts 2>&1 | grep -EA20 "Failed Tests|Test Files"`
    - 正则含特殊符号时建议单引号包裹并在内部转义，如 `-t 'd\$P\.'`

- ### Debug Tips & Tools

- 生成组合差异时优先跑 `bash utils/vitest-quickcheck.sh`（Tap + bail）或 `npx vitest run --pool=threads -t "<case>" 2>&1 | grep -EA20 "Failed Tests|Test Files"`，务必加上过滤，避免全量输出淹没上下文。
- 若 Neovim 输出夹杂错误日志，可用 `utils/nvim-state-probe.cjs` 直接打印 stdout/stderr 以及解析后的 state：`node utils/nvim-state-probe.cjs --lines '["foo bar"]' --cursor 1,5 --keys 'p' --debug`；支持 `--file` 读取文本文件，`--cursor` 为 1-based。
- Parity 排查时，先用 probe 得到 Neovim 游标/模式，再用 `runSimKeys` 对比，减少反复跑大测试集。
- `utils/vimprove-debug.cjs`：输入 shard 与 label（用例名）对比模拟器与 Neovim 的缓冲区/光标/模式，并可加 `--trace` 打印逐键状态（需 `node -r sucrase/register utils/vimprove-debug.cjs <label> <shard> [--trace]`）。
- 定位顺序：1) quickcheck 抽样锁定 case；2) probe 取 Neovim 真实游标/模式；3) `runSimKeys` 对比；避免直接跑全量。

## Architecture

### Core Design Principles

1. **配置驱动的课程系统**: 新增课程只需添加配置文件，无需修改核心代码
2. **纯函数的 Vim 引擎**: 所有状态更新通过 reducer，便于测试
3. **严格的模块分离**: Core/Data/Hooks/Components 各层职责单一

### Module Structure

```
src/
├── core/              # Vim 引擎核心（纯逻辑，零依赖）
│   ├── types.ts      # 所有类型定义（Buffer, Cursor, VimState, Command, ChallengeGoal 等）
│   │                 # Lesson/ContentBlock/KeyItem 支持可选 i18nKey 字段
│   ├── vimReducer.ts # 核心状态管理 reducer
│   ├── motions.ts    # 移动逻辑（h/j/k/l/w/b/0/$/f/t 等）
│   ├── operators.ts  # 操作符逻辑（d/c/y + motion）
│   └── utils.ts      # 工具函数（clampCursor, isWhitespace）
│
├── data/              # 课程数据（只依赖 core/types）
│   ├── categories.ts # 课程分类定义
│   └── lessons/      # 每个课程一个文件（按章节组织）
│       ├── chapter1/ # 模式与基础移动（5课）
│       ├── chapter2/ # 单词移动与小编辑（5课）
│       ├── chapter3/ # 高级编辑（5课）
│       ├── chapter4/ # 行内 find/till 精准编辑（4课）
│       ├── chapter5/ # 文本对象（5课）
│       └── chapter6/ # 搜索与重构（4课）
│
├── hooks/             # 自定义 hooks（业务逻辑封装）
│   ├── useVimEngine.ts    # 封装 vimReducer
│   ├── useChallenge.ts    # 挑战逻辑（目标验证、计时）
│   ├── useProgress.ts     # 进度持久化（localStorage）
│   ├── useHjklSnakeStats.ts # hjkl 贪吃蛇本地记录
│   └── useI18n.ts         # i18n hooks（useTranslationSafe, useLocale）
│
├── components/        # UI 组件
│   ├── common/       # 通用组件（MarkdownBlock, KeyListBlock）
│   ├── lesson/       # 课程相关（LessonView, LessonNav）
│   ├── challenge/    # 挑战相关（VimChallenge 内置编辑器 + GoalsList）
│   ├── minigame/     # 小游戏组件（HJKL Snake）
│   └── layout/       # 布局组件（Sidebar, Header）
│
├── pages/            # 页面组件
│   ├── HomePage.tsx
│   └── LessonPage.tsx
│
└── i18n/             # 国际化支持
    ├── config.ts     # i18n 配置与初始化（使用 Vite glob 动态加载）
    ├── index.ts      # 导出配置与类型
    └── locales/      # 语言包（按语言/命名空间组织）
        ├── en/       # 英文
        │   ├── common.json    # 通用词汇
        │   ├── layout.json    # 布局组件
        │   ├── home.json      # 主页
        │   ├── lesson.json    # 课程页面
        │   ├── challenge.json # 挑战组件
        │   ├── example.json   # Run Example 组件
        │   ├── settings.json  # 设置页面
        │   └── lessons.json   # 课程内容翻译（按 slug 组织）
        └── zh/       # 中文（结构同 en）
```

### i18n Architecture

**技术栈**: `i18next` + `react-i18next` + `i18next-browser-languagedetector`

**配置位置**: `src/i18n/config.ts`
- 使用 Vite 的 `import.meta.glob` 动态加载所有语言包（eager 模式）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jerry-Terrasse/vimprove](https://github.com/Jerry-Terrasse/vimprove) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
