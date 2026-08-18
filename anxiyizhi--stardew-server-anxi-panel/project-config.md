---
trigger: always_on
description: - 本项目所有功能、修复、文档、测试和发布工作只能直接在本地 `main` 上进行，并同步到远端 `origin/main`。
---

# AGENTS.md

## Git 单主线硬规则

- 本项目所有功能、修复、文档、测试和发布工作只能直接在本地 `main` 上进行，并同步到远端 `origin/main`。
- 禁止代理自行创建、切换或保留任何非 `main` Git 分支，也禁止为任务或发布自行创建额外 Git worktree。除非用户在当次请求中明确要求特定分支，否则不得使用功能分支、发布分支、临时分支或 PR 分支。
- 发布 tag 必须从已与 `origin/main` 完全同步、工作树干净且通过全部发布门禁的 `main` 提交创建；不得从其它分支创建 tag，也不得移动已有 tag。
- 发现历史非 `main` 分支或 worktree 时，先确认其有效提交已进入 `main`，再删除对应 worktree 和本地/远端分支；不得因清理分支丢失未合并提交或未提交文件。

## 工作开始前

每次工作开始前先阅读 `docs/01-project-overview.md` 和 `.agents/error-notebook.md`，再按任务范围阅读：

- 后端任务：`docs/02-backend.md` 和 `docs/backend-handoff/` 下最新的后端接手文档。
- 前端任务：`docs/03-frontend.md` 和 `docs/frontend-handoff/` 下最新的前端接手文档。
- 前后端联调任务：`docs/06-integration.md`。
- 路线、排期、已完成状态：`docs/08-future-roadmap.md`。
- Docker 镜像、部署、发布：`docs/09-image-build.md`。

修改或新建功能时，优先寻找 Junimo 容器已有能力进行通信，不要绕过 `stardew_junimo` driver 直接在 API 层堆 Stardew 逻辑。

## 项目接手文档规则

每完成或修改一个功能，都必须同步更新对应长期文档：

- 后端功能：更新 `docs/02-backend.md`、最新后端接手文档，并在 `docs/08-future-roadmap.md` 标记状态变化。
- 前端功能：更新 `docs/03-frontend.md`、最新前端接手文档，并在 `docs/08-future-roadmap.md` 标记状态变化。
- 跨端接口或联调：更新 `docs/06-integration.md`。
- 镜像、部署、发布流程：更新 `docs/09-image-build.md`。
- 后期暂缓事项：更新 `docs/07-later-optimizations.md`。

接手文档至少记录：改了什么、影响哪些接口/文件、如何验证、下一步注意事项。不要只更新 README；README 面向使用者，接手文档面向下一位维护者。

## 正式版本与 Tag 发布硬门禁

任何创建、移动或推送 `v*` tag，更新 `latest`，推送正式镜像，或创建 GitHub Release 的工作，都必须先完成以下门禁。目标是用一次不可变候选证明“新旧功能没有回归、上一正式版能从 Web 一键升级”，不再为未变化链路机械重复长门禁。

1. 发布前在 `docs/09-image-build.md` 写出本版变更清单、受影响链路和本版专项矩阵。专项矩阵只覆盖本版新增或修改功能的正常路径、关键边界、权限安全、幂等/恢复、数据完整性与资源清理；与本版无关的历史故障不重复手工注入，由既有自动化回归负责。
2. 正式候选只能从工作树干净、与 `origin/main` 完全同步的本地 `main` 构建，版本、完整 commit 和 UTC build date 必须固定。影响镜像或正式部署资产的提交推送到 `main` 后，由 `.github/workflows/release-candidate.yml` 自动取最新正式版并递增补丁版本；手动 dispatch 只用于明确覆盖 major/minor 版本或受控重试。Windows Docker Desktop 本机复现使用 `pwsh -NoLogo -NoProfile -File scripts/release-candidate.ps1`，Linux/CI 使用 `scripts/release-candidate.sh`。所有测试使用任务专属 Compose project、容器、网络、端口、bind 和 volume，不得使用生产数据或长期凭据。
3. 同一版本和 commit 只允许一个待发布候选身份。候选完成测试后以 `candidate-<version>-<sha12>` 推到 GHCR，并把精确 digest、版本、commit、build date、上一正式版和候选 workflow run 写入不可变候选证明 artifact。任何会改变镜像内容、运行契约或测试输入的提交都会使旧证明失效；纯 tag、Release 说明和发布后证据不得触发重建。
4. 候选代码门禁始终包含后端 test/vet/build、前端状态回归与 production build、脚本测试/ShellCheck、兼容清单、updater/Docker integration。网站只在 `website/**`、公开文档或 README 变化时构建；SMAPI 真实下载、远程制品和 Junimo 长 integration 只在运行栈清单或对应实现变化时执行。跳过必须由 `scripts/run-release-gates.sh` 基于上一 tag 到候选 SHA 的路径差异自动判定，不能凭口头判断。
5. 每个候选必须完成全新安装、`/health`、`/api/version`、未初始化状态和 Panel 重启冒烟；本版新增或修改功能至少有一条真实 Docker E2E，并在升级得到的新 Panel 上再次验收。未变化旧功能由全量自动回归覆盖，升级后再抽验初始化/用户、实例、存档/Mod/备份等受影响的核心状态。
6. 必须从“当前上一正式版”通过真实 Panel Web API 完成更新检查、dry-run、管理员确认、apply、预期断线重连和终态恢复，验证 SQLite、初始化状态、任务相关长期数据、非目标游戏容器/volume 及 Panel 重启后的状态。必须对同一候选引用注入一次 unhealthy 目标，确认 `failed_rolled_back/health_check_failed` 和旧版恢复。不能用直接调用 updater、手工改 Compose 或仅 `docker compose up` 替代这条验收。
7. 默认不再每版测试更老版本。只有本版改变数据库迁移、部署格式、运行栈、长期数据结构或跨版本兼容逻辑时，才增加“受影响的最老支持版本 → 候选”一条代表升级；版本边界必须来自迁移代码和长期文档。Control/SMAPI/Junimo/网站未变化时不重复对应真实长链。
8. Tag 必须是 annotated tag，指向候选证明中的完整 commit，且该 commit 必须仍精确等于 `origin/main`。候选 workflow 成功后，`.github/workflows/release-after-candidate.yml` 只有在该 commit 仍是最新 `origin/main` 时才自动创建 tag 并通过 `workflow_dispatch` 启动正式提升；被后续提交取代的候选只记录 superseded，不发布。`.github/workflows/release.yml` 只能提升候选证明里的精确 digest 到三仓正式版本和 `latest`，禁止重新 build；三仓六引用 digest 与 OCI 元数据必须一致。digest 一致时只从一个正式仓库回拉做 health/version 冒烟，不再启动三份相同镜像。
9. 候选 workflow、任一必跑门禁、真实 Web 升级、unhealthy 回滚、digest 提升或正式冒烟失败时停止发布。只重跑失败及其受影响的下游步骤；候选输入未变化时不重复已经成功的独立门禁。高风险必测场景受外部条件阻塞时仍不得降级为“发布后观察”。
10. 发布完成后把候选 workflow ID、正式 release workflow ID、唯一 digest、选择/跳过的矩阵、实际耗时、故障和资源清理结果写入 `docs/09-image-build.md`、对应接手文档和路线图。发布后证据提交不得移动既有 tag，也不得使已经发布的同 digest 候选重新走门禁。

