---
trigger: always_on
description: 本文件适用于采用它的整个仓库，可复用于不同的 DeepSeek Harness（下称 DSH）外部插件项目。规则不得写入某个插件独有的协议、namespace、业务对象、字段或产品行为；项目专属事实应从当前仓库的 README、manifest、源码和测试中读取。
---

# AGENTS.md — DeepSeek Harness 插件项目通用规则

本文件适用于采用它的整个仓库，可复用于不同的 DeepSeek Harness（下称 DSH）外部插件项目。规则不得写入某个插件独有的协议、namespace、业务对象、字段或产品行为；项目专属事实应从当前仓库的 README、manifest、源码和测试中读取。

## 权威来源与开始工作前的检查

- 使用本规则的仓库是 **DSH 插件项目**，不是独立应用、DSH fork 或 DSH 内置 package。默认通过公开插件扩展点实现需求，不为接入插件而修改 DSH 本体。
- 本地 DSH 源码的固定位置是 `D:\work\workspace\github\deepseek-harness`。开始任何涉及 DSH API、构建、运行时或 UI 的工作前，先确认该目录存在；若不存在，停止相关工作并向用户询问，不得自行克隆、改用其他 checkout 或猜测接口。
- 先检查当前插件仓库的 `package.json`、锁文件、README、构建配置、patch、exports、源码目录和测试，确定它实际包含 Host、Client、Service、Tool、Provider、Consumer 或 bundle 中的哪些部分。不得假设所有插件都有相同目录结构或同时包含 Host/Web 两半。
- 官方技术文档入口如下。这些 URL 是章节入口而非单页；根据任务沿侧边栏继续阅读相关子页，不能只读取入口页：
  - https://deepseek-harness.github.io/deepseek-harness/guide/quickstart
  - https://deepseek-harness.github.io/deepseek-harness/develop/basic/
  - https://deepseek-harness.github.io/deepseek-harness/reference/
- 插件生命周期、服务、事件、配置、发布或 UI 任务必须继续阅读对应的 Develop、Cordis tutorial、Cordis API、subsystem 和 cookbook 子页。设置界面任务还必须阅读 `reference/cookbook/adding-a-settings-card`、`reference/subsystems/client-modules` 与 `reference/subsystems/settings`。
- 在线文档表达公开设计和最新约定，本地 checkout 表达当前实际 ABI、slot 结构和视觉实现。两者不一致时，记录本地 commit，核对目标版本的类型与源码，采用能在该 checkout 上验证通过的接口并说明差异；不得静默拼接两个版本的 API。
- 将 sibling checkout 视为只读参考和联调目标。除非用户明确要求，不得在 `D:\work\workspace\github\deepseek-harness` 中提交、格式化、生成或修改文件。

## 插件架构与职责边界

- 遵循 DSH“行为由插件组成”的架构。新行为应进入已有公开 Service、event、registry、slot、settings 或 bundle 扩展点；不要修改 agent loop、Web shell 或其他核心包来绕过缺失的插件接入。
- 先识别当前插件在 capability seam 中的角色：Service Definition、Provider、Consumer，或只负责组合它们的 bundle。新增能力时必须考虑当前所需角色及其真实调用方，不把 provider 私有行为塞进公共 service，也不把某个 UI/transport 的需要写进通用 service 接口。
- 保持 Host、Client、模型可见行为、持久化和 transport 的职责分离。跨插件协作通过 Cordis service、typed event、registry 或 typed slot；不要导入另一个插件的私有实现来共享状态。
- 除非需求明确改变公开结构，保留当前仓库已经声明的 package 边界、入口、subpath exports 和 patch 组合方式。不要为了套用别的插件示例而重排现有架构。
- 所有新增公开行为都要有明确所有者。配置由注册 namespace 的插件所有，UI 只编辑；持久状态由拥有它的 subsystem 所有；transport 只负责传递，不成为业务真源。

## Cordis 插件与生命周期

- 函数插件把适用的 `name`、`inject`、`Config`、`apply` 元数据作为命名导出，不要混入 default export；提供具名服务的插件才使用默认导出的 `Service` 子类。以目标 checkout 当前 package 约定为准。
- 必需服务写入 `inject`，让 fiber 等待依赖并在依赖消失时自动卸载；可选服务在使用点通过 `ctx.get(name)` 查询。不要通过手工启动顺序代替依赖声明。
- 所有注册和资源必须属于 fiber 生命周期。优先使用本身可撤销的 `ctx.on()`、registry `register()` 和 `ctx.plugin()`；Cordis 不管理的连接、定时器、watcher、订阅或进程必须放入 `ctx.effect()` 并返回 disposer。
- `ctx.plugin()` 创建的 child fiber 随父级卸载。需要提前替换或终止时等待 `fiber.dispose()` 完成；不能泄漏注册、子进程、网络连接、监听器、重试或后台任务。
- 异步 disposer 之间需要顺序时，把相关步骤放进同一个 disposer 并显式串行等待；不要依赖多个 effect 的完成顺序。实现必须能经受配置更新、依赖消失、显式 dispose 和 HMR。
- 新 registry contribution 必须验证 dispose 后确实移除。配置热替换不得留下旧实例状态；用稳定 entry `id` 保持 Loader/HMR 只重载发生变化的部分。
- waterfall listener 除非明确拥有短路决策权，否则必须调用 `next()`；事件模式是公开约定，使用与声明一致的 `emit`、`waterfall`、`parallel` 或 `serial` 分发方式。
- 需要跨插件公开能力时使用 Service；仅由单个实现内部使用的逻辑保持私有。函数、service、事件和声明合并均使用目标 package 的公开入口，不从 `src/` 或未导出的内部路径导入。

