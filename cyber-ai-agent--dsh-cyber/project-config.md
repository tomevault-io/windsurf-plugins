---
trigger: always_on
description: 本文件约束本仓库中的 Agent、贡献者与自动化改动。实现功能前遵守现有 contracts、持久化与领域边界；修改界面时必须完成以下视觉审批，不能只以类型检查或单张截图代替。
---

# DSH Cyber 工程与视觉护栏

本文件约束本仓库中的 Agent、贡献者与自动化改动。实现功能前遵守现有 contracts、持久化与领域边界；修改界面时必须完成以下视觉审批，不能只以类型检查或单张截图代替。

## 新开发分支基线（强制）

- 每个新需求开始前必须执行 `git fetch origin --prune`，并从当时的 `origin/main` 新建分支；不得从旧功能分支继续派生后声称基于最新代码。
- 若依赖 PR 尚未合并，必须先按依赖顺序合并、重新 fetch，再创建或快进新分支。恢复暂存改动前核对基线。
- 开发前验证并记录：`git branch --show-current`、`git rev-parse HEAD`、`git rev-parse origin/main`、`git merge-base HEAD origin/main`；开始改动前后三个提交值必须一致。

## 基础视觉审批（合并门禁）

1. 图片与世界地图自适应
   - 在 1440×900、1920×1080、3840×2160 三种视口实际检查。
   - 主视觉默认铺满其容器并保持比例；不得出现由错误 `contain`、固定尺寸或错误定位造成的大段黑边、空白、缩成一团、拉伸或裁掉主要对象。
   - 如同时需要“铺满”和“完整全景”，默认采用铺满取景，并提供明确的“显示全景”控制。
2. 文字对比度与透明度
   - 正文、说明、状态文字必须在实际背景上清晰可读；不得用低透明度叠加代替合格的文字颜色。
   - 正文与关键操作至少达到 WCAG AA 对比度；禁用会令文字发灰、发虚的父级 `opacity`。
3. 语言一致性
   - 面向中文用户的标题、介绍、状态、按钮与能力说明统一使用中文。
   - 品牌名、版本号、软件包 ID、协议名等不可翻译标识只能出现在明确的技术详情位置；不得在主要介绍文案中中英文混杂。
4. 字号与信息密度
   - 常规正文以 14px 为基线，辅助文字不得小于 12px，关键标题与操作应更大。
   - 8–10px 只允许用于非关键技术标注，且必须可放大或在详情中查看；不得用于主要说明、状态、按钮、卡片正文或表单标签。
   - 不得通过缩小字号来容纳过多信息；优先分组、换行、折叠次级技术详情或增加合理空间。
5. 审批证据
   - 截图必须覆盖上述三个视口，并记录控制台 error/warn。
   - 对每个视口逐项记录：图片适配、大段空白、文字对比度、语言一致性、最小字号、文案可读性。
   - 任一项未通过时不得标记视觉验收完成。

## 产品信息架构（不得回退）

