---
trigger: always_on
description: 本文件是 AI 工具参与本项目开发时的基础规则。修改代码前先阅读本文件；当规则与局部代码风格冲突时，优先保持现有模块风格，并补充必要说明。
---

# AGENTS.md

本文件是 AI 工具参与本项目开发时的基础规则。修改代码前先阅读本文件；当规则与局部代码风格冲突时，优先保持现有模块风格，并补充必要说明。

## 项目定位

- 本项目是基于 Hyperf 3.2、Swoole 6.2、PHP >= 8.4 的通用开源后台架构。
- 目标是低成本接入新模块，同时保证权限、数据范围、租户隔离、发布升级和性能默认安全。
- 后端代码主要位于 `plugin/*/src`，业务插件前端、语言包、迁移等资源位于 `plugin/*/stc`，主前端 `web/apps/web-antd` 只作为通用壳、公共页面、共享组件和编译期插件宿主。
- 插件通过本地 Composer path 包、Provider、`plugin.json`、菜单/节点同步和 Web 编译期扫描接入；源码/CI 模式下由 Library 内置 `xadmin:plugin:package/install/remove/backup/restore` 辅助打包、安装、移除和备份恢复，backup 默认只备份代码，显式 `--with-data` 才备份插件自有表，remove 自动备份必须带数据。命令必须使用 `SourceOnlyCommand`，发布 Phar/SFX 内不出现这些命令，也不是运行时远程插件加载。插件前端变更后需要重新生成前端产物。
- 仓库与发布链仅使用 GitHub：`SmartAdminDeveloper` 为私有全量开发源，也是唯一生态维护入口；TAG Actions 自动同步 `SmartAdminLibrary`、`SmartAdminBuilder` 与 `SmartAdmin`；不得再配置其它代码托管地址或发布说明。普通用户只使用公开 `SmartAdmin` / `zoujingli/smartadmin` 运行和二次开发。
- Composer 包名统一使用横线规则：主包为 `zoujingli/smartadmin`，基础库为 `zoujingli/smart-admin-library`，构建器为 `zoujingli/smart-admin-builder`，插件为 `zoujingli/smart-plugin-xxx`；私有/商用插件不发布 Composer 远程包，只通过 `xadmin:plugin:*` ZIP 分发，ZIP 内 `composer.json` 仅服务本地 path autoload。
- AI 修改时优先复用现有基类和注册机制，不新增重复框架层。

## 分层标准

- `Controller` 只负责路由、参数读取、权限注解、操作日志注解和标准响应。
- `Service` 负责业务编排、事务、唯一性校验、跨 Mapper 调用和领域规则。
- `Mapper` 负责数据访问、查询条件、数据范围、分页、软删、状态变更和列表后处理。
- `Model` 只负责表结构映射、fillable、隐藏字段、关联、访问器和转换器。
- 标准 CRUD 优先继承 `CoreController`、`CoreService`、`CoreMapper`、`CoreModel`。
- `Library/System` 是基础插件，`SmartAdminBuilder` 是独立构建器 Composer 包；新业务能力优先放在 `plugin/<Business>`，不要把业务功能继续堆入 `System` 或 `web/apps/web-antd`。
- 应用插件推荐目录结构：`plugin/<Module>/src/Controller`、`src/Service`、`src/Mapper`、`src/Model`、`src/Support`、`src/Provider.php`，静态与资料目录可使用 `stc/view`、`stc/languages`、`stc/migrations`，并由 `plugin.view_root`、`plugin.language_root`、`plugin.migration_root` 显式启用。
- 不要把 SQL 查询写在 Controller；不要把 HTTP Request、Response 传入 Mapper。

## Controller 规则

- Controller 类使用 `final class XxxController extends CoreController`。
- 路由使用 Hyperf 注解：`#[Controller]`、`#[GetMapping]`、`#[PostMapping]`、`#[PutMapping]`、`#[DeleteMapping]`。
- 每个受保护接口必须有 `#[Auth]`，权限码要与菜单/节点对应，不能复用无关模块权限码。
- 写操作必须加 `#[Logger]`，敏感字段使用 `excludeFields`。
- 响应使用 `$this->success()`、`$this->error()`、`$this->respondFound()`、`$this->deleteByIds()`。
- 标准响应体固定为 `code/info/data/path`，`info` 是项目标准消息字段；自定义响应直接抛 `BaseResponseException`，不要在 Controller 中直接 `json()`。
- HTTP status 固定返回 `200`，标准业务码只允许写入 body.code：`200/401/403/404/500`。
- 统一响应由 `ResponseExceptionHandler` 处理，包含业务响应异常和 Hyperf 路由 404；不要再新增独立 404 响应处理器。
- `401` 仅表示 Token 缺失、过期、无效或刷新失败；账号密码错误、账号禁用等登录业务失败必须返回 `500`。
- `403` 仅表示 Token 有效但无操作权限；业务校验失败、状态不允许、数据不存在必须返回 `500`。
- `404` 仅表示页面或 API 路由不存在；业务数据、文件、父级记录不存在必须返回 `500`。
- 控制器只做轻量参数整理，复杂校验和业务逻辑下沉到 Service。

