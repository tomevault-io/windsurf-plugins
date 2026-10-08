---
trigger: always_on
description: > 本文件是本项目给所有 AI 编码代理（Claude Code / Codex / Cursor 等）的统一说明，也是 catbus 的规范本身。
---

# AGENTS.md — catbus（猫巴士）

> 本文件是本项目给所有 AI 编码代理（Claude Code / Codex / Cursor 等）的统一说明，也是 catbus 的规范本身。
> `CLAUDE.md` 通过 `@AGENTS.md` 引用本文件，**只维护这一份**。

## 1. 目标

做**一个 CLI —— 操作各大平台接口的统一入口**。

- 一个命令 `catbus`，按统一的命令规范调用 10 个平台：小红书、抖音、TikTok、B 站、快手、微博、闲鱼、淘宝、京东、X。
- 能力来源是作者 [cv-cat](https://github.com/cv-cat) 的 10 个开源仓库，已克隆到 `references/`，作为**只读的原始来源**。
- 所有平台都在本仓库里**用 TypeScript 重写**，不在运行时调用上游的 Python。
- **用户只需要安装 Node**（版本见 7.1）。不需要 Python，不需要 C/C++ 编译器，也不需要其他系统依赖。
  - `npm i -g catbus-cli` 装完就能离线使用。
  - 必须支持全部目标系统（见 7.2）。
- 以 **npm 包**分发，不发 PyPI。安装体积不是约束，优先保证开箱即用。
- 上游目前只有 **web 端**接口。规范为 **app 端**、**pc 端**预留位置，例如 `catbus xianyu item get <id> -e app`。
- 登录态统一保存在 `~/.catbus/`，**按 平台 × 端 × 账号** 隔离。web 端不登录几乎看不到内容，所以都需要登录；游客态留给 app / pc 端。
- 输出对人和对 Agent 都友好：stdout 只输出 JSON，字段跨平台统一。
- **同义同名**：含义相同的能力在所有平台上用同一个命令、同一组参数、同一种输出结构。例如商品一律叫 `item`，搜索一律叫 `search`。
- 将来的非 JS 上游（Go / Java……）通过 provider 协议接入（见 7.8）。

### 1.1 命名（已定）

| 项 | 取值 |
|---|---|
| 品牌 | **catbus（猫巴士）**：一只能载你去任何地方的猫，呼应 cv-cat |
| slogan | 上车，开往任何平台 / all aboard, every platform |
| 命令 | `catbus` |
| npm 主包 | `catbus-cli`（npm 上的 `catbus` 已被别的库占用） |
| npm scope | `@cv-cat`，用于 `@cv-cat/catbus-assets-jd`、`@cv-cat/catbus-assets-ocr` 和将来的 provider 包 |
| 仓库 | https://github.com/cv-cat/catbus ，默认分支 `master` |
| 许可证 | MIT（主包、assets-jd、assets-ocr 相同） |
| 数据目录 | `~/.catbus/`，可用 `CATBUS_HOME` 覆盖 |
| 环境变量前缀 | `CATBUS_` |
| Logo | `assets/logo.svg`（图标）、`assets/logo-wordmark.svg`（横版）、`assets/banner.txt`（终端字符画）。猫巴士：车身就是一只橘猫，车头是猫脸和耳朵，车尾是尾巴，轮毂是肉垫；车顶路牌是终端提示符 `>_`；车窗里坐着小红书红、B 站粉、抖音青三只小猫乘客。会颠簸、转轮子、摇尾巴、眨眼、抖耳朵；字符画沿用 `( >ω_ )`。配色取自 Catppuccin Mocha |

## 2. 范围

- 目前支持第 3 节的 10 个平台的 web 端；app / pc 端在注册表里为 planned（见 4.3）。
- 矩阵里的 ○（平台有这个概念、上游没有的能力）只占位：注册为 planned，执行时返回 NOT_IMPLEMENTED，有需要时再实现。移植中发现矩阵与上游不符（上游其实没有、或只是占位）的能力，按上游实际情况标为 ○ 或 ◐，以 `docs/capabilities.md` 为准。
- 新平台开始移植时，先把平台 id、别名、`item` 叫法等登记到第 3 节、4.2 和 4.5，再进注册表；在那之前 `catbus <平台>` 是未知平台（`USAGE`）。
- 改规范先改本文件，再写代码。

## 3. 参考仓库 `references/`

| 平台 id | 仓库 | 主要入口 |
|---|---|---|
| xhs | Spider_XHS | `apis/xhs_pc_apis.py`、`apis/xhs_creator_apis.py`、`xhs_utils/` |
| douyin | DouYin_Spider | `dy_apis/douyin_api.py`、`dy_apis/douyin_creator_api.py`、`dy_apis/login_api.py` |
| tiktok | TiktokApis | `api/tiktok.py`、`api/tiktok_web.py`、`signing/` |
| bilibili | BilibiliApis | `apis/bili_apis.py`、`apis/bili_*_apis.py`、`builder/auth.py` |
| kuaishou | KuaiShou-Spider | `ks_apis/kuaishou_api.py`、`ks_apis/publish_api.py`、`ks_apis/login_api.py` |
| weibo | WeiboApis | `apis/weibo_apis.py`、`apis/weibo_mobile_apis.py`、`apis/weibo_creator_apis.py` |
| xianyu | XianYuApis | `goofish_apis.py`、`goofish_live.py` |
| taobao | TaoBaoApis | `taobao_apis.py`、`taobao_live.py`、`utils/taobao_utils.py` |
| jd | JdApis | `jd_apis/jd_api.py`、`jd_apis/jd_login_api.py`、`builder/auth.py` |
| x | XApis | `x_apis/x_api.py`、`x_apis/x_write_api.py`、`x_apis/login_api.py` |

参考资料：
- 每个仓库的鉴权、登录、API 方法、JS 资产、游客态生成位置，以及 Python → JS 的替换对照，见 [docs/upstream-map.md](docs/upstream-map.md)。
- 各平台的能力矩阵见 [docs/capabilities.md](docs/capabilities.md)。

规则：
- `references/` 只读。
  - 可以阅读，也可以运行（用来生成对拍数据，见 7.5）。
  - 可以在上游 gitignore 的位置装依赖，例如 `node_modules/`。
  - **不能修改其中任何受版本控制的文件**。
- `references/` 不进入本仓库的版本控制，它们各自是独立的 git 仓库。
- 每个平台的移植基于哪个上游 commit，记录在 `src/platforms/<p>/UPSTREAM`。

## 4. 命令规范

### 4.1 语法

```
catbus <platform> <resource> <action> [参数...] [选项...]
```

- 选项可以出现在任意位置。`--` 之后的内容一律按参数处理，用于以 `-` 开头的文本。
- 端用选项 `-e/--endpoint` 指定，默认 `web`（见 4.3）。

示例：

```bash
catbus xhs item get "https://www.xiaohongshu.com/explore/<id>?xsec_token=..."
catbus xhs user items <user> --all -o jsonl
catbus bilibili item get BV1xx411c7mD
catbus douyin comment list <item> --limit 100
catbus x user get elonmusk
catbus jd item search 机械键盘 --sort sales
catbus taobao msg send "在吗" --item <商品 url>
catbus bilibili live listen <room> --duration 10m
catbus xhs auth login
catbus kuaishou auth status -a work
catbus xianyu item get <id> -e app           # app 端尚未实现：NOT_IMPLEMENTED，退出码 4
```

### 4.2 平台 id 与别名

| 规范 id | 别名 |
|---|---|
| `xhs` | `xiaohongshu`, `rednote` |
| `douyin` | `dy` |
| `tiktok` | `tt` |
| `bilibili` | `bili`, `b` |
| `kuaishou` | `ks` |
| `weibo` | `wb` |
| `xianyu` | `goofish`, `xy` |
| `taobao` | `tb` |
| `jd` | `jingdong` |
| `x` | `twitter` |

目录名、配置键、输出里的 `platform` 字段一律使用**规范 id**。只有平台有别名；resource 和 action 没有别名。

### 4.3 端（endpoint）

- `-e/--endpoint web|app|pc`，默认 `web`，没有配置项可以改默认值。
- 每个平台在注册表里声明各端的状态：目前 web 为 available，app / pc 为 planned。

命令无法执行时，按下面的顺序判断：

| 情况 | code | 退出码 |
|---|---|---|
| 端为 planned，命令在词表内（第 4.5、4.7 节） | `NOT_IMPLEMENTED` | 4 |
| 端可用，命令是矩阵里的 ○（规划中） | `NOT_IMPLEMENTED` | 4 |
| 命令是矩阵里的 —（平台没有这个概念），或者不在词表内 | `UNSUPPORTED` | 2 |

- `UNSUPPORTED` 时，如果别的端有这个命令，`hint` 里提示加 `-e <端>`。
- 用了平台原生叫法（如 `note`、`video`），`hint` 里提示对应的规范词（见 4.6）。
- 参数和选项的校验（`USAGE`，以及筛选取值的 `UNSUPPORTED`）先于 ○ 的判断：写错的命令先报用法错误。端为 planned 时不做校验。

### 4.4 动作语义

| 动作 | 语义 |
|---|---|
| `get` | 取单个对象 |
| `list` | 取一个集合，不需要关键词。默认是当前账号自己的，或者推荐流 |
| `search` | 按关键词搜索 |
| 成对动作 | `like/unlike`、`collect/uncollect`、`repost/unrepost`、`follow/unfollow` |
| 写入 | `add`、`send`、`publish`、`create`、`update`、`delete`、`upload` |
| 长连接 | `listen`：持续输出 jsonl，直到 Ctrl-C 或 `--duration` 到期 |

### 4.5 通用词表

| resource | 含义 | action |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cv-cat/catbus](https://github.com/cv-cat/catbus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
