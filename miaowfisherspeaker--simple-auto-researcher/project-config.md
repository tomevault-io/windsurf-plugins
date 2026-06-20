---
trigger: always_on
description: 自动化学术研究助手 - 搜索→下载→阅读→归档→创新点生成验证
---


# AutoResearcher - 自动化学术研究助手

## 📖 变量引用规范

在执行任务时，需要从配置文件中读取动态变量。语法如下：

- "工作量" - 引用变量，值由配置文件决定（如 PREFERENCES.md 中定义的工作量）
- `"工作量"` - 强调变量名本身，用于需要显式说明"此处为变量"的语境
- 两者本质相同，均指向同一变量，唯一的区别是显示效果不同

**示例：**

| 写法                      | 含义       | 实际效果（假设"工作量"=5） |
| ------------------------- | ---------- | -------------------------- |
| "工作量" 篇               | 引用变量   | 5 篇                       |
| `"工作量"`                | 强调变量名 | "工作量",即5               |
| 下载 "工作量" 篇论文      | 引用变量   | 下载 5 篇论文              |
| 当前工作负载为 `"工作量"` | 强调变量名 | 当前工作负载为 5           |

**说明：**

- 使用全角引号 "" 表示引用变量值
- 使用单反引号 + 全角引号 \`""\` 表示显式标注变量名
- 普通文字不带引号，表示固定说明内容

## 目录设置

【⚠️重要】

- 阅读 `PATHS.md` 了解 "AutoResearcher项目根目录"、"小龙虾目录"等。
- 阅读 `DIRECTIONS.md` 了解用户的"研究方向"（即科研方向）等。
- 阅读 `PREFERENCES.md` 了解用户的"联网搜索偏好"、"偏好会议或者期刊"、以及最重要的期望`"工作量"`等。
- 阅读 `GUIDELINES.md` 了解rethink.md综述的写作规范和要求、创新点模板要求、papers.csv的要求

## ⚠️ 重要：工作目录

**所有任务必须在 `"AutoResearcher项目根目录"` 目录下执行！**

定时任务（isolated 模式）默认工作目录是 `~/.openclaw/workspace/`，因此任务消息开头必须包含：

```
【重要】请先执行：
 cd "AutoResearcher项目根目录"
 然后继续以下任务：
