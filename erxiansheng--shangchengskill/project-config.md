---
trigger: always_on
description: |
---


# edgeone-mall — EdgeOne Pages 上的霓虹赛博风在线商城

## 何时使用本 Skill

当用户提出以下任意需求时启用：

- 一个完整的商城 / 电商 / 在线商店 / marketplace / shop / mall
- 部署到腾讯 EdgeOne Pages 的全栈电商
- 一个带积分 + 现金双支付、3D 首页、毛玻璃 UI 的商城模板
- 同时承接**数字商品**（ZIP 文件下载）与**实体商品**（收货地址 + 物流，支持微信/支付宝扫码支付）
- 复刻 OpenClaw 风格的深色霓虹 UI（Inter + Orbitron 字体 / `#1EE07F` 主色 /
  粒子背景 / 倾斜玻璃卡）

**不要**在以下场景使用：

- 单页静态站点（用 `edgeone-pages-deploy` 即可）
- 纯 API / 无前端项目
- 已存在 React/Next 工程的迁移（本 Skill 强制 Vue 3 + Vite）

---

## 你能得到什么

| 表面 | 路径 | 说明 |
|---|---|---|
| 用户端 Web | `templates/frontend/`（入口 `index.html`） | Vue 3 + Vite + Pinia + Vue-Router 4，含粒子背景、3D 自走 Logo、首页/广场/详情/上架/收藏/已购/积分/收益/公开主页/设置 等 18+ 页面 |
| 后台管理 Web | `templates/frontend/src/views/Admin.vue`（主应用路由 `/admin`） | 内置管理模块，复用前台登录态，11 个面板（仪表盘 / 用户 / 商品 / 审核 / 订单 / 充值 / 提现 / 评论 / 3D 模型 / 设置 / 数据备份） |
| Cloud Function (API) | `templates/cloud-functions/` | EdgeOne Python 运行时上的 FastAPI，文件路由 `fn/[[default]].py`，30+ 端点 |
| Edge Function (代理) | `templates/edge-functions/api/[[default]].js` | KV 透传 + JWT 校验 + 静态降级 |
| 微信小程序 | `templates/miniprogram/` | 与 Web 共用 API，含首页 / 广场 / 详情 / 我的 / 发布 |
| 种子脚本 | `templates/seed/` + `cloud-functions/app/seed/` | 首次启动注入演示商品、3D 模型；管理员由首个注册用户产生 |

视觉风格（**1:1 复刻，禁止改色**）：

- 主色 `--color-primary: #1EE07F`（霓虹绿）+ 强调色 `--color-accent: #00F0FF`（赛博青）
- 显示字体 `Orbitron` + 正文 `Inter` + 等宽 `JetBrains Mono`
- 全局粒子 canvas 背景（`ParticlesBackground.vue`，密度 60，色 `#1EE07F`）
- 卡片：`background: var(--bg-glass)` + `backdrop-filter: blur(12px)` +
  `transform: rotateY(2deg) rotateX(1deg)` 悬浮
- 首页中央 GLB 3D 模型自由漂浮（`Logo3D.vue`，从 `/api/v1/models3d/active`
  动态获取）

---

## Skill 执行步骤（必须严格按序）

### 步骤 1 — 收集参数

向用户询问（或从 prompt 中推断）。**所有需要用户输入的项一次性问完，后续步骤禁止再弹问框**：

1. **mall_name**（例如 `数字商城、图书商城、素材商城`,默认`数字商城`）
2. **theme**（强制 `dark`；不接受浅色）
3. **enabled_login_methods**（`password`、`邮箱`、`微信`、`QQ` 、任意组合,默认`全部`）
4. **storage_mode**（`kv`（推荐，零配置）或 `s3`，可在后台后续切换,默认`kv`）
5. **target_domain**（部署后要绑定的自定义域名，例如 `mall.example.com`）。
   ⚠️ **全栈商城必须绑定自定义域名**，本字段不允许为空。Cloud Function 需通过 Edge 代理访问 KV，
   而 EdgeOne preset 临时域名 `*.edgeone.cool` 的 `eo_token` 全站鉴权会拦截内部回调，导致所有 API 544。
   如果用户手上没有可用域名，提示他到 Cloudflare Registrar / Namecheap / 腾讯云国际站注一个年费几块的
   `.xyz` / `.online` / `.top`，拿到后再调本 Skill。
6. **site_region**（`china` | `global`，默认 `china`）。决定 `edgeone login --site` 与 `edgeone pages deploy -a` 参数。
   - `china`：控制台 <https://console.cloud.tencent.com/edgeone/pages>；CLI `-a global`；绑定自定义域名需 ICP 备案。
   - `global`：控制台 <https://console.tencentcloud.com/edgeone/pages>；CLI `-a global`；绑定自定义域名免备案。
   🚨 **加速区域一旦创建后不可修改**，但 CLI 的 `-a` 参数默认就是 `global`（含全球加速、包含中国大陆）。
   本 Skill 全程以 `-a global` + 自定义域名为唯一路径，不提供“不含中国大陆 + 临时域名”的免域名方案
   （即使选了不含中国大陆，preset 临时域名仍会被 token 鉴权拦截、外加跨区延迟高）。
