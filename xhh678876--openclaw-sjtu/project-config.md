---
trigger: always_on
description: |
---


# 上海交通大学全能校园助手 (sjtu)

> **平台无关。** 本 skill 是 `scripts/` 下的一组独立 CLI 脚本（Python3 / Node.js）。任何能运行
> shell 命令、读文件的 agent 都能调用——不依赖任何特定网关、浏览器代理或私有运行时。
> 凭证集中在仓库根 `config.json`（已被 `.gitignore` 忽略，不会进版本库）。

---

## 一、给 Agent 的调用约定（必读）

1. **执行前先 `cd ~/openclaw-sjtu`**。脚本按仓库根定位 `config.json` / `templates/` / `fonts/`，
   不在此目录会找不到配置或资源。
2. **调用形式**：Python 脚本用 `python3 scripts/<名>.py <子命令> [参数]`；
   `.mjs` 脚本用 `node scripts/<名>.mjs <子命令>`（**水源用 node，不是 python3**）。
   不带参数一般打印用法。
3. **凭证**：统一放仓库根 `config.json`（从 `config.example.json` 复制）。**按需填**——只填要用的
   功能对应字段即可。部分脚本也支持等价环境变量（见 §2）。
4. **写操作必须先确认**：会改服务器/外部状态的命令（发邮件、传承下载消耗积分等），执行前必须向
   用户复述对象并取得确认。本文档用 ⚠️ 标出。注：Canvas **提交作业当前未暴露为 CLI 子命令**
   （仅库函数 `submit_assignment`），不会被误触发。
5. **缺凭证 = 明确报错**：脚本不会假装成功。缺 key/token 会打印「缺什么、怎么配」，多数带 `selftest`。
6. **状态语义**：✅实时=真实抓线上数据；📌静态=内置离线数据（稳定但非实时，跨学期可能过期）；
   🔑需登录=缺凭证会明确报错；⚙️本地=纯本地生成；⚠️注意=有依赖缺失或接口变更，见说明。
7. **环境提示**：macOS **无 `timeout` 命令**，文档示例都不用 timeout，脚本自带超时，直接运行即可。

---

## 二、配置 (config.json)

从 `config.example.json` 复制为 `config.json`，按需填写：

| 配置字段 | 用于功能 | 怎么获取 |
|----------|----------|----------|
| `canvas_token` | Canvas 全套（DDL/课程/成绩/课件/作业/日历） | oc.sjtu.edu.cn → 头像 → 设置 → 「+ 新建访问许可证」 |
| `canvas_profiles` + `canvas_default_profile` | Canvas 学生/教师双身份 | 各生成一个 token，分别填到 `canvas_profiles.student.token` / `.teacher.token`，默认身份填 `canvas_default_profile` |
| `base_url` | Canvas 站点 | 默认 `https://oc.sjtu.edu.cn`，一般不用改 |
| `save_dir` | 课件下载目录 | 默认 `~/Downloads/Canvas课件` |
| `jcourse_api_key` | 选课社区课评 | course.sjtu.plus → 个人中心 → API 密钥 |
| `dyweb_token` | 传承交大资料 | jAccount 登录后抓 token（见 §3.5） |
| `sjtu_username` / `sjtu_password` | 交大邮箱 IMAP/SMTP | 邮箱账号 + 密码（或授权码） |
| `shuiyuan_user_api_key` + `shuiyuan_user_api_client_id` | 水源社区 | 脚本内 `auth init` 链接授权（见 §3.7） |

**等价环境变量**（优先级高于 config.json）：`JCOURSE_API_KEY`、`DYWEB_TOKEN`、`OPENCLAW_SJTU_CANVAS_PROFILE`。

> 🔒 `config.json` 已在 `.gitignore`，含真实凭证，**切勿 `git add`**。只提交 `config.example.json`（占位符）。

---

## 三、功能 × 入口 × 认证 × 状态 总表

