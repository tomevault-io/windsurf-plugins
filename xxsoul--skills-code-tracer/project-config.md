---
trigger: always_on
description: Track code changes' impact on business scope through LSP call chain tracing. Use this skill when users ask about code impact analysis, business scope affected by code changes, call chain tracing, "what does this function affect", "which APIs call this code", or need impact reports with git commit references. **IMPORTANT:** Also use this skill when users say "我要提测", "我要投产", "准备提测", "准备投产", "提测分析", "投产分析", or any similar phrases about testing/deploying their current branch. Supports any language w
---


# Code Impact Tracker

追踪代码变更对业务的影响范围，通过 LSP 向上回溯调用链，最终生成业务影响报告。

## 子命令

### 提测 / 投产

当用户说 "我要提测"、"我要投产"、"准备提测"、"准备投产" 等类似表达时，执行以下流程：

**Step 1: 推导主干分支**

自动检测项目的主干分支：

```bash
# 方法 1: 检查 origin/HEAD 指向
git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'

# 方法 2: 直接检查常见主干分支名
git branch -a | grep -E 'main|master'
```

推导规则：
1. 如果 `origin/HEAD` 指向某个分支，优先使用该分支
2. 如果没有 `origin/HEAD`，检查是否存在 `master` 或 `main`
3. 默认推荐 `master`（国内项目常见）

**Step 2: 用户确认主干分支**

使用 AskUserQuestion 让用户确认：

```
检测到当前分支: <当前分支名>
推荐的主干分支: <推导出的主干分支>

请确认要对比的主干分支：
1. 使用推荐的 <主干分支>
2. 使用 main 分支
3. 使用 master 分支
4. 手动输入其他分支名、tag 或 commit ID
```

**Step 3: 执行分支对比分析**

确认后，执行分支对比分析（使用 1.2 的流程）：

```bash
git diff <主干分支>..<当前分支> --stat
git diff <主干分支>..<当前分支> --name-only
```

**Step 4: 生成报告**

报告文件名格式：
- 提测: `提测影响报告_<当前分支>_YYYYMMDD.md`
- 投产: `投产影响报告_<当前分支>_YYYYMMDD.md`

报告额外章节：

**提测报告增加**:
```markdown
## 提测检查清单

- [ ] 核心功能已自测通过
- [ ] 边界条件已覆盖测试
- [ ] 新增配置项已同步到测试环境
- [ ] 依赖的外部服务已确认可用

## 测试建议

| 测试类型 | 覆盖范围 | 优先级 |
|---------|---------|--------|
| 功能测试 | <具体功能> | P0 |
| 回归测试 | <受影响的 API> | P1 |
```

**投产报告增加**:
```markdown
## 上线检查清单

- [ ] 测试环境验证通过
- [ ] 配置文件已同步到生产环境
- [ ] 相关人员已通知
- [ ] 回滚方案已准备

## 上线风险

| 风险项 | 影响范围 | 应对措施 |
|-------|---------|---------|
| <风险描述> | <影响模块> | <应对方案> |

## 需要通知的人员

- 运维团队: <具体通知内容>
- 开发团队: <具体通知内容>
```

---

## 核心流程

```
输入起点 → LSP 回溯调用链 → 识别业务入口 → 汇总影响范围 → 生成报告
```

---

## Step 1: 解析输入起点

用户可通过三种方式指定分析起点：

### 1.1 位置格式 (文件：行号)

```
文件路径：行号 [: 列号]
例如：websrv/public_websrv/userinterface/lands.go:52
```

解析步骤：
1. 确认文件存在
2. 确认行号有效（不超过文件行数）
3. 定位到该位置的符号（函数/方法/变量）

### 1.2 Git 提交编号 / 分支对比

```
提交哈希 (完整或简短)
例如：09ac4b9f8 或 09ac4b9

分支对比格式:
git diff <base>..<branch>
例如：git diff master..online/online_260423
```

解析步骤：

**Step 1: 获取变更文件列表**
```bash
git diff <base>..<branch> --stat
git diff <base>..<branch> --name-only
```

**Step 2: 识别变更类型**
对每个变更文件，判断变更类型：
| 变更类型 | 判断方式 | LSP 处理策略 |
|---------|---------|-------------|
| 新增文件 | `git status` 显示 A | 无需回溯 (无调用方) |
| 删除文件 | `git status` 显示 D | 需查找谁调用了被删除的代码 |
| 修改文件 | `git status` 显示 M | **需要 LSP 回溯** |
| 重命名文件 | `git status` 显示 R | 需确认调用点是否受影响 |

**Step 3: 提取变更的函数/方法**
对每个修改的文件：
```bash
git diff <base>..<branch> -- <file>
```
解析 diff 输出，提取：
- 函数定义行变更（`func xxx` 行附近）
- 函数体变更（行号范围）
- 如果是结构体/接口变更，记录受影响的方法

**Step 4: LSP 回溯调用链（核心步骤）**