- 顶部工具栏是全局能力入口，只保留一个统一的“市场”和一个“创意工坊”入口；主题、插件、角色模板属于同一个市场内部分类，禁止重新拆成三个顶栏按钮。
- 创意工坊属于全局创建工具，放在顶栏并按需加载；不得塞回左侧会话栏，也不得把重型编辑状态继续堆进 `App.tsx`。
- 左侧采用类似微信首页的会话模型：**只展示会话**。禁止重新加入独立角色列表、通讯录列表、文件列表或市场入口。
- 一个活动角色在一个世界中最多一个 canonical 私聊；管家默认置顶；私聊和群聊均支持置顶/取消置顶与从列表隐藏。隐藏会话不删除历史，重新从角色/世界进入时恢复同一 canonical 会话。
- 角色实例的浏览、新增、设置、Skill 授权和成长记录统一放在右侧“档案”。市场负责安装角色模板，档案负责把模板实例化为角色，两者语义不得混用。
- 右侧 Dock 默认只常驻“世界”“轨迹”和“更多”。“更多”承载“角色”“知识”“产物”“日程”等低频入口；点击后将对应页面提升为可关闭的一级临时页签，并按世界在本机恢复打开顺序。关闭页签只收起入口，不删除内容或状态。“更多”始终可见，窄屏时临时页签区可横向滚动。文件读取/预览 API 可以保留，但“文件/预览”入口暂时隐藏；未经新的产品评审不得重新暴露。
- Chat 只展示最终会话结果：用户消息、最终回复、附件和明确的产品通知。推理摘要、工具调用、工具结果、任务/协作/Skill/世界执行明细统一进入“轨迹”，不得重新塞回聊天气泡。
- 世界轨迹是 `DomainEvent`、`WorkMessage`、`CharacterSkillAction` 和未来 `ScheduledRun` 的只读投影，不是新的事实表；新增来源必须注册 Trace Adapter，禁止在 Trace 核心按供应商写条件分支。
- Trace 输出必须统一经过脱敏，不得包含 API key、Authorization、Cookie、密码、token、完整 Prompt、原始工具输入/结果、敏感文件内容或隐藏思维链。
- 世界视图只负责可视化和角色互动，不负责新增/删除角色；空世界只引导用户前往“档案”。
- 新增角色后必须立即刷新 Conversation Hub，使新角色的 canonical 私聊无需刷新页面即可出现在左侧。
- 浏览器 E2E 必须验证以上信息架构。旧测试出现“左侧添加角色”“角色市场弹窗”“文件/预览 Tab”等断言时，应更新测试，不得为了通过测试恢复旧 UI。

## 本地优先与升级安全

- 当前创意工坊、世界、角色、会话、成长档案、Skill 动作与本地资产以用户机器上的 `stateRoot` 为权威数据源。未来服务器能力只能作为可选同步/备份层，不能替代本地所有权。
- 应用源码目录与 `stateRoot` 必须保持物理和逻辑解耦。默认数据目录继续使用 `%LOCALAPPDATA%\\DSH Cyber`（Windows）或 `~/.dsh-cyber`（macOS/Linux）；禁止默认把用户数据写进 Git 工作树。
- `git pull`、重新安装依赖、重新构建 Web/CLI、替换内置 Harness bundle 都不得删除、覆盖或重新初始化已有 `stateRoot`。
- 数据迁移只能做 versioned migration，并必须保持向前迁移、失败可诊断；不得用“重新创建世界/角色”代替迁移。
- 新增任何持久化目录时，必须同步加入本地 Backup Bundle。当前 Bundle 至少包含：SQLite、`worlds/`、`assets/`、`packages/`、`workshop/`、`skills/`、`integrations/`；凭据、运行时二进制、缓存和旧备份继续排除。
- 应用更新、Harness 激活和 Harness 回滚都必须使用相同的完整本地 Backup Bundle 边界，不能只保护 SQLite 而遗漏 Workshop/Skill 文件。
- 升级说明必须先给备份命令，再给 `git pull --ff-only` / install / build / doctor / restart 流程，并明确指出这些命令只更新程序，不触碰 `stateRoot`。
- 文档中不得建议用户通过删除 `~/.dsh-cyber`、`%LOCALAPPDATA%\\DSH Cyber`、`data/`、`worlds/`、`workshop/` 来解决普通升级问题。

## 世界、角色与 Skill 解耦