| 功能 | 入口脚本 | 认证 | 状态 |
|------|----------|------|------|
| Canvas 课程 / DDL / 成绩 / 身份 | `canvas_api.py` | canvas_token | ✅ 实时 |
| 自动作业扫描 / 作答上下文 / 新作业监测 | `auto_homework.py` | canvas_token | ✅ 实时 |
| DDL+课程 → .ics 日历文件 | `sjtu_timetable_ics.py` | canvas_token | ✅ 实时 |
| DDL → macOS 日历同步 | `calendar_sync.py` | canvas_token | ⚙️ 本地(仅 macOS) |
| 课程评价 / 老师对比 (jCourse) | `sjtu_course_review.py` | jcourse_api_key | ✅ 实时 |
| 传承交大 往年资料 / 试卷 | `sjtu_legacy.py` | dyweb_token | 🔑 需登录 |
| 交大邮箱 未读 / 搜索 / 发信 | `sjtu_mail.py` | 邮箱账密 | 🔑 需登录 |
| 教务通知 / 交大新闻 | `sjtu_news.py` | 无 | ✅ 实时 |
| 正版软件查询 | `sjtu_software.py` | 无 | ✅ 实时 |
| 生存手册 | `sjtu_survive.py` | 无 | ✅ 实时 |
| 视觉交大 官方图库 | `sjtu_visual.py` | 无 | ✅ 实时 |
| 水源社区 (只读) | `shuiyuan_discourse.mjs` | shuiyuan_user_api_key | ✅ 实时 |
| 课件文本提取（配合 Canvas） | `file_extractor.py` | 无 | ✅ 本地 |
| 教学周 / 校历 / 校园巴士 | `sjtu_info.py` | 无 | 📌 静态 |
| 食堂推荐 / 菜单 | `sjtu_canteen.py` | 无 | 📌 静态 |
| 图书馆 信息 / 座位说明 | `sjtu_library.py` | 无 | 📌 静态 |
| 空教室 / 教学楼清单 | `sjtu_classroom.py` | 无 | 📌 静态 |
| 在线工具目录 | `sjtu_tools.py` | 无 | 📌 静态 |
| 交大 PPT 生成 | `generate_ppt.py` | 无 | ⚙️ 本地(需 python-pptx) |
| 手写 PDF 生成 | `handwrite_pdf.py` | 无 | ⚠️ 需装 handright |
| 镜像换源指引 | `sjtu_mirror.py` | 无 | ⚙️ 本地(list 接口已变) |

---

## 四、功能详解

> 每条都标注了**真实子命令**（已逐行核对源码 + 实跑）。脚本帮助里写 `python3 sjtu_xxx.py`，从仓库根
> 调用时实际路径是 `scripts/sjtu_xxx.py`。

### 4.1 Canvas 课程 / DDL / 成绩 ✅ — `canvas_api.py`

需 `canvas_token`。**CLI 仅 5 个只读子命令**，全局 `--profile <name>` 可放命令前后任意位置切换
学生/教师身份（缺省用 `canvas_default_profile`）。无参数时默认执行 `courses`。

```bash
python3 scripts/canvas_api.py me                       # 验证 token + 显示当前身份(学生/教师)
python3 scripts/canvas_api.py courses                  # 列出所有课程: [课程ID] 课程名 [角色]
python3 scripts/canvas_api.py ddls                     # 未来未提交作业, 按截止排序(🔴<24h/🟡<72h/🟢)
python3 scripts/canvas_api.py ddls-all                 # 本学期DDL全景: 统计+待交+老师反馈
python3 scripts/canvas_api.py grades                   # 各课已评分作业的得分
python3 scripts/canvas_api.py --profile teacher courses # 教师端: 管理课程/全班提交/成绩册
```
- 课程 ID 从 `courses` 输出取，用于其它需要 course_id 的脚本。
- DDL 走 Canvas Planner API 一次性拉取（含 submitted/graded/late/missing/feedback）；时区固定 UTC+8。
- ⚠️ 课件下载、作业提交、讨论区、日历事件等是**库函数，未做成 CLI 子命令**；提交作业需另调
  `canvas_api.submit_assignment(...)` 并确认，普通调用不会触发写操作。
- `ddls-all` 的学期起始日在源码里硬编码（`2026-02-17`），跨学期需改源码。
- 依赖 `requests`；需能访问 oc.sjtu.edu.cn（校园网或可达网络）。

### 4.2 自动作业流水线 ✅ — `auto_homework.py`

需 `canvas_token`。5 个子命令（无参默认 `scan`）：

```bash
python3 scripts/auto_homework.py scan                  # 扫所有未提交作业, 按剩余时间升序(只读)
python3 scripts/auto_homework.py urgent 24             # 只看 N 小时内截止(默认48)(只读)
python3 scripts/auto_homework.py context <course_id> <assignment_id>  # 拉题面+下载图片/课件, 生成 ai_prompt.md
python3 scripts/auto_homework.py full <course_id> <assignment_id>     # 同 context + 提示后续提交步骤
python3 scripts/auto_homework.py watch                 # 对比上次状态, 报新增作业(写 .homework_state.json, 适配 cron)
```
- `context`/`full` 会向 `~/Downloads/Canvas作业上下文/` 与 `~/Downloads/Canvas课件/` **下载文件**；
  生成的 `ai_prompt.md` 可喂 LLM 辅助作答。**本身不提交作业**。
- `context`/`full` 提取课件正文需 `python-pptx` / `pdfplumber` / `python-docx`（缺失则该文件标注「需安装」，不崩）。

### 4.3 DDL → 日历 ✅ / ⚙️ — `sjtu_timetable_ics.py` + `calendar_sync.py`

需 `canvas_token`。`sjtu_timetable_ics.py` 3 个子命令，可选第二参为输出路径：

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xhh678876/openclaw-sjtu](https://github.com/xhh678876/openclaw-sjtu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