对每个变更的函数，执行 LSP 回溯：

```
函数变更起点
    ↓ documentSymbol 精确定位函数名
    ↓ prepareCallHierarchy 获取调用层级项
    ↓ incomingCalls 获取第 1 层调用方
    ↓ 对每个调用方重复 incomingCalls
    ↓ 回溯直到业务入口点 (API handler / 配置驱动)
```

**Step 5: 汇总影响范围**

构建影响树：
```
变更文件 1
├── 变更函数 A
│   ├── 影响 API-1: /path/to/api
│   ├── 影响 API-2: /path/to/api
│   └── 影响定时任务：cron_job_xxx
├── 变更函数 B
│   └── ...
└── 变更结构体 C
    └── 影响所有引用该结构体的函数

变更文件 2
└── ...
```

**Step 6: 生成报告**

报告需包含：
1. 变更文件清单
2. 每个变更函数的调用链
3. 受影响的 API 和业务入口
4. 建议的测试覆盖范围

**Step 7: 变更较多时的处理**

如果变更文件超过 10 个或函数超过 20 个：
1. 按模块/目录分组
2. 优先分析核心模块
3. 询问用户是否全部分析或选择特定文件/模块
4. 使用表格汇总，避免报告过长

### 1.3 函数名搜索

```
函数名或方法名
例如：ProjectGroupAdd 或 project_group.GroupAdd
```

解析步骤：
1. 使用 Grep 搜索函数定义位置
2. 若找到多个定义，列出位置供用户选择
3. 确认唯一定义位置后作为起点

---

## Step 2: LSP 环境检测与启动等待

在开始回溯前，**必须**检测 LSP 是否可用，并确保 LSP 已完全启动。

### 2.1 检测方法

根据文件类型判断需要的 LSP：

| 语言 | LSP 服务器 | 检测方式 |
|------|-----------|---------|
| Go | gopls | 检查 `gopls version` |
| Python | pylsp/pyright | 检查 `pylsp --version` 或配置 |
| Java | jdtls | 检查 jdtls 进程 |
| TypeScript/JS | typescript-language-server | 检查 `tsserver` |
| Rust | rust-analyzer | 检查进程或配置 |
| C/C++ | clangd | 检查 `clangd --version` |

### 2.2 LSP 启动等待机制

LSP 服务器启动需要时间，直接调用可能返回 "server is starting" 错误。

**等待流程**：

```
Step 1: 发送 documentSymbol 触发 LSP 启动
    ↓ 如果成功，LSP 已就绪
    ↓ 如果返回 "server is starting" 错误

Step 2: 等待 3-5 秒
    ↓ 重试 documentSymbol

Step 3: 如果仍失败，等待并重试（最多 3 次）
    ↓ 如果 3 次后仍失败，使用 Grep 备选方案
```

**判断 LSP 是否就绪**：
- 成功返回 documentSymbol 结果 → LSP 就绪
- 返回 "server is starting" 或类似错误 → LSP 未就绪，需要等待
- 返回其他错误 → 可能是文件无法解析，需进一步诊断

### 2.3 LSP 不可用时的处理

如果 LSP 不可用：
1. **明确告知用户**：当前环境没有配置该语言的 LSP 或 LSP 启动失败
2. **提供安装指引**：列出常见 LSP 安装方法
3. **询问是否继续**：是否使用备选方案（grep 搜索调用）

备选方案（无 LSP 时）：
- 使用 Grep 搜索函数名调用点
- 精度较低，但可提供基本分析

---

## Step 3: LSP 调用链回溯

### 3.0 位置定位优化

`prepareCallHierarchy` 要求光标定位在函数名上，定位到参数位置会报错 "xxx is not a function"。

**问题场景**：
- 用户指定 `file.go:40` 但未指定列号
- 第 40 行是 `func FeatureSchedule(ctx *fasthttp.RequestCtx)`
- 默认列号 1 定位到 `func` 关键字
- 列号 18 定位到参数 `ctx`，会报错 "ctx is not a function"

**解决方案 - 使用 documentSymbol 精确定位**：

```
Step 1: 对目标文件调用 documentSymbol
    ↓ 获取文件中所有符号的位置信息

Step 2: 根据目标行号匹配对应符号
    ↓ 例如：documentSymbol 返回 "FeatureSchedule (Function) - Line 40"

Step 3: 读取该行内容，计算函数名起始列号
    ↓ 行内容："func FeatureSchedule(ctx *fasthttp.RequestCtx) {"
    ↓ 函数名 "FeatureSchedule" 起始于第 6 列 (func + 空格 = 5 个字符)

Step 4: 使用精确列号调用 prepareCallHierarchy
    ↓ character = 6 (指向函数名第一个字符)
```

**列号计算规则**：

| 语言 | 函数定义格式 | 函数名列号计算 |
|------|-------------|---------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xxsoul/skills-code-tracer](https://github.com/xxsoul/skills-code-tracer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