如果任何高风险场景因外部条件无法验证，停止发布并向用户说明阻塞项；不得擅自降低为“发布后再观察”。

## 命令执行与编码错题本

项目级错题本固定为 `.agents/error-notebook.md`。它是每次工作的必读输入，不是任务结束时可选的复盘。

- 命令、Shell、工具选择、路径、权限、环境、编码、换行或引号导致非预期失败时，先判断根因，再修改执行方式；禁止不改变假设地重复同一失败命令。
- 找到正确方式后，必须在继续同类操作前或至少在本次任务结束前更新错题本。记录日期、环境、错误命令或模式、症状/退出码、根因、正确命令或做法、预防检查和适用范围；所有密码、token、cookie、私有 URL 参数必须脱敏。
- 已有同类条目时更新“最近复发/补充”，不要堆重复条目。某错误重复出现两次，必须把预防规则提升到 `AGENTS.md`、脚本、测试或自动门禁，不能只继续记笔记。
- 产品测试按预期发现的业务失败不必逐条记；但测试命令写错、环境选错、乱码、误用 Shell、清理范围错误等执行问题必须记录。
- 任务交付前检查本次是否出现新的执行类错误；若有但错题本未更新，任务不得标记完成。
- 多代理协作只需即时状态时使用 `list_agents`；调用 `wait_agent` 等待结果时 `timeout_ms` 必须不低于工具 schema 的 `10000` 最小值，不得用短轮询试探参数边界。
- Codex Desktop 任务工具固定使用服务端分页上限：`list_threads limit<=50`、`read_thread turnLimit<=10`；需要更多任务或更早历史时只使用响应游标分页，不得试探更大的单页参数。

## Shell、工具与文件编码约定

### 生产 SSH

- Windows 当前用户已持久安装 `Posh-SSH 3.2.7`，模块路径为 `C:\Users\anxi\Documents\PowerShell\Modules\Posh-SSH\3.2.7`。连接飞牛服务器时优先在 PowerShell 7 中使用 `New-SSHSession`、`Invoke-SSHCommand` 和 `Remove-SSHSession`，不要再临时安装 Paramiko、Plink 或其它 SSH 客户端。
- 飞牛连接参数固定为主机 `121.40.29.22`、端口 `22000`、用户 `cz`。只使用用户在当前会话明确提供的密码构造内存中的 `SecureString`/`PSCredential`；密码不得写入本文件、脚本、PowerShell profile、环境持久化、日志或 Git。每次操作必须在 `finally` 中关闭 SSH session。
- 默认采用用户名密码认证。除非用户在当次请求中明确要求，不得创建本机 SSH 密钥、上传公钥、修改服务器 `authorized_keys` 或切换为密钥认证。首次主机指纹只允许在已核对目标主机时用 `-AcceptKey` 接受，后续不得绕过主机密钥校验。
- 普通只读或非交互命令使用 `Invoke-SSHCommand`；必须输入 `sudo` 密码时使用受控 `New-SSHShellStream`，密码只写入该会话流且不得回显或拼进远端命令行。传给 SSH 的 PowerShell 双引号字符串中禁止出现远端 `$变量`、`$()` 或反引号命令替换；简单探针改用不需要远端插值的独立命令，复杂远端诊断写任务专属脚本或使用 UTF-8 base64 载荷，避免 `pwsh → SSH → sh` 多层转义。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnXiYiZhi/stardew-server-anxi-panel](https://github.com/AnXiYiZhi/stardew-server-anxi-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