- `World` 只负责世界身份、主题/场景能力、文件根、设置与运行时投影；不得根据角色显示名称写业务分支。
- `EmployeeBlueprint` 只声明角色身份、Persona、请求的 Skill/Capability 与可移植具身语义；角色名称不是权限来源，也不是空间行为来源。
- `EmbodimentProfile` 只能保存 role/zone/facility/ambient/rig 等语义标签；禁止保存像素坐标、路径、动画帧逻辑或 `role.includes(...)` 之类的运行时判断。
- `requestedSkills` 与 `skillGrants` 必须保持不同含义：模板和创意工坊只能请求能力，实际授权必须由角色 revision/审批链显式产生。
- Skill Runtime 只负责授权、路由、调度、持久化与审计。任何 Home Assistant、GitHub、浏览器、飞书等供应商逻辑必须放进独立 `CharacterSkillAdapter`；禁止在核心 Runtime 中继续添加 `if (skillId === ...)`。
- 一个 Skill ID 只能由一个活动 Adapter 提供；注册冲突必须失败，不能用注册顺序静默覆盖。
- 第三方包不能拿到 Adapter 实例、系统命令、任意网络或任意文件权限。包只声明请求，宿主受信任 Adapter 执行结构化动作。
- 外部副作用必须记录 `risk`、`authorization`、`adapterId`、结构化参数和最终执行状态；凭据不得进入动作记录、Prompt、SQLite 普通字段或前端响应。
- 计划任务执行前必须重新检查角色是否存在、是否归档、世界归属和当前 Skill Grant；撤销授权后不得继续执行旧计划。
- 群聊按 Skill 选中角色不等于外部动作已执行。讨论或任务协作在创建 AgentRun 前必须进入 CharacterSkillRuntime；同一用户请求只允许首个实际匹配且已授权的角色提出一个宿主动作，额外 proposal 在持久化和执行前截断，避免多人或多动作重复外发。需要审批时 WorkTurn 与 Queue 同时 `waiting-approval` 且不占角色通道；批准、拒绝或过期后继续原群聊 WorkTurn，复用已持久化 Action 事实，禁止重新 prepare 或重复 Adapter。

## Harness 设计采用规则

- 参考 DeepSeek Harness 的 provider/registry 思路：新增能力优先注册在稳定接口旁边，不修改 Agent loop；注册应可替换、可测试，并有清晰作用域。
- Skill 的“部署/发现”与角色的“获授权消费”必须分离。后续 Registry 演进优先采用 Host + workspace/world/character scope 解析，而不是在角色数据里复制 Provider 实现。
- 参考 Codex 的最小权限与逐动作审批思路：权限判断发生在具体动作边界，危险能力不得因为“安装了插件”或“角色请求过 Skill”就默认放行。
- 可复用审批只能收敛成明确的 Skill/Action/Target/Scope 策略；禁止“以后这个插件全部允许”这种宽泛授权。
- DSH Cyber 上层领域模型不绑定任一 Harness 内部 API。DeepSeek Harness、Codex 或未来其他 Harness 只能通过适配层提供模型、工具、子代理或执行能力。
- Harness 返回的真实工具/执行结果才能注入 Agent 上下文；UI 动画、NPC Ambient Life、模型自然语言声明都不能替代执行事实。

## 创意工坊构建规范

- 创意工坊生成物采用 `project source -> generated package -> PackageManager -> runtime entity` 流程，禁止直接把表单数据写成活动角色绕过包验证。
- 在创建世界前必须先验证全部角色输入、EmbodimentProfile、manifest 和 PackageManager preview；单个角色校验失败不得留下半个世界。
- PackageManager 仍拥有包 staging、内容哈希、激活和单包回滚边界；Workshop 不复制一套安装逻辑。
- Workshop 项目源与 generated 产物保存在 `stateRoot/workshop`，应用源码升级不得覆盖用户项目。
- 创意工坊前端必须独立组件化并按需加载；不要继续扩大 `App.tsx`，世界编辑器、角色编辑器、Skill 选择器应保持可复用边界。
- 新增世界模板、角色语义预设或 Skill Adapter 时必须补合同/服务测试；核心创建流程必须补 Chromium E2E。

## 世界与会话隔离

- Product rule: **Chat surfaces final conversational results. Execution details belong to Trace.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyber-ai-agent/dsh-cyber](https://github.com/cyber-ai-agent/dsh-cyber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
