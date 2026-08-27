---
trigger: always_on
description: 本文件约束在此仓库工作的 AI 编程代理。面向人类的内容在 `README.md`、`README.zh.md` 与 `docs/`;不要把 AI 专用规范写回 README。
---

# AGENTS.md

本文件约束在此仓库工作的 AI 编程代理。面向人类的内容在 `README.md`、`README.zh.md` 与 `docs/`;不要把 AI 专用规范写回 README。

## 仓库不变量(任何改动不得破坏)

- **唯一仓库 = 唯一插件 dsh-plugin-subhub 的可安装组合包(bundle)**:根 `package.json` 声明 `dsh.bundle.patch`,根 `cordis.patch.yml` 是本插件唯一一行补丁。
- 不上 npm、不拆仓库;纯 JavaScript、无构建步骤、无 `prepare` 脚本;插件源码直接位于 `src/index.js`(宿主半边)与 `src/client.js`(客户端半边)。
- **命名约定**:插件身份一律 `dsh-plugin-subhub`(行 id、挂载模块、客户端模块 id、登录 API `/api/dsh-plugin-subhub/*`、CSS 前缀 `dsh-plugin-sub-*`、凭据目录 `~/.dsh-plugin-subhub/`);具体订阅服务相关 id 一律 `dsh-plugin-subhub-<服务商>`(provider 路由 id、settings.yaml 设置节,如 `dsh-plugin-subhub-openai`)。新服务商沿用此约定,不得引入新前缀。
- 插件如需 npm 依赖,只能加在根 `package.json`(用户只安装根包)。
- 密钥、凭据、token 一律不进仓库文件;机密走环境变量或宿主侧配置。
- **凭据隐私边界**:插件只读写自己拥有的凭据文件(如 `~/.dsh-plugin-subhub/openai-auth.json`),不得读取、复用其它程序(如 codex CLI 的 `~/.codex/auth.json`)的认证文件;用户显式用配置指定文件路径的除外。安装插件不构成对既有登录的授权。
- **provider id 全局唯一**:`ctx.llm` 的 configurable provider id 不能与外壳内置目录重名(内置已声明 `openai`、`deepseek`、`anthropic-messages` 等,见 `dsh-llm-pi-ai`);重名会让插件树以 DUPLICATE_DIRECTORY 加载失败。本插件因此用 `dsh-plugin-subhub-openai`;新提供商选 id 前先核对内置清单。
- **登录门控契约**:provider 路由与「模型」页目录条目只在凭据存在时注册;注册触发器固定为四处——插件挂载、插件设置节变更、网页登录/退出成功回调、中心页状态轮询的自愈补注册(脚本登录后打开一次「第三方订阅」页即可注册)。改动登录链路不得破坏自愈行为。
- **宿主显示名跟随 harness 界面语言**:客户端把 locale 快照(`ctx.locale.getSnapshot().active`)作为 `locale` 查询参数随每次登录 API 调用发给宿主;宿主以显式 `locale.preference` 设置为优先、快照为兜底决定显示名,并用目录条目 `replace()` 原子换名。用户可见名称不得写死中文。
- **依赖只用 pnpm**:仓库根依赖一律 `pnpm install`;禁止 `npm i`(会把临时包写进根 `package.json` 并生成 `package-lock.json`,污染清单)。诊断类临时工具装到自带 `package.json` 的独立目录,用后即删。
- **临时验证产物不进库**:验证用临时目录一律 `.tmp-repro/`(已在 `.gitignore`),结束即删;测试凭据(含假 token 文件)不得提交、不得留在工作区。
- **仓库已公开**:推送前必须跑密钥模式扫描(`sk-`、`eyJ…`、`ghp_`、私钥块等)与绝对路径扫描(`/Users/`、本机用户名等),并核对 `git ls-files` 只含预期清单与插件文件,凭据、会话、设置一律不得入库。

## 用户文档边界

- `README.md`(英文主文档)与 `README.zh.md`(中文镜像)只服务插件使用者,按「定位 → 安装 → 快速开始(登录/选择模型/图片使用/账户管理)→ 命令行登录 → 界面预览 → 安全与隐私 → 支持 → 许可」组织;优先写用户要点击什么、执行什么、看到什么。安全隐私小节直接写在两个 README 末尾,不另建用户文档页面。
- **定位口径**:README 把插件定位为「第三方订阅服务接入插件」,不得写成仅支持某一家订阅;「支持的服务」段落必须与已合入的提供商保持同步。产品方向为**仅 OAuth 认证的订阅账户**,不接入粘贴 API Key 类套餐(密钥类服务直接用 harness「模型」页的内置目录)。模型、额度、可用性由对应订阅服务商与账户决定。
- 不在 README 展开行 id、provider 路由、内部 API 与端点、OAuth/JWT 刷新、缓存与并发锁、SSE、请求字段、附件编码、工具注册方式、源码目录职责等实现细节。稳定的维护约束写在本文件,短期实现事实留在代码及 `docs/development.md`。
- 不在 README 写死某个外部模型当前是否可用、是否支持图片或有哪些思考档位;以界面根据账户能力显示的结果为准。
- 用户文案避免「完全」「实时」「永不」「与官方实现等价」等无法长期保证的绝对说法。描述外部服务时明确模型、额度、限速和可用性由服务商与账户决定。

## 功能行为契约

