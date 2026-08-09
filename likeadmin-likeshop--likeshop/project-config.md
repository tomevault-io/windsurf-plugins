---
trigger: always_on
description: 本文件适用于整个仓库，是 AI 编码代理和开发者理解本项目时的首要架构说明。`CLAUDE.md` 是面向 Claude 类工具的同步摘要；如果目录、技术栈、接口约定、构建发布方式或强约束发生变化，必须同时更新这两个文件。
---

# AGENTS.md

## 文档定位

本文件适用于整个仓库，是 AI 编码代理和开发者理解本项目时的首要架构说明。`CLAUDE.md` 是面向 Claude 类工具的同步摘要；如果目录、技术栈、接口约定、构建发布方式或强约束发生变化，必须同时更新这两个文件。

事实优先级：实际代码与配置 > 本文件 > `AI_GUIDE.md` / `README.md`。后两者包含历史版本和商业版说明，若与当前仓库不一致，以实际代码为准。

## 项目定位与当前基线

Likeshop 是单商户 B2C 商城。当前仓库是免费企业版 `v3.5.1`，包含一个 ThinkPHP 单体服务端、服务端渲染的 PC 管理后台，以及一个 Vue 2 uni-app 移动商城源码。

- 服务端：PHP `>= 7.0`、ThinkPHP `5.1.*`、MySQL，Composer 管理依赖。
- 管理后台：admin 模块服务端渲染，Think 模板 + Layui/Layui Admin + jQuery；不是 Vue/React 独立工程。
- 移动商城：Vue 2 + uni-app + Vuex + 本地 uView 组件，JavaScript 项目；主要通过 HBuilderX 构建。
- PC 商城：最新 `php-b2c` Nuxt 2/Vue 2 源码已迁入 `pc/`；构建产物按发布步骤输出到 `server/public/pc/`，该目录未必存在于源码工作区。
- 基础设施：Nginx、PHP-FPM、MySQL、Redis；`docker/` 提供旧版 Compose 开发配置。
- 自动化测试：当前业务代码没有可用的 PHPUnit/Jest/Vitest 测试套件，也没有统一 lint/type-check 脚本。

不要把参考项目 `php-amdj` 的 Vue 3、TypeScript、Vite、ThinkPHP 8 或多 uni-app 目录约定套用到本仓库。

## 仓库目录职责

- `server/`：唯一服务端工程，同时包含 API、管理后台源码和已部署静态资源。
- `server/application/admin/`：PC 管理后台的 controller、logic、validate、model、server、view、middleware 等。
- `server/application/api/`：移动商城 JSON API 的 controller、logic、validate、cache、middleware 等。
- `server/application/common/`：跨 admin/api 共用的 model、logic、server、cache、validate、middleware。
- `server/application/index/`：站点根入口；按设备尝试返回 PC 或 mobile 的 `index.html`。
- `server/config/`：ThinkPHP 全局配置。
- `server/route/route.php`：H5、PC、定时任务等显式路由。
- `server/public/`：Nginx Web Root、PHP 入口、静态资源及安装资源。
- `server/public/static/`：后台 Layui、插件和公共前端资源。
- `server/public/mobile/`：uni-app H5 的已编译产物，属于发布结果，不是移动端源码。
- `server/public/install/db/like.sql`：新安装环境的数据库结构和初始化数据，默认表前缀为 `ls_`。
- `server/extend/`：非 Composer 扩展代码。
- `server/thinkphp/`、`server/vendor/`：框架和第三方依赖，除非任务明确要求依赖升级，否则不要直接修改。
- `server/runtime/`：运行缓存和日志；不要把运行产物纳入功能修改。
- `server/public/uploads/`：用户上传数据；不要清理、覆盖或提交真实业务文件。
- `uniapp/`：mobile/H5/微信小程序/App 的唯一可编辑源码。
- `uniapp/pages/`：主包页面。
- `uniapp/bundle/pages/`：分包业务页面。
- `uniapp/components/`：业务组件和仓库内置的 `uview-ui`。
- `uniapp/api/`：按业务域拆分的 API 调用封装。
- `uniapp/utils/request.js`：统一请求、token 注入和响应处理。
- `uniapp/store/`：Vuex store。
- `uniapp/config/`：API 地址、版本号和缓存键等运行配置。
- `uniapp/manifest.json`、`uniapp/pages.json`：平台能力、H5 路由、页面与分包注册的权威配置。
- `uniapp/unpackage/`：HBuilderX 构建输出；如本地生成，视为临时产物。
- `docker/`：旧版容器编排和 Nginx/PHP/MySQL 配置。
- `doc/`：安装相关补充资料。

## 整体请求架构

### 公共入口