```

## 📁 工作目录应该有的项目结构

```
AutoResearcher/                  # "AutoResearcher项目根目录"
├── papers.csv                    # 论文总库（所有论文元信息汇总）
├── README.md                    # 项目说明
├── files/
│   ├── downloaded/*.pdf         # 已下载论文 PDF
│   └── tobedownloaded/YYYY-MM-DD.md # 下载失败论文记录（MD格式）
├── notes/
│   ├── YYYY-MM-DD/*.md          # 论文阅读笔记（按日期分离）
├── knowledgeOutput/
│   ├── daily/                   # 日归档（同日累计，异日分离）
│       └── daily_YYYY-MM-DD.md
│   ├── weekly/                  # 周归档（同月累计，异月分离）
│       └── weekly_YYYY-MM.md
│   ├── rethink.md               # 知识体系构建（上限10万字）
│   └── bak/                     # 备份（最多5份）
├── idea/
│   ├── all/                     # 待验证创新点
│   ├── viewer/                  # 验证记录
│   ├── deprecated/              # 废弃创新点
│   └── idea_*.md                # 已验证创新点
├── logs/
│   ├── search/                  # 搜索记录（同日累计，异日分离）
│       └── search_YYYY-MM-DD.md
│   ├── download/                # 下载记录
│       └── download_YYYY-MM-DD.md
│   └── read/                    # 已读论文列表
│       └── read_YYYY-MM-DD.md
├── trash/*.md                   # 清理后不符合笔记要求的文档
├── projects/
│   ├── {project_name}_{idea_id}/ #可以运行的idea的demo完整框架
│   ├── {project_name}_{idea_id}.ipynb #可以在colab上运行的jupyter notebook
│   └── datasets/                # 下载的数据集，不允许过大

```

**⚠️注意**：不允许在每个子目录出现命名不正确的文件，如果需要临时存储，均放到`trash/`中

## ⏰ 定时任务配置

### 核心任务

| 任务 ID | 任务名称              | 间隔/时间         | 每次工作量                    |
| ------- | --------------------- | ----------------- | ----------------------------- |
| 搜索    | AutoResearcher-搜索   | 每 12 小时        | `"工作量"`\*4\*10 篇          |
| 下载    | AutoResearcher-下载   | 每 30 分钟        | `"工作量"`\*2 篇              |
| 阅读    | AutoResearcher-阅读   | 每 15 分钟        | `"工作量"` 篇                 |
| 日归档  | AutoResearcher-日归档 | 每 1 小时         | 根据未归档的文章数决定        |
| 创新点  | AutoResearcher-创新点 | 每日10：00和22:00 | 每次至多`"创新点数"`个 创新点 |

### 周期性任务/其他任务

| 任务       | 任务名称                  | 频率         | 说明                                                                                                                                                      |
| ---------- | ------------------------- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Rethink    | AutoResearcher-Rethink    | 每 3 小时    | 知识体系重构，上限 10 万字                                                                                                                                |
| 备份       | AutoResearcher-备份       | 每日 03:00   | 备份 daily/ + weekly/ + rethink.md，最多 5 份                                                                                                             |
| 周归档     | AutoResearcher-周归档     | 每周日 21:00 | 生成周小结                                                                                                                                                |
| 创新点审查 | AutoResearcher-创新点审查 | 每天22:00    | 对已有的创新点再一次进行审查                                                                                                                              |
| 清理       | AutoResearcher-清理       | 每12小时     | 清理.openclaw目录、已经阅读并归档的PDF；清理notes目录下的不是论文阅读笔记的笔记，即标题显然不是论文标题的markdown，如果进行了清理，立马进行一次papers整理 |
| papers整理 | AutoResearcher-papers整理 | 每12小时     | 对整理的papers.csv进行整理，确保各个位置的论文及其数量一一对应                                                                                            |

## 📋 任务详细说明

### 🔍 搜索任务

**触发条件**：每 12 小时

**工作目录**：`"AutoResearcher项目根目录"`

**执行步骤**：

1. 【待读检查】
   - 读取 `papers.csv`，统计"未阅读"数量
   - 如果未读 ≥ `"工作量"`\*4\*12 → 记录日志，跳出，说明当前论文够12小时内阅读

2. 【搜索 + 查重】（最多循环3次）
   - （根据用户"偏好会议或者期刊"以及用户"论文要求"偏好的"论文时间年限"进行搜索）搜索 `"工作量"`\*4\*10篇新论文。
   - 对每篇论文：
     - 检查 papers.csv 是否已存在/下载/归档（已存在/下载/归档 → 忽略不存入）
   - 如果不够，则进行补充。

3. 【搜集论文信息】
   - 通过dblp，搜集"papers.csv要求"所需要的条目，可以留空（除了标题、链接）
   - 新的论文状态（下载）、状态（阅读）、状态（归档）分别标记为未下载、未阅读、未归档

4. 【记录 + 更新】
   - 写入 `logs/search/search_YYYY-MM-DD.md`，做好日志。
   - 增量更新 `papers.csv`

**输出文件**：

- `logs/search/search_YYYY-MM-DD.md`
- `papers.csv`

---

### 📥 下载任务

**触发条件**：每 30 分钟

**工作目录**：`"AutoResearcher项目根目录"`

**执行步骤**：

1. 【待下载搜集】读取 `paper.csv` 中标注为"待下载"的论文
2. 【按序下载】按顺序选择 **`"工作量"*2` 篇**下载 PDF
3. 【**清理 PDF 文件名**】：下载后立即去掉序号前缀（如 `01_`、`02_`、`1_`、`2_`、`03_` 等）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MiaowFisherSpeaker/simple-auto-researcher](https://github.com/MiaowFisherSpeaker/simple-auto-researcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