## Service 规则

- Service 类使用 `final class XxxService extends CoreService`。
- 标准 Service 通过构造函数注入 `protected XxxMapper $mapper`。
- 新增/更新数据通过 `create()`、`update()` 和 `filterData()` 做统一过滤。
- `filterData()` 是写入数据验证与过滤的标准入口；优先直接使用 `_vali($rules, $data)` 返回过滤后的数据，再追加唯一性、跨表、状态流转等业务校验。
- `_vali()` 只应在 Service 层或更底层业务支持类中使用；Controller 不做字段级业务验证，只做路由参数轻量整理和响应。
- 唯一性校验优先使用 `ensureUniqueField()`。
- 涉及多表写入必须显式事务处理。
- Swoole 下不要在单例 Service 属性中保存请求态、用户态、租户态、Token 用户模型上下文等可变状态；使用方法参数或协程 `Context`。

## Mapper 规则

- Mapper 类使用 `final class XxxMapper extends CoreMapper`。
- 标准 Mapper 通过构造函数声明 `protected string $model = XxxModel::class`。
- 列表查询优先使用 `getPageList()`、`getDataList()`、`makeQuery()`、`handleSearch()`、`handleListItems()`。
- `handleSearch()` 的常规筛选优先使用 `_query($query, $params)` 统一处理 `like/equal/in/dateBetween` 等白名单条件；特殊查询只在必要时手写，并显式说明范围边界。
- 读、改、删、恢复、启停默认走 `CoreMapper` 的数据范围保护方法，不要直接 `Model::find()` 后写入。
- 特殊原生查询必须显式处理数据范围；无用户上下文时必须 fail closed。
- 对外请求参数不能直接控制 select、orderBy、raw SQL、关联名或字段名；需要白名单。
- 查询别名和表达式必须在代码中注册，不能让请求直接传入表达式。

## Model 规则

- Model 类使用 `final class Xxx extends CoreModel`。
- 表字段必须写入 `$fillable`；敏感字段写入 `$hidden`。
- 有软删除字段时使用 `SoftDeletes`。
- JSON 字段使用 `_toJson()`、`_toArray()` 做统一转换。
- 有 `tenant_id` 且在 `$fillable` 中时会自动启用租户范围；不要绕过租户上下文做普通业务查询。
- 关联关系写在 Model，列表展示转换写在 Mapper 或 Service。

## 权限与数据范围

- 接口权限以 `#[Auth]`、`plugin.json` 菜单清单、权限节点同步为准。
- 新模块菜单通过 `plugin.json` 清单与 Registry 注册，避免写死在 System 中央配置。
- 数据权限默认 fail closed；拿不到用户上下文不能返回全量数据。
- 部门数据范围必须明确使用用户字段或部门字段，不能让 `deptField` 参数空转。
- 角色数据范围枚举必须与数据库、前端表单、后端校验保持一致。
- 多角色数据范围策略必须在代码和注释中保持一致，不能注释写“最严格”但实现取最宽。
- 用户选项、导出、统计、详情、更新、删除等非标准接口也必须套数据范围。

## 租户规则

- 登录后必须建立租户上下文，普通业务查询依赖 `TenantContext` 和 `CoreModel` 租户范围。
- 超级管理员逻辑要显式判断，不要让租户 ID 默认为 0 后误用平台数据。
- 新模块如支持租户，表必须包含 `tenant_id`，Model `$fillable` 必须包含该字段。

## 日志与敏感数据

- 操作日志和全局请求日志都必须脱敏。
- 操作日志包含三个主体：`change_data` 变更、`request_data` 请求、`response_data` 响应；列表和详情首屏以变更日志为主。
- 关键业务 Model 如需审计必须定义 `$logRules`，字段展示格式为 `字段中文名(字段代码)旧值改为新值`。
- 枚举字段必须配置 `values`，展示为 `映射文本(原始值)`；数量、金额、年龄、比例等字段必须配置 `unit`。
- 角色授权、用户角色/部门/岗位分配、公告接收人等关系变更不会稳定触发模型字段事件，必须手动追加变更记录。
- 脱敏规则要支持点路径，例如 `drivers.oss.access_secret`。
- 全局请求日志默认记录请求和响应 body 预览；大内容必须只读取并保存长度限制内的内容，超出部分用 `...` 截断，且必须脱敏。
- 密码、Token、Secret、Key、Cookie、Authorization、上传签名等不得明文入库或入日志。

## Release 数据库规则

- `migrations` 主要用于开发期建表和 fresh 初始化。
- 打包发布升级使用 DBAL 快照机制，不依赖迁移文件执行生产升级。
- 发布配置只允许使用 `config/autoload/release.php` 中的 `backup_tables` 和 `ignore_tables`。
- `backup_tables` 是发布包强制接管的数据表，升级前备份旧数据，升级时清空并用快照数据直接替换。
- `ignore_tables` 是发布系统完全不维护的表，不进入结构 diff、不备份、不恢复、不删除。
- `ignore_tables` 优先级高于 `backup_tables`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zoujingli/SmartAdmin](https://github.com/zoujingli/SmartAdmin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