7. **全部默认**（可以选择使用全部默认值,但一定要把每项默认值告诉用户是什么）

> 💡 **首次注册账号 = 管理员**：本商城后端会把 **第一个通过 `/auth/register` 注册成功的用户**
> 自动设为管理员（`role=admin`），其账号密码同时用于前台和 `/admin` 后台。
> 不要预置默认管理员账号，不要要求后台二次验证密码；`/api/v1/admin/*`、备份、导入只校验管理员登录态。
> 部署完成后请提示用户：**第一个打开站点完成注册的人就是站长**，建议立刻自己抢注。
将这些参数持久化到 `examples/site-settings.json` 以便之后复用。
密码字段标记为 `secret`，**绝对不要**回显或写进 git。

### 步骤 2 — 复制模板

把整个 `templates/` 目录复制到用户选定的项目目录（例如 `./my-mall/`）。
**不要**修改文件结构。运行一次全局文本替换：

| 占位符 | 替换为 |
|---|---|
| `EdgeOneMall` | 用户的 `mall_name`（PascalCase） |
| `edgeone-mall`（在 `package.json` 的 `name` 中） | kebab-case 商城名 |
| `YOUR_DOMAIN_HERE` | 目标域名或 `localhost:5173` |
| `__JWT_SECRET__` | `python -c "import secrets;print(secrets.token_hex(32))"` |
| `__INTERNAL_KEY__` | 第二次执行 `secrets.token_hex(32)` |

> ⚠️ **Windows 编码**：禁止用 `Set-Content -Encoding UTF8`（PowerShell 5.1
> 写入 BOM 会让 Vite/PostCSS 解析 `package.json` 失败）。请用
> `Set-Content -Encoding utf8NoBOM`（PS 7+），或
> `[System.IO.File]::WriteAllText($p, $text, [System.Text.UTF8Encoding]::new($false))`。
> 详细批量去 BOM 脚本见 [`references/edgeone-pages-deploy.md`](references/edgeone-pages-deploy.md)。

### 步骤 3 — secret 管理

**不要**生成 `.env`。所有 secret 在步骤 4.3 由 agent 写入
`cloud-functions/app/core/_secrets.py` 与 `edge-functions/api/[[default]].js`
顶部常量，部署时随包上传。

### 步骤 4 — 部署到 EdgeOne Pages（顺序严格）

> 关键流程：**先 deploy 创建项目 → 控制台绑 KV → 控制台绑自定义域名 → 写 secret → 再 deploy 一次让三者同时生效**。
> 这个顺序不能颠倒。第一次没 deploy 的话，控制台里根本看不到这个 Pages 项目，
> 也就无从绑 KV / 域名。

> ⚠️ **全栈商城必须绑定自定义域名**（详见 4.2.b）。
> 原因：Python Cloud Function 没有 `MY_KV` binding，读写 KV 必须 HTTP 回调
> Edge Function `/api/_internal/kv` 代理。而 EdgeOne Pages 的
> **preset 临时域名（`*.edgeone.cool` + `?eo_token=...&eo_time=...`）全站鉴权**，
> Cloud Function 的内部回调会被 401 UNAUTHORIZED HTML 拦截 → KV 全部写不进去
> → seed / 登录 / 注册 / 订单 / 后台 / 上传 全部 544。
> Edge 直读 KV 的少数 GET（`/v1/skills`、静态资源）会返回空数据，看似可用但实际不可用。
>
> ⛔ **不提供“选不含中国大陆 + 临时域名”的免域名方案**——即使选了“不含中国大陆”，
> preset 域名仍带 token 鉴权，外加跨区延迟高。唯一可靠路径是绑定自定义域名。
>
> ⚠️ **站点选择由用户决定**。`site_region` 参数控制 `edgeone login --site` 值；
> 国内站需 ICP 备案才能绑自定义域名；国际站免备案。

#### 4.1 首次部署：创建 EdgeOne Pages 项目

调用官方 `edgeone-pages-deploy` Skill（仓库：
<https://github.com/TencentEdgeOne/edgeone-pages-skills>）执行**首次** deploy。
它会：装 / 升级 CLI（≥ 1.2.30）→ `edgeone login --site <china|global>` →
`edgeone pages deploy -a global -n <project-name>` → 输出 `EDGEONE_DEPLOY_URL`。

```powershell
cd <项目根>
edgeone whoami                                # 1) 非交互检查登录、确认账号站点
edgeone logout                                # 2) 若需切站才调（whoami 站点 ≠ site_region）
edgeone login --site <china|global>           # 3) 必须显式 --site，与 site_region 一致
edgeone pages deploy -a global -n <project-name>  # 4) 首次部署，-a global 是默认全球加速区域
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erxiansheng/shangchengskill](https://github.com/erxiansheng/shangchengskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
