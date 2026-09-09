---
trigger: always_on
description: > Coding agent 会自动读取此文件。这是项目的核心知识库入口。
---

# luatex-cn 项目上下文

> Coding agent 会自动读取此文件。这是项目的核心知识库入口。

## 项目概述

**luatex-cn** 是一个用于排版中国古籍的 LuaTeX 包，支持竖排、句读、夹注、批注、眉批、版心等传统古籍排版功能。

- **GitHub**: https://github.com/open-guji/luatex-cn
- **许可证**: Apache 2.0
- **当前版本**: 0.3.8

## 核心指令

1. **说中文** - 与用户交流使用中文
2. **先读文档** - 开始前通读 `ai_must_read/` 文件夹
3. **使用 Context7 MCP** - 需要 API 文档时自动使用

## 必读文档

每次对话开始时，先浏览这些文档：

| 文档 | 内容 | 何时读取 |
|------|------|----------|
| `ai_must_read/LEARNING.md` | 开发经验与教训（Lua/TeX 陷阱、渲染问题） | **必读** - 避免重复踩坑 |
| `ai_must_read/design.md` | 架构设计文档 | 实现新功能前 |
| `ai_must_read/expl3_note.md` | expl3 语法详解（参数展开、xparse陷阱） | **遇到 expl3 问题时必读** |

## expl3 问题速查

遇到以下问题时，**必须先阅读** `ai_must_read/expl3_note.md`：

- 参数展开问题（`\exp_args:N...`、`:n`/`:V`/`:x` 区别）
- xparse 可选参数 `[...]` 传递变量
- Token list 与整数展开差异
- `\lua_now:e` 中的空格处理
- Key-Value 布尔值传递

## 项目结构

```
tex/
├── ltc-guji.cls / ltc-guji-digital.cls / ltc-cn-vbook.cls / ltc-tw-vbook.cls   # 文档类
├── core/           # 核心渲染引擎
│   ├── luatex-cn-core-render-page.lua   # 主渲染逻辑
│   ├── luatex-cn-layout-grid.lua        # 网格布局
│   ├── luatex-cn-core-page.sty          # 页面设置
│   └── luatex-cn-core-sidenote.lua      # 侧批处理
├── guji/           # 古籍功能
│   ├── luatex-cn-guji-judou.sty         # 句读
│   ├── luatex-cn-guji-jiazhu.sty        # 夹注
│   ├── luatex-cn-guji-pizhu.sty         # 批注
│   └── luatex-cn-guji-yinzhang.sty      # 印章
├── shared/         # clreq 共享规则内核（标点表 / 行内调整求解器 / 禁则 / 字面锚点，横竖排共用）
├── hori/           # 横排后端（clreq 横排管道）
├── banxin/         # 版心相关
├── decorate/       # 装饰元素
├── digital/        # 数字化布局模式（ltc-guji-digital）
├── fonts/          # 字体检测
├── util/           # 工具函数
├── configs/        # 模板配置文件
└── debug/          # 调试工具

test/
├── unit_test/        # 单元测试（43 个文件，texlua 运行）
│   ├── util/         # 工具函数测试
│   ├── core/         # 核心渲染引擎测试
│   ├── shared/       # clreq 共享内核测试
│   ├── hori/         # 横排后端测试
│   ├── guji/         # 古籍功能测试
│   ├── decorate/     # 装饰元素测试
│   ├── banxin/       # 版心测试
│   ├── fonts/        # 字体检测测试
│   └── debug/        # 调试模块测试
├── run_all.lua       # 运行全部 unit test
├── test_utils.lua    # 测试框架（mock + assert）
├── digital_test/     # 数字化对照测试（*-digital.tex）
├── regression_test/  # 视觉回归测试（basic / past_issue / complete 三个套件）
│   └── <套件>/
│       ├── tex/      # 测试用 .tex 文件
│       ├── baseline/ # 基准图像
│       └── current/  # 当前输出
└── regression_test.py

AGENTS.md             # 项目指令（本文件，coding agent 自动读取）
CLAUDE.md             # 仅含 @AGENTS.md 导入，供 Claude Code 读取

.claude/
└── skills/           # 可用技能（每个技能一个目录，含 SKILL.md）
```

## 常用命令

### 单元测试（Unit Test）
```bash
# 运行全部 unit test（必须先通过再跑 regression test）
texlua test/run_all.lua

# 运行单个测试文件
texlua test/unit_test/core/layout-grid-test.lua
```

