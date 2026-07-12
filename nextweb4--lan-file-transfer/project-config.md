---
trigger: always_on
description: - `lan_transfer/api.py`：FastAPI 路由、游客/用户/管理员 session 鉴权、文件可见性过滤、文件搜索/排序、下载响应 header。
---

# AGENTS.md

## 1. 项目结构
- `lan_transfer/api.py`：FastAPI 路由、游客/用户/管理员 session 鉴权、文件可见性过滤、文件搜索/排序、下载响应 header。
- `lan_transfer/audit.py`：保存目录内 `.lan-transfer-audit.jsonl` 审计日志，记录登录、登出、上传、下载、删除、权限、账户和组变更；不得记录密码、session token、authorization、cookie、secret 类字段或文件内容。
- `lan_transfer/auth.py`：账户、批量创建账户、删除账户、用户组创建/删除、密码 PBKDF2 哈希、`.lan-transfer-auth.json`、默认 `admin / 12345678`、默认 `everyone` 组、内存 session、5 次错误后按 IP 锁定 3 小时。
- `lan_transfer/storage.py`：二进制流保存、`.part` 临时文件、大小校验、SHA-256、manifest、owner、allowed_groups、旧 `default` 组迁移、用户删除后的文件转主、zip entry mtime。
- `lan_transfer/security.py`：文件名、相对路径、manifest 已存保存路径和根目录控制文件保留名安全处理；新增保存路径逻辑必须经过这里。
- `lan_transfer/server.py`：本地 uvicorn 线程生命周期；启动超时和停止失败必须显式报错，不能静默留下旧服务线程。
- `lan_transfer/config.py`：保存目录、端口和 LAN URL；LAN IP 优先使用 UDP 路由选出的非 loopback / 非 link-local 私有地址。
- `lan_transfer/logging_config.py`：保存目录内日志 handler 切换；必须先创建新 handler 成功，再替换并关闭旧 handler。
- `lan_transfer/desktop.py`：Tkinter Windows 窗口、用户/管理员 URL、二维码、服务启停、保存目录选择。
- `lan_transfer/static/user.html` 与 `user.js`：默认中文游客页，内置登录、中英文切换、文件搜索/排序；登录用户可上传并删除自己上传的文件。
- `lan_transfer/static/admin.html` 与 `admin.js`：默认中文管理员页，右上角工具/管理菜单、批量创建账户、可搜索组选择器、文件可见组、审计日志、上传、删除、修改密码。
- `tests/`：pytest 覆盖账户权限、批量创建、用户组可见性、搜索/排序、上传完整性、mtime、下载 header、zip、路径安全、同名文件、windowed exe 日志和发布元数据一致性。
- `docs/OPEN_SOURCE_AUDIT.md`：外部方案审计、许可证和冲突检查。
- `LICENSE`：MIT License，作者固定为 HaoXiang Huang。
- `RELEASE_NOTES.md`：GitHub Release 双语说明，发布前必须同步版本号、文件用途和作者信息。
- `version_info.txt`：PyInstaller Windows exe 版本资源，必须保持公司名、产品名、版权、版本和主页正确。
- `scripts/build_exe.ps1` 与 `LANFileTransfer.spec`：Windows exe 打包入口；脚本会运行 pytest、PyInstaller，并生成 `release-assets/` 下的 exe、zip 和 `SHA256SUMS.txt`。

## 2. 运行命令
- 首次准备：`py -m venv .venv`
- 安装依赖：`.\.venv\Scripts\python.exe -m pip install -r requirements.txt`
- 本地运行：`.\.venv\Scripts\python.exe -m lan_transfer.desktop`

## 3. 测试命令
- 运行测试：`.\.venv\Scripts\python.exe -m pytest`
- 修改 `api.py`、`auth.py`、`storage.py`、`security.py`、`static/user.js` 或 `static/admin.js` 的上传/下载/权限协议后，必须运行 pytest。

## 4. 构建命令
- 打包 exe：`powershell -ExecutionPolicy Bypass -File .\scripts\build_exe.ps1`
- 产物路径：`dist\LANFileTransfer.exe`
- 发布产物路径：`release-assets\LANFileTransfer.exe`、`release-assets\lan-file-transfer-v1.0.0-windows.zip`、`release-assets\SHA256SUMS.txt`

## 5. 代码风格
- Python 代码优先使用标准库；当前第三方依赖只用于 HTTP、ASGI、multipart、二维码、Pillow 图像显示和 PyInstaller 打包。
- 前端保持原生 HTML/CSS/JS，不引入 CDN、云 SDK 或运行期外部网络资源。
- 发布署名必须统一为 Author `HaoXiang Huang`、Email `didadida1688@gmail.com`、Homepage `https://nextweb4.github.io/`、GitHub `https://github.com/NextWeb4`；README、LICENSE、Release Notes、网页页脚、桌面 About/页脚和 exe metadata 都必须一致。
- Web UI 使用当前暗色工具型界面；移动端下载、删除、上传按钮必须在卡片或工具区内可见，不依赖横向滚动表格。
- 前端字号必须使用稳定尺寸或媒体查询断点，不使用 `vw`/`vh` 驱动 `font-size`。
- 前端背景保持低噪声工具界面，不使用装饰性 `radial-gradient`、orb 或 bokeh 背景。
- 当前未发现 lint / format 命令；新增 lint / format 后需同步到本文件和 README。

