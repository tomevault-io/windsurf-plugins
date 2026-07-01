---
trigger: always_on
description: Abp RadzenUI 是一个基于 ABP Framework 和 Radzen Blazor 的 Blazor Server UI 主题与管理后台基础包。它的核心类库项目是 `src/Abp.Blazor.Server.RadzenUI`，NuGet 包名为 `AbpRadzen.Blazor.Server.UI`。
---

# AGENTS.md

## 项目概览

Abp RadzenUI 是一个基于 ABP Framework 和 Radzen Blazor 的 Blazor Server UI 主题与管理后台基础包。它的核心类库项目是 `src/Abp.Blazor.Server.RadzenUI`，NuGet 包名为 `AbpRadzen.Blazor.Server.UI`。

这个仓库不只是外观主题，还封装了 Blazor Server 运行入口、Radzen 服务注册、布局、菜单、本地化、认证跳转、通知/消息适配、设置页承载、头像上传、关联账户、数据字典、用户消息、审计日志、身份安全日志、租户和身份管理页面等能力。

当前解决方案主要面向 `.NET 10`，集中包版本管理在 `Directory.Packages.props` 中。关键版本包括：

- ABP：`10.4.0`
- Radzen.Blazor：`10.4.6`
- 目标框架：`net10.0`

## 解决方案结构

根目录重要文件：

- `Abp.RadzenUI.sln` / `Abp.RadzenUI.slnx`：解决方案文件。
- `Directory.Packages.props`：集中管理 NuGet 包版本。
- `Directory.Build.props`：统一构建属性。
- `README.md` / `README_zh-CN.md`：英文和中文使用说明。
- `docker-compose.yml`：示例运行相关的编排文件。
- `tasks/lessons.md`：维护过程中的经验记录，改 UI 或公共组件前建议先看。

源码目录：

- `src/Abp.Blazor.Server.RadzenUI`：核心 Blazor Server UI 主题包。
- `src/Abp.RadzenUI.Application.Contracts`：应用服务接口、DTO、权限定义。
- `src/Abp.RadzenUI.Application`：内置应用服务实现。
- `src/Abp.RadzenUI.Domain.Shared`：本地化、常量、错误码等共享定义。
- `src/Abp.RadzenUI.Domain`：领域实体，例如数据字典、用户消息。
- `src/Abp.RadzenUI.EntityFrameworkCore`：内置实体的 EF Core DbContext 与模型映射。
- `src/Abp.RadzenUI.LinkAccounts`：关联账户独立模块。

示例目录：

- `samples/CRM.*`：完整 CRM 示例应用，演示主题集成、菜单、产品 CRUD、数据库迁移、OpenIddict、Swagger、Redis、Aspire 等。

测试目录：

- `tests/Abp.RadzenUI.LinkAccounts.Tests`：关联账户会话栈、账号显示等回归测试。

## 核心 UI 包

核心项目：`src/Abp.Blazor.Server.RadzenUI/Abp.Blazor.Server.RadzenUI.csproj`

包信息：

- `PackageId`：`AbpRadzen.Blazor.Server.UI`
- `RootNamespace`：`Abp.RadzenUI`
- SDK：`Microsoft.NET.Sdk.Razor`

核心依赖：

- `Radzen.Blazor`
- `Volo.Abp.AspNetCore.Components.Server`
- `Volo.Abp.Autofac`
- `Volo.Abp.Identity.EntityFrameworkCore`
- `Volo.Abp.Account.Web.OpenIddict`
- `Volo.Abp.TenantManagement.*`
- `Volo.Abp.SettingManagement.*`
- `Volo.Abp.PermissionManagement.*`
- `Volo.Abp.AuditLogging.EntityFrameworkCore`
- 项目引用：Application、Application.Contracts、EntityFrameworkCore、LinkAccounts

## 模块注册

核心模块：`src/Abp.Blazor.Server.RadzenUI/AbpRadzenUIModule.cs`

它通过 `[DependsOn]` 接入：

- `AbpAutofacModule`
- `AbpRadzenUIApplicationModule`
- `AbpRadzenUIEntityFrameworkCoreModule`
- `AbpRadzenUILinkAccountsModule`
- ABP 审计、权限、设置、租户、身份、Blazor Components Web 模块

`ConfigureServices` 中的关键动作：

- 配置头像扩展属性。
- 为 `AbpRadzenUIResource` 增加 ABP UI、审计、设置管理等本地化基类。
- 重写 Cookie 认证跳转：未登录跳 `/account/login`，无权限跳 `/forbidden`。
- 注册 Razor Components 和 Interactive Server Components。
- 注册 Radzen 组件服务、查询字符串主题服务、级联认证状态。
- 替换 ABP 默认 `IUiMessageService` 和 `IUiNotificationService` 为 Radzen 实现。
- 注册全局样式和脚本 Bundle。
- 清空默认菜单贡献器，改用 RadzenUI 自己的菜单贡献器。
- 注册设置页 Contributor：Email、TimeZone、Account。
- 注册页面大小偏好、消息中心状态、菜单装饰状态、侧边弹窗、上传服务、关联账户登录管理等服务。