## 配置与 Settings

- 配置类型与同名 Schemastery `Config` schema 同步维护，默认值写在 schema 中。部署可调值必须是配置字段；能在加载时判断的错误要大声失败，跨字段或依赖资源的约束在最早可判断处失败。
- `cordis.yml` 保留部署组合；用户可编辑子集才进入 settings namespace。namespace 由拥有配置的插件选择一次，并在 Host、Client 与文档中保持一致；遵循目标版本要求的命名和 brand，不复制其他插件的 namespace。
- Host 是 schema、默认值、composition base、变更应用和最终校验的唯一所有者。已有 Loader entry 的 owner 优先使用目标 checkout 公开的 `installSettingsSection`；实时 owner 通过 scope/watch 响应提交后的变更，`restart` owner 则明确声明生效时机。
- 浏览器设置 UI 优先通过目标 checkout 的 `ctx.settingsScope.bind({ namespace })` 读写。scope snapshot 中 `value`、`base` 与原始 `user` 层含义不同；字段是否被用户覆盖按其是否出现在 `user` 层判断，reset/unset 应恢复继承值而不是写回一个复制的默认值。
- Client 校验只提供即时反馈，不能替代 Host schema/validator。写入携带最近读取的 namespace revision；冲突、拒绝或失败时重读并显示失败，不得 last-write-wins 覆盖未知更新。
- 机密字段使用目标版本支持的 schema secret role 或 credentials reference。任何 wire、日志、错误、遥测、fixture 或 snapshot 都不得泄露 secret。持有脱敏视图的调用方只能发送 path-addressed `set`/`unset` 操作，不能从脱敏 snapshot 重建并整体 replace 分节。
- 只有在标准 settings transport 无法服务目标插件、且当前源码/文档确认需要时，才增加插件自有 Remote。自有 Remote 仍须经过 Host owner、schema 校验、secret redaction、revision fence、写后回读，以及外部变更和重连刷新；不得直接修改 settings 持久化文件。
- 注册、持久化、通知遵循提交点：操作成功后才发布新状态。对设置 provider 的外部编辑与进程内写入都要维持同一个权威读取路径。

## Bundle、发布与 Client 模块

- 先根据当前仓库目标判断它是普通插件依赖、可安装 bundle，还是两者兼有。可安装 bundle 在 `package.json` 声明 `dsh.bundle.patch`；profile 是用户运行时组合，插件包不得把自己声明为 profile。
- 对同时提供聚合安装与按需安装的多功能仓库，新增功能不得只加入聚合包。每个新增功能必须同时提供可独立安装的 package/bundle，并接入仓库现有的功能选择与安装流程；省略选择时可保留全量安装，但用户必须能只安装该功能而不安装其他功能。
- 独立包只声明该功能实际需要的 Host、Client、Service、配置与 patch entries，不得通过依赖聚合包或加载聚合入口间接带入其他功能。聚合包只负责组合各功能；可共享源码和构建基础设施，但独立发布物必须自包含并可单独完成 `prepare`、pack、安装与加载。
- 功能选择信息必须有单一权威来源，至少包含稳定的安装名称、对应 package/bundle、构建目标及必要的冲突/迁移信息。新增、重命名或移除功能时，要同步更新安装器的列举、选择、默认全量、清理未选功能和加载验证逻辑，不得在多个脚本中维护容易漂移的平行映射。
- 按需安装的选择表示目标 profile 最终应保留的功能集合。安装器应先成功构建并安装全部所选功能，再移除被替代的聚合包、未选择的同仓库功能包及已声明冲突的旧包；任一步失败时不得提前破坏原有可用安装。
- patch 只贡献本插件所需的最小 Loader entries，使用稳定且不冲突的 `id`。后应用层会整体替换目标行的 `config`，不是深度合并；不要依赖未声明的隐式层顺序。
- 保持 ESM、严格 TypeScript 和必要的公开 exports。包内相对 ESM import 遵循当前编译配置要求的扩展名；跨 package 使用包名，不引用 sibling checkout 的未发布源码路径作为运行时依赖。
- Git 安装取得源码而非构建产物。若插件支持 git 安装，提供自包含 `prepare` 产出全部发布入口，不能依赖 sibling checkout 才能构建；tarball/npm 发布物应预先包含构建产物并通过 files/exports 检查。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sky-unicorn/dsh-enhanced-plugins](https://github.com/sky-unicorn/dsh-enhanced-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
