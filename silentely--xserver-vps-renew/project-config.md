---
trigger: always_on
description: > 自动为 Xserver 免费 VPS 执行续期操作，基于 Puppeteer Stealth + rebrowser-patches 绕过 Cloudflare Turnstile 检测。
---

# Xserver VPS 自动续期工具

> 自动为 Xserver 免费 VPS 执行续期操作，基于 Puppeteer Stealth + rebrowser-patches 绕过 Cloudflare Turnstile 检测。

## 变更记录 (Changelog)

| 日期 | 变更内容 |
|------|----------|
| 2026-08-27 | 十轮迭代打磨：日志时间戳 `Intl.DateTimeFormat` 按 tz 缓存（每条日志不再新建 Intl 对象）；轮询路径日志降噪（`getTurnstileToken` 读取失败 error→debug；求解成功/链路/域名代理提示去重降 debug）；提交结果轮询自适应退避 `resolveSubmissionPollIntervalMs`（10s/30s 两段，120s 窗口 CDP 往返 ≈300→≈90）；`TURNSTILE_PROVIDER_ORDER` 拼写错误启动 warn（`listUnknownTurnstileProviderNames`）；entrypoint `show_cron_schedule` 支持「27 */4」错峰文案；diagnostics.sh 状态文件可写性探测；用户脚本到期判定对齐主脚本 #5（仅今天到期进续期页）+ 面板关闭按钮键盘可达（1.0.7）；通知 `<strong>`→`<b>` 统一 + `handleCaptchaPage` 边界防御；启动日志新增运行环境行（25 文件 / 479 用例） |
| 2026-08-22 | 任务 41：收尾任务 40——npm 纯构建期工具，`npm ci` 后整体从运行时镜像移除（含 npx/corepack/yarn，node 基础镜像自带的包管理器全清），根治 npm 捆绑运行期依赖的 CVE 打地鼠（picomatch/sigstore→升 npm、tar→显式升级均为该类别）；运行时仅依赖 node，entrypoint 直接执行主脚本；镜像体积略减，Trivy 无 HIGH/CRITICAL |
| 2026-08-22 | 任务 40：修复 CI Trivy 门禁（CVE-2026-73566 node-tar DoS，HIGH）：npm 官方 tarball 捆绑的运行期依赖 tar 7.5.19 落后于修复版 7.5.21，`npm install -g npm@latest` 直接解包捆绑依赖、不会按 semver 重新解析（第三次 npm 内嵌依赖漏洞：picomatch/sigstore→升 npm、本次 tar→显式升级），Dockerfile 在 npm 升级后显式 `npm install -g tar@7.5.22` 并覆盖进 `/usr/local/lib/node_modules/npm/node_modules/tar`（7.5.19 与 7.5.22 包结构一致，main/exports/dist 兼容；本地已下载 tarball + trivy 复现确认） |
| 2026-08-22 | 任务 39：修复两次手动运行暴露的两个竞态——(1) 提交结果判定过早：官方 /extend/do 处理需 60-90s，`evaluateSubmissionResult` 对「停留 conf 无失败标识」从即时 retry 改为 `pending`，`waitForSubmissionResult` 长轮询至 `SUBMISSION_RESULT_TIMEOUT_MS`（新配置，默认 120s，三处同步），避免重试导航中止在途 POST（ERR_ABORTED）把可成功的提交误判失败；(2) Turnstile token 注入撞 detached frame：新增 `injectTurnstileTokenWithRetry`（frame 脱离类错误原地重试 2 次），失败原因区分「求解失败」与「注入失败」并透传至通知；顺带 debug 日志埋点请求失败降噪 `isBenignRequestFailure`（GA/广告回传被导航中止不再刷屏）（24 文件 / 458 用例） |
| 2026-08-11 | 任务 37 十轮迭代：成功路径状态文件读取合并（3 次→2 次 I/O，`priorTotalRuns` 由 `totalRuns>1` 推导）；debug 级浏览器 console/pageerror/requestfailed 监听（条数上限防刷屏）；失败通知附验证码重试次数 `captchaRetries`（`error.captchaAttempts`，+2 用例）；日志超长截断 `clampLogMessage`（+3 用例）与 `[步骤N]` 序号；用户脚本 Turnstile token 监听改轮询主路径（原 MutationObserver 仅观察 attribute 恒等超时）；用户脚本状态面板标题/关闭按钮/success 3s 自动收起；22 个多余导出收敛 + `pollTurnstileTaskResult` 拆分 + theme/action 精简；CLI `--version/--help` + 启动横幅文档入口；人工确认通知「下次执行」→「下次检查」（23 文件 / 439 用例） |
| 2026-08-08 | 任务 36 十轮迭代：Turnstile 截图按 debug 按需写入（`SAVE_TURNSTILE_SCREENSHOTS`）；`page.close` 异常防御 `safeClosePage` 防双通知；自然通过降级立即点击；VPS 行解析纯函数 `extractVpsInfoFromCellTexts`；skip 日志去重；人工确认通知补本地 Node 重跑命令；用户脚本 Turnstile 超时不再强制提交；诊断脚本代理地址脱敏；新到期日提取收敛 `extractNewExpireDate`；entrypoint/cron-run/diagnostics 时间戳显式 `TZ`（23 文件 / 430 用例） |
| 2026-08-07 | 迭代打磨：分阶段耗时日志（pushStep 每步耗时）；`waitForSubmissionResult` 提交后轮询成功信号替代固定 2s（+4 用例）；Turnstile 注入后 token 软等待；通知/日志时间格式统一（`formatTokyoDateTime` 委托 `formatLogTimestamp`）；失败通知补「下次检查」行；Turnstile 轮询 debug 日志降噪（每 5 轮）；启动日志打印上次运行结果摘要；用户脚本 UI 无障碍（reduced-motion/aria-live/滚动）；`diagnostics.sh` 新增 Keras + 打码平台 API 连通性探测（22 文件 / 410 用例） |
| 2026-08-07 | 打磨：日志增加 [DEBUG]/[INFO]/[WARN]/[ERROR] 级别标签（formatLogLine）；关键告警提级 warn（failover 失败/熔断）；时间戳单源化 formatLogTimestamp + renewal-status 接入 logger；通知「下次执行」→「下次检查」+ 成功通知「连续成功 N 次」（countConsecutiveSuccesses）；Telegram 响应体 ok:false 校验（parseTelegramSendResult）；Turnstile/验证码图等待改软等待 waitForSelectorSoft；指纹体检 analyzeFingerprintHealth；用户脚本状态面板状态色/动画（21 文件 / 403 用例） |
| 2026-08-06 | 打磨：Turnstile 参数属性双名兼容（`data-c-data`/`data-cdata` 等，防 Anti-Captcha cData 漏取）；抽取 `getBodyText`/`listFailedTurnstileProviders`/`FAILURE_CATEGORY_LABELS` 消除重复；`checkRenewalNeeded` 统一 `nowMs` 基准；修复 `resolveCaptchaRetryUrl` 的 `/index` 重复 `extend` 段路径 bug；登录失败抛错附带页面提示（19 文件 / 364 用例） |
| 2026-08-06 | 代码质量重构：主脚本 1638→795 行（Turnstile UI 层 → `src/turnstile-flow.mjs`、面板流程 → `src/panel-flow.mjs`、页面工具 → `src/page-utils.mjs`）；消除 notify↔turnstile 跨模块 magic string 重复（委托 `isTurnstileOutageError`）；修复 Turnstile callback 双重触发；`extractTurnstileParams` 与 `readTurnstileWidgetParams` 共享属性名常量（单一来源）；结构化分级 logger 取代 `isNoisyModuleLog` 字符串嗅探（净删）；失败路径 outage 判定单次求值；`isRenewalDue` 移除死参数、`formatTokyoDateTime` 尊重 `TZ`；新增 cron 白名单防漂移测试（20 文件 / 372 用例） |
| 2026-08-06 | 防漂移：`env-whitelist.test.mjs` 校验 cron-run.sh 白名单 ↔ 主脚本 CONFIG ↔ .env.example 三处同步（21 文件 / 376 用例） |
| 2026-08-05 | 修复：官方新增「個人情報の取り扱いについて」同意页（`/xapanel/myaccount/agreement`），登录后未同意即被重定向导致误判「未找到免费 VPS」；新增 `ensureAgreementAccepted` / 用户脚本 `handleAgreement` 自动勾选 `#agree_flag_1` 并提交；自动处理无效时（同意页改版/未进入面板页）发 `buildManualConfirmNotifyMessage` 提醒用户人工确认后重跑容器；`checkRenewalNeeded` 增加表格等待与页面结构诊断（诊断日志定位到本根因）（19 文件 / 359 用例） |
| 2026-08-04 | 修复：`finishWithSkip` 越界引用 try 块内 `page`，导致「无需续期」场景双通知（skip+failure）且退出码 1；`page` 改显式传参（19 文件 / 356 用例全绿） |
| 2026-08-04 | 重构：拆分 `src/notify.mjs`（Telegram 通知构建）；utils 收纳 `escapeHtml`/`findChromePath`/`cleanChromeLocks`/`formatTokyoDateTime`；主脚本去除死重导出与模块包装层 |
| 2026-08-04 | 打磨：emitLog 单次取时间戳修复跨秒双时间戳、skip 通知统一出口、parsePositiveInt 严格校验、文档测试清单同步（19 文件 / 356 用例） |
| 2026-07-31 | 修复 #10 附带发现：Docker cron 下通知「下次执行」恒回退 +6h；cron-run 透传仅展示用 `CRON_SCHEDULE_DISPLAY`（不作模式开关） |
| 2026-07-31 | 采纳 #9：compose 默认调度 `0 */6` → `27 */4`（每 4h 错峰 27 分，任意 12h 续期窗口内 ≥3 次尝试） |
| 2026-07-29 | 修复 #7：entrypoint `--once` 优先于 `CRON_SCHEDULE`，cron-run 调用时清空调度变量，避免嵌套 supercronic 死锁 |
| 2026-07-26 | Turnstile：先注入 token 再对齐 UA；未通过禁止提交；重试回 index?id_vps；文档提醒 AntiCaptcha 域名代理 Proxyless IP 不一致 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Silentely/xserver-vps-renew](https://github.com/Silentely/xserver-vps-renew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