Nginx 的站点根目录应指向 `server/public/`。请求经 `server/public/index.php` 进入 ThinkPHP；未检测到 `server/config/install.lock` 时会跳转安装程序。

ThinkPHP 开启多模块但未强制路由，因此主要 URL 由“模块/控制器/操作”自动解析：

- `/admin/...`：管理后台 HTML 或同路由 AJAX。
- `/api/<controller>/<action>`：移动端 JSON API。
- `/mobile/...`：返回 `server/public/mobile/index.html`，供 H5 history 路由使用。
- `/pc/...`：返回 `server/public/pc/index.html`；深层 history 路由由 `route/route.php` 的 `.*` 规则兜底。
- `/crontab`：触发 ThinkPHP 的 `crontab` 控制台命令。

根路径由 `application/index/controller/Index.php` 按终端类型选择 PC 或 mobile 构建产物。改入口、伪静态、H5 history 或站点子目录时，要一起检查 Nginx、`route/route.php`、`manifest.json` 和 `config/app.js`。

### 移动端到后端的数据流

1. 页面或组件调用 `uniapp/api/*.js` 的业务函数。
2. API 封装通过 `uniapp/utils/request.js` 请求 `${baseURL}/api/`。
3. 请求拦截器清理 `null`、`undefined`、空字符串，并注入登录 `token`；登录请求还会获取并附带短时 `X-Consume-Token` 票据。
4. `application/api/http/middleware/Login.php` 处理跨域、免登录方法和 token 校验；消费票据功能开启时，它会继续校验需登录接口的 `X-Consume-Token`，再将用户信息写入 request。
5. API controller 继承 `ApiBase`，读取参数和分页信息，调用 validate/logic。
6. logic 调用 `application/common` 下的模型、公共 logic/server 或第三方集成。
7. controller 用 `_success()` / `_error()` 返回统一 JSON。

统一响应字段为 `code`、`msg`、`data`、`show`、`time`：`code = 1` 表示成功，`code = 0` 通常表示业务失败，`code = -1` 表示登录态失效。修改该契约会同时影响所有移动端页面，不能在单个接口随意另造格式。

分页参数统一沿用 `page_no`、`page_size`，基类默认每页 15 条并将最大值限制为 100。

## 服务端架构与开发约定

### 技术与分层

- PHP 目标基线以 Composer 和 Docker 配置为准：PHP 7.x，ThinkPHP 5.1。不要在普通功能开发中使用 PHP 8 专属语法。
- PSR-4 根命名空间是 `app\`，对应 `server/application/`。
- 现有业务分层通常为 `controller -> validate/logic -> model/server`。
- `controller` 负责 HTTP 参数、登录上下文、校验调用和响应，不应堆积复杂业务规则。
- `logic` 负责业务编排、查询组装和事务边界。
- `model` 负责持久化及模型关联。跨端共享模型优先放 `application/common/model/`。
- 本项目历史命名使用 `server` 表示公共服务类，例如支付、微信、短信、文件和配置服务；新增代码应先沿用邻近模块的命名，不要擅自批量改成 `service`。
- 全局通用函数位于 `server/application/common.php`。新增前先检索 `common.php`、common logic/server 和现有模型，避免重复实现。

### API 模块

- controller 继承 `app\api\controller\ApiBase`。
- 免登录 action 写入 controller 的 `$like_not_need_login`，值按 action 名小写比较。
- 登录态通过 `token` header 传递；不要另加不兼容的认证头而不更新统一请求层。
- 输入校验优先复用/新增 `application/api/validate/` 中的 Validate 类。
- 返回必须使用基类 `_success()` / `_error()`，保持 `code/msg/data/show/time` 结构。
- API controller/action 名就是前端调用路径的一部分。重命名时必须同步搜索 `uniapp/api/` 和页面调用方。
- `GET /api/account/captcha` 返回一次性图形验证码；`sms/send`、`user/send` 以及账号密码 `account/login` 都必须携带 `captcha_key` 和 `captcha`，验证码由 `application/common/server/CaptchaService.php` 消费且只能使用一次。
- admin 登录页通过 `admin/account/captcha` 获取同一类一次性图形验证码；后台账号登录也必须提交 `captcha_key` 和 `captcha`。
- 密码传输统一使用 `account/passwordKey` 获取短时公钥（API 和 admin 各自模块路径）；密码字段以 `RSA:` 前缀密文提交，并携带 `password_key_id`。服务端在 API/Admin 的 Login middleware 中解密后再进入 controller 校验和业务逻辑，密钥默认 120 秒且只消费一次。新增密码字段必须加入 `PasswordCryptoService` 及三端公共请求层的字段白名单。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [likeadmin-likeshop/likeshop](https://github.com/likeadmin-likeshop/likeshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