### 回归测试（Regression Test）
```bash
# 首次运行前：下载测试字体 TW-Kai 到 test/fonts/（不入库，不装系统字体）
sh scripts/download_test_fonts.sh

# 使用者日常排版的免安装字体（同一 manifest/脚本）：
#   --user 装入 TEXMFHOME/fonts/truetype/luatex-cn/；--dest DIR 放入文档项目目录
#   文档中 \设置字体族{Jigmo}（注册表别名）、\设置字体族{fonts/MyFont.ttf}（文件路径）
#   或 \setmainfont{TW-Kai-98_1.ttf} 直接引用
python3 scripts/download_fonts.py --all --user

# 运行所有测试
python3 test/regression_test.py check

# 测试单个文件
python3 test/regression_test.py check test/regression_test/basic/tex/guji.tex

# 更新基线（确认改动正确后）
python3 test/regression_test.py save test/regression_test/basic/tex/guji.tex
```

### 几何测试（Geometry Test）
```bash
# 不依赖基线图像的自校验：解析 PDF 内容流，断言每列字形基线等距。
# 像素回归测试只能保证"和上次一样"——若 bug 已存在于基线中则无法发现；
# 此测试能抓「一」「丶」等墨迹不跨基线的字被整字偏移的问题（LEARNING.md 3.6）
python3 test/geometry_test.py
```

### 颜色 key 测试（Color Key Test）
```bash
# 不依赖基线图像的自校验：断言各模块「文字颜色」的两种拼法都真正落到 PDF 上——
# core 层写 font-color / 字体颜色，批注/眉批/侧批/句读写 color / 颜色，两边互为别名；
# 并断言颜色名、0-255 三元组、0-1 三元组三种写法等价，
# 以及未知 key 会发警告而不是被静默丢弃（issue #163 拖延的根因）。
python3 test/color_test.py
```

### clreq 断言测试（横排规范符合性）
```bash
# 解析横排 PDF 内容流（字形 x 坐标 + advance），对 clreq 条款做度量断言：
# 中西间距 ∈ [1/8, 1/2] em、点号旁/夹注号内侧无间距、行首行尾禁则、
# 符号分离禁则（数字串/单位/货币同行零间隙）、两字宽标点、
# 行末标点半字宽（H2 强制断言）、行间注词对齐（H4）等。
# 每条断言注明 clreq 条款。
python3 test/clreq_test.py                 # 编译并检查 basic/hori.tex
python3 test/clreq_test.py yourfile.tex    # 检查其他横排文档
```

回归测试要点：

- **测试字体**：测试 .tex 硬编码 TW-Kai 字体。`regression_test.py` 会自动把
  `OSFONTDIR` 指向 `test/fonts/`，字体由 `scripts/download_test_fonts.sh`
  下载（固定 mirror commit，可复现），无需安装到系统字体目录。
- **JSON 基线**：layout JSON 与基线比较时忽略 `source_mtime` 等易变元数据
  （git 检出会改变 mtime，与布局无关）。
- **CI**：`.github/workflows/test.yml` 会对每个 PR 及 main/dev 的 push
  先跑 unit test 再跑 regression test，本地通过后 CI 应当同样通过。

### 示例重建（Examples Rebuild）
```bash
# 渲染算法一改，示例/ 下的 PDF 与预览图就全部过期。清单驱动一键重建：
python3 scripts/build/build_examples.py            # 重建 示例/ 全部 PDF + PNG
python3 scripts/build/build_examples.py --check    # 只报告哪些页变了，不写文件
python3 scripts/build/build_examples.py --only 红楼梦
python3 scripts/build/build_examples.py --all      # 含 全书复刻/（92 页，慢）
python3 scripts/build/build_examples.py --list     # 看清单：tex → PDF → 第几页出哪张图
```
- 清单在脚本顶部的 `DOCS`：新增示例时在这里加一条，不要手工导图。
- 素材与底本扫描（`文渊阁宝印.png`、`ref-*.png`、`page_*.jpg`）永不重写。
- 预览图统一 150 dpi；字体同回归测试经 `OSFONTDIR` 指向 `test/fonts/`。

### 编译测试
```bash
# 在对应套件的 tex 目录下编译
cd test/regression_test/basic/tex && lualatex guji.tex

# 带调试输出
lualatex -interaction=nonstopmode guji.tex 2>&1 | grep -E "\[DEBUG\]|\[ERROR\]"

# 导出 layout JSON（用于 converter 验证）
ENABLE_EXPORT=1 lualatex yourfile.tex
# 生成 yourfile-layout.json
```

### Git 操作
```bash
# 查看最近提交

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-guji/luatex-cn](https://github.com/open-guji/luatex-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