- **模型目录**:正常情况下只展示账户接口返回且允许显示的模型;在线目录不可用时才使用静态备用列表,不得把备用项混入成功的在线结果。上下文窗口、可选及默认思考档位、图片输入能力优先采用模型目录字段,不得在用户文档中维护易过期的固定模型快照。
- **思考档位**:发送值必须符合后端接受的档位;附加的代理行为只能在所需工具真实存在时启用。不得把提示驱动的行为描述成模型必然执行。`ultra` 当前到 `max` 的映射与主动委派方式属于实现细节,变更时需验证选择器兼容性和会话行为。
- **图片输入**:用户附件和工具结果图片都应传给声明支持图片的模型;不支持图片的模型必须在发送前拒绝。后端拒绝工具结果图片时只做有限降级重试,不得无限重试。
- **图片工具生命周期**:`generate_image` 随有效凭据注册,退出登录后注销。生成或编辑成功必须得到附件服务可接受的真实图片,写入附件并在助手消息中回显;没有图片字节或附件写入失败时必须如实报错,不得伪造成功。回显按 attachment id 去重,后续轮次仍应能使用该图片。
- **图片编辑**:`edit_latest_image: true` 使用当前会话最近一张可用图片;没有图片时明确要求用户先上传,不得悄悄退化为文生图。
- **本机登录边界**:网页登录 API 只接受通过 `127.0.0.1`、`localhost` 或回环地址访问的请求,不得为反向代理无条件放宽。远程场景引导使用 SSH 端口转发或独立登录脚本。
- **适配器兼容性**:只发送目标后端支持的参数,推理档位限制在模型目录声明的选项内;当前 stop 序列和输出 token 上限的处理方式是实现细节,修改时必须验证后端兼容性。

## 新订阅商接入规范(所有第三方订阅一律适用)

1. **模型与思考档位必须动态获取,不得写死**——不同订阅档位可用的模型与思考深度不同,写死会导致用户升级订阅后部分模型/档位不可用。选择器只展示账户目录声明的模型与档位;静态列表仅作离线兜底,绝不替代在线结果(见「功能行为契约·模型目录」)。若服务商没有账户模型目录接口,以其官方已知模型清单为目录并保持精简,规格文件中必须写明该事实。
2. **多思考档位按从低到高排序,首项为 Off**(仿照 deepseek 模型的设计);Off 仅在账户目录声明关闭档时展示——若后端拒绝显式 off(如 xai 实测 HTTP 400 invalid reasoning effort),只展示目录声明的档位;默认档优先取账户目录声明的默认值,其次才用配置。
3. **声明多模态(图片输入)的模型必须实测**——以真实账户或等效往返测试覆盖「用户图片输入」与「工具结果图片」两条路径(见「功能行为契约·图片输入」),发现问题必须修复,验证通过前不得视为接入完成。
4. **不得重复接入外壳内置目录已有的服务,且只接入 OAuth 订阅**——接入前核对 harness 的 `dsh-llm-pi-ai`(基于 pi-ai 内置目录)与「模型」页「Add provider」清单:内置已提供同一服务与同一凭据方式的(如 `minimax-cn`、`qwen-token-plan-cn`、`openrouter`、`zai-coding-cn` 的 API-key 路由),插件不再接入。插件产品方向是**仅 OAuth 认证的订阅账户**(OpenAI/xAI/GitHub),不接入任何粘贴 API Key 类的套餐;内置的 api-key 路由与插件的订阅 OAuth 登录凭据模型不同,属互补而非重复。

接入清单、xAI 实战踩坑速查(版本门/指纹头、历史消息 `usage`/`stopReason`、工具结果图片回声、`latestConversationImageRef` 事件扫描等)与真实账户最小验证配方见 `docs/development.md` 的「新订阅商接入:规范与实战速查」。

## 配置兼容面

- `settings.yaml` 中 `dsh-plugin-subhub-openai` 节当前识别 `authFile`、`baseURL`、`apiBaseURL`、`defaultContextWindow`、`modelsCacheTtlMs`、`defaultReasoningEffort`、`streamIdleTimeoutMs`、`enableImageTool`、`imageModel`、`retryPolicy`。其中 README 或发布说明曾面向用户公开的键属于兼容面;内部调优键可以演进,但删除或改名之前必须检查仓库历史与用户迁移影响。
- `settings.yaml` 中 `dsh-plugin-subhub-xai` 节当前识别 `authFile`、`baseURL`、`apiBaseURL`、`defaultContextWindow`、`modelsCacheTtlMs`、`defaultReasoningEffort`、`streamIdleTimeoutMs`、`retryPolicy`;推理档位的可选值来自账户目录的 `reasoning_efforts` 声明,不在插件内写死。
- `settings.yaml` 中 `dsh-plugin-subhub-github` 节当前识别 `authFile`、`baseURL`、`defaultContextWindow`、`modelsCacheTtlMs`、`streamIdleTimeoutMs`、`retryPolicy`(不含 `defaultReasoningEffort`,GitHub Copilot 无推理档位)。
- `authFile` 一经显式配置,登录、刷新、状态查询与退出必须始终使用该文件;退出会删除它,因此不得暗中改读其它程序的凭据。
- 内部端点、请求体候选形态、缓存默认毫秒数和重试步骤属于可变实现,除非升级为兼容性承诺,否则不要复制到 README。

## 文件职责

- `README.md`:人类文档主文件,用英文;`README.zh.md`:中文镜像,与 `README.md` 结构同步;`docs/`:人类文档,用中文。
- `AGENTS.md`:本文件,AI 行为规范。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kinoward/dsh-plugin-subhub](https://github.com/kinoward/dsh-plugin-subhub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