## 运行入口与路由

宿主项目需要在应用初始化管线末尾调用：

```csharp
app.UseRadzenUI();
```

实现位置：

- `src/Abp.Blazor.Server.RadzenUI/Extensions/ApplicationBuilderExtensions.cs`
- `src/Abp.Blazor.Server.RadzenUI/Extensions/EndpointRouteBuilderExtensions.cs`

`UseRadzenUI()` 会：

- 配置状态码跳转到 `/404`。
- 调用 `UseConfiguredEndpoints`。
- 映射 `MapRazorComponents<App>()`。
- 启用 `AddInteractiveServerRenderMode()`。
- 添加 `AbpRadzenUIOptions.RouterAdditionalAssemblies` 中的页面程序集。

路由组件：`src/Abp.Blazor.Server.RadzenUI/Components/Routes.razor`

- 使用 `AuthorizeRouteView`。
- 默认布局为 `MainLayout`。
- 未登录时导航到 `/account/login`。
- 页面程序集来自主题自身和 `RouterAdditionalAssemblies`。

## 主题配置

配置类：`src/Abp.Blazor.Server.RadzenUI/AbpRadzenUIOptions.cs`

主要配置项：

- `RouterAdditionalAssemblies`：宿主应用 Razor 页面程序集。接入业务页面时必须设置。
- `TitleBar`：标题栏设置，包括标题、GitHub 链接、语言菜单、侧边菜单多展开、面包屑。
- `LoginPage`：登录页标题和 Logo。
- `Theme`：默认主题和高级主题开关，默认主题为 `material-dark`。
- `ExternalLogin`：第三方登录图标配置。

示例项目在 `samples/CRM.Blazor.Web/CRMBlazorWebModule.cs` 中演示了完整配置方式。

## UI 外壳

主布局：`src/Abp.Blazor.Server.RadzenUI/Components/Layout/MainLayout.razor`

布局包含：

- `RadzenComponents`
- 顶部 `Header`
- 左侧 `MenuSiderbar`
- 右侧 `ThemeSiderbar`
- 右侧 `MessageSidebar`
- 移动端遮罩
- `CustomErrorBoundary`
- `SideDialogClickAway`

标题栏：`src/Abp.Blazor.Server.RadzenUI/Components/Layout/Header.razor`

标题栏能力：

- 菜单开关。
- 标题和面包屑。
- 刷新按钮。
- GitHub 链接。
- Radzen 外观切换。
- 主题侧栏开关。
- 消息中心按钮和未读数量。
- 全屏切换。
- 语言切换。
- 登录入口。
- 用户头像和个人菜单。
- 关联账户入口及返回上一个账号入口。

## 菜单系统

内置菜单贡献器在 `src/Abp.Blazor.Server.RadzenUI/Menus`：

- `DefaultRadzenMenuContributor`：创建 Administration 根菜单。
- `AbpIdentityMenuContributor`：身份管理菜单。
- `AbpTenantMenuContributor`：租户管理菜单。
- `AuditLoggingMenuContributor`：审计日志菜单。
- `IdentitySecurityLogMenuContributor`：身份安全日志菜单。
- `DataDictionaryMenuContributor`：数据字典菜单。
- `MessageMenuContributor`：消息菜单。
- `SettingManagementMenuContributor`：设置管理菜单。

菜单图标使用 Radzen/Material Symbols 风格的字符串图标名。菜单颜色扩展在 `Infrastructure/Navigation/ApplicationMenuItemIconColorExtensions.cs`。

示例业务菜单：`samples/CRM.Blazor.Web/Menus/CRMMenuContributor.cs`

要新增业务页面，通常需要：

1. 在宿主 Web 项目新增 Razor 页面。
2. 将页面程序集加入 `AbpRadzenUIOptions.RouterAdditionalAssemblies`。
3. 在业务 `IMenuContributor` 中添加菜单项。
4. 如有权限控制，菜单项调用 `RequirePermissions(...)`。

## 页面开发模式

通用 CRUD 页面基类：`src/Abp.Blazor.Server.RadzenUI/AbpCrudPageBase.cs`

它面向 ABP 的 `ICrudAppService`，提供：

- Radzen DataGrid 数据加载。
- 分页、排序、页大小偏好。
- 创建、编辑、删除弹窗流程。
- 权限检查：`CreatePolicyName`、`UpdatePolicyName`、`DeletePolicyName`。
- 成功通知和统一错误处理。
- DTO 与 ViewModel 映射。

页大小偏好通过 `GridPageSizePreferenceService` 和 Cookie 保存。支持的页大小为 `10、20、30、50、100`。

公共组件目录：


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShaoHans/Abp.RadzenUI](https://github.com/ShaoHans/Abp.RadzenUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