## 6. 模块边界
- `auth.py` 只处理账户、密码、用户组、session 和 IP 锁定，不处理文件内容。
- `auth.py` 中账户/组/密码变更写 `.lan-transfer-auth.json` 失败时，必须回滚内存态，不能让当前进程与磁盘配置分叉。
- `auth.py` 写 `.lan-transfer-auth.json` 必须使用保存目录内用户上传不可到达的隐藏随机临时文件，写入、序列化或替换失败时必须删除临时文件，不能退回固定 `.lan-transfer-auth.json.tmp`。
- `auth.py` 中登录必须先校验密码，再暴露账户禁用状态；禁用账户的错误密码仍要计入 IP 失败次数和锁定。
- `auth.py` 中强审计 auth 变更必须通过 `state_transaction()` 覆盖 snapshot、写入、审计和回滚窗口，避免审计失败回滚覆盖并发成功变更；同时需要碰 storage 的跨模块操作必须按 storage lock -> auth lock 顺序，避免和上传提交路径死锁。
- `auth.py` 中 `restore_state()` 只在 settings 真的变化时才需要写盘；写回快照失败时，必须把内存恢复到调用前状态，不能让进程内 auth 状态与磁盘 settings 分叉。
- `auth.py` 中普通用户改密、管理员重置用户密码、禁用用户和变更用户 `role` 都必须在写盘成功后失效该用户既有 session；如果后续审计失败，API 必须通过快照恢复旧密码和旧 session。
- `storage.py` 只处理磁盘、manifest、hash、mtime、zip，不读取 FastAPI `Request`。
- `storage.py` 中需要审计确认后才算完成的删除，必须使用 prepare / audit / commit 或失败 rollback，不能在审计前不可逆删除文件。
- `storage.py` 中上传在审计成功前必须保持 `audit_status="pending"` 并对列表、下载和打包不可见；发布为 `complete` 和写上传审计必须处于同一个 `manifest_transaction()`，锁释放前审计必须成功，发布失败不能写成功审计。
- `storage.py` 中文件权限变更必须通过 `manifest_transaction()` 覆盖 manifest 更新、审计和回滚窗口，避免审计成功前并发请求看到新可见性。
- `storage.py` 删除 rollback 中如果原文件已经从 tombstone 恢复，即使 manifest 写回失败，也必须保留内存 manifest 记录，避免当前进程丢失已恢复文件。
- `storage.py` 中 `.delete` tombstone 代表待提交或待回滚删除；同名新上传选择保存路径时必须把 tombstone 视为占用，不能复用原 `saved_relative_path`。
- `storage.py` 写 `manifest.json` 必须使用保存目录内用户上传不可到达的隐藏随机临时文件，不能使用固定 `manifest.json.tmp`。
- `storage.py` 从 manifest 读取 `saved_relative_path` 后必须重新经过 `security.py` 校验，禁止旧/坏 manifest 指向根部控制文件。
- `storage.py` 的保存路径占用判断必须按 Windows 文件系统语义大小写不敏感比较 manifest 路径和 `.delete` tombstone；zip entry 去重也必须避免大小写不同但 Windows 解压会冲突的名称。
- `api.py` 只编排 HTTP 参数、鉴权、文件可见性过滤、响应 header 和错误码，不自行拼接不安全文件路径。
- `api.py` 中登录、登出、密码、账户、组、文件删除等强审计变更，审计写入失败时必须回滚本次 session/auth/storage 变更。
- `api.py` 必须覆盖 FastAPI 请求校验错误响应，不能把无效请求体中的密码、session token 或内部结构通过默认 422 `input` 字段回显给客户端。
- `api.py` 的文件列表排序必须容忍旧/坏 manifest 中非数字的 `file_size` / `server_mtime`，按 0 参与排序，不能让 `/api/files` 变成 500。
- `api.py` 的用户页原生下载 scope 只能收窄权限：`scope=guest` 必须按游客处理，`scope=user` 遇到管理员 cookie 时也必须收敛到游客，不能扩大为管理员下载。
- `api.py` 的用户端身份接口 `/api/login`、`/api/session`、`/api/logout`、`/api/password` 只接受普通 `user` 角色或游客清理无效 session；管理员必须使用 `/api/admin/*` 对应接口，避免同源 cookie 让用户页继承管理员权限。
- `api.py` 的管理员身份接口必须拒绝已认证的非管理员；`/api/admin/logout` 可以让游客清理无效本地 cookie，但不能删除普通用户 session 或产生未审计登出。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NextWeb4/lan-file-transfer](https://github.com/NextWeb4/lan-file-transfer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
