---
trigger: always_on
description: 让 Claude Science 的模型推理走第三方 API（阿里通义千问 / DeepSeek 等），保留 Science 那套「AI Jupyter」体验，模型换成便宜或开源的。类比 CC Switch 之于 Claude Code。
---

# CSSwitch

让 Claude Science 的模型推理走第三方 API（阿里通义千问 / DeepSeek 等），保留 Science 那套「AI Jupyter」体验，模型换成便宜或开源的。类比 CC Switch 之于 Claude Code。

## 一、铁律（最高优先级，任何会话都不得违反）

1. **绝不影响用户真实的订阅与登录状态。** 真实 Claude Science 的数据目录是 `~/.claude-science`，登录凭证在 `~/.claude-science/.oauth-tokens`、`active-org.json`、`encryption.key`、`orgs/`、`.key-backups/`。这些文件**只读都要谨慎，绝不复制、绝不修改、绝不删除**。
2. **绝不把真实 OAuth token 复制进任何沙箱。** 复制后两个实例共享同一 token，刷新时 Anthropic 可能轮换刷新令牌，导致用户真实实例被登出。要给沙箱登录，只能在沙箱里**全新独立登录**（另一套会话 token，对真实登录零影响），且由用户手动完成，Claude 不代做登录。
3. **绝不用改过的环境变量去启动用户的真实实例。** 真实实例跑在端口 8765。所有实验用的沙箱必须用**独立 data-dir + 独立端口 + 独立 HOME**，与 8765 完全隔开。
4. **测试默认不碰 Science。** 能用「代理↔上游」单独验证的，就不启动 Science（见 `test/`）。只有到最终整链联调、且用户明确同意时，才启动沙箱 Science，并且仍然遵守第 2、3 条。
5. 动任何有状态的东西前，先确认它不在铁律清单里；拿不准就停下来问用户。

## 二、架构

```
Claude Science（保留登录，仅当启动门票；推理不走 Anthropic）
   │  ANTHROPIC_BASE_URL=http://127.0.0.1:<port>
   ▼
翻译代理（本项目 proxy/qwen_proxy.py，或 CC Switch 内建代理）
   │  剥离入站 OAuth Bearer，注入第三方 key，Anthropic Messages ↔ OpenAI 格式互转
   ▼
阿里 DashScope（通义千问）/ DeepSeek / 其它 OpenAI 兼容端点
```

关键点：Claude 登录只是**启动 Science 的门票**，推理被 `ANTHROPIC_BASE_URL` 导去本地代理后，Anthropic 服务端不经手推理。代理负责把 Science 带来的 OAuth Bearer 丢掉、换成第三方 key，并做格式翻译。

## 三、已验证的事实（有证据，别重复推导）

来自对二进制 `/Applications/Claude Science.app/Contents/Resources/bin/claude-science`（内部代号 operon）的静态分析 + 实测：

1. **base_url 无条件生效**：`LJ()` 直接读 `process.env.ANTHROPIC_BASE_URL`，登录后推理请求会打到它。Codex 实测：登录态下 Science 向本地代理发出了 `GET /v1/models`、`POST /v1/messages`。
2. **手动 API key 被写死拒绝**：凭证解析器 `HLO.resolve()` 只认 OAuth（`_tryOauthToken`），`_tryManualApiKey()` 恒返回 `null`；还有守卫把「等于环境变量的凭证」置空。所以**完全不登录 + 只填 key 的路子走不通**，必须有 OAuth 门票。这也是早期「隔离 HOME 后 mock 收到 0 请求」是**假阴性**的原因：隔离把登录也隔离了，Science 在发 HTTP 前就因无 OAuth 而终止。
3. **CC Switch 的代理本身就是完整翻译器**：其二进制含 `/v1/messages`、`/v1/chat/completions`、`cc_switch_transform_error`、两套协议字段与 SSE 桥接，内建模型目录含 DeepSeek/Qwen/Kimi 等。翻译引擎不用自己造。CC Switch 代理端口默认 `127.0.0.1:15721`，`proxy_config` 的 `app_type` 只允许 `claude/codex/gemini`（Science 复用 `claude` 那条即可，无需新增类型）。
4. **翻译代理 ↔ 真实通义千问，整条链路已跑通**（本项目 `proxy/qwen_proxy.py`，隔离环境实测，未碰 Science/OAuth/CC Switch）：
   - `/v1/models`、非流式、**流式 SSE**、**tool_use 发起**、**tool_result 回喂后模型接着作答** 全部通过；
   - 入站 OAuth Bearer 逐条确认被剥离，未转发上游。
   - 证据：`findings/e2e-proxy-qwen-proof.log`。
5. **虚拟 OAuth（本地自造令牌，零 Anthropic、零真实凭证）已跑通整链**（2026-07-02，证据 `findings/e2e-virtual-oauth-fullchain-proof.log`）：
   - `_tryManualApiKey` 恒 null，但登录门票**不必真登录**：直接在沙箱 auth_dir 伪造一份加密令牌即可让 Science 认为已登录。
   - 令牌文件 `.oauth-tokens/<account_uuid>.enc`，格式 `"v2:"+base64(IV(12)‖AES-256-GCM‖tag(16))`；派生 `hkdfSync("sha256", base64(OAUTH_ENCRYPTION_KEY), Buffer.alloc(0), "operon:aes-256-gcm:oauth", 32)`，AAD=`v2:oauth`，明文是 token blob JSON。目录里须**恰好一个** `.enc`。
   - `encryption.key` 是换行分隔 `KEY=VALUE`：`OAUTH_ENCRYPTION_KEY`/`ANTHROPIC_API_KEY_ENCRYPTION_KEY`/`USER_SECRET_ENCRYPTION_KEY`(base64≥16B) + `JWT_SIGNING_SECRET`(≥16 字符)。keychain 镜像账号按**路径 SHA256** 派生（`encryption.key-<hash12>`），沙箱与真实天然隔离；本机实测 keychain 写入超时被跳过，纯用文件。
   - 关键坑：`token_expires_at` 必须设远期 ISO 串（如 `2099-01-01T...Z`），否则 `qP()` 判过期 → `_refreshToken` 联网打 `platform.claude.com` → 失败即无凭证。`provider="claude_ai"`，scopes=`user:inference user:file_upload user:profile user:mcp_servers user:plugins`。
   - `subscription_type` 由令牌自填、启动/鉴权阶段**不做服务端付费校验**（profile/account 走硬编码 api.anthropic.com，失败无害）。即**无需任何 Anthropic 账号**，"免费账号门票"问题作废。
   - 工具：`scripts/make-virtual-oauth.mjs`(Node，与二进制字节级一致) + `scripts/launch-virtual-sandbox.sh`。
   - Science 自身 `GET /api/auth/status` 返回 `authenticated:true, email:virtual@localhost.invalid`；真实 agent 会话中 `claude-opus-4-8→qwen-max`(推理) 与 `claude-haiku-4-5-20251001→qwen-turbo`(标题) 都经代理译到千问并在 transcript 渲染。
   - 沙箱守护 API 驱动：身份取自磁盘令牌（`AE()="none"` 用 `O9()`），写操作需 `Origin: http://localhost:<port>` + 双提交 CSRF（cookie `operon_csrf` 回显到头 `x-operon-csrf`）；建会话 `POST /api/frames {project_id}`，发消息 `POST /api/frames/:id/message {input_data:{request:"..."}, model}`（**用户文本键是 `request`，不是 text**）。
   - **沙箱钥匙串弹窗（已修，2026-07-02）**：Science 会把 `encryption.key` 镜像进 macOS 钥匙串。沙箱用独立 HOME(`.sandbox/home`)，其下无任何钥匙串，`HOME=$SANDBOX_HOME` 的进程 securityd 报「找不到默认钥匙串」，于是反复弹「找不到钥匙串 → 还原为默认」窗。这是纯隔离副作用，不是报错；误点「还原为默认」会改钥匙串默认设置，正解是点「取消」，Science 会退回读磁盘上的 `encryption.key` 文件照常工作。修法：`launch-virtual-sandbox.sh` 在**沙箱 HOME 内**建一个独立、空密码、不自动锁的 `Library/Keychains/login.keychain-db`，只在 `HOME=$SANDBOX_HOME` 上下文里 `security create/list-keychains/default-keychain -d user -s`（写的是沙箱侧偏好，`securityd` 按 HOME 隔离）。核对前后**真实** `~/Library/Keychains` 的 default 与 list 逐字节不变。修后启动日志出现 `encryption keys copied to the macOS Keychain`，弹窗消失。

## 四、尚未验证 / 待办（别当已全通）

- [x] **整链联调**：真实 Science(沙箱·虚拟登录) → 本项目代理 → 千问，同一次运行合上。见上第三节第 5 点。
- [x] **门票能否用免费账号**：作废——根本不需要任何 Anthropic 账号，伪造令牌即可，tier 自填不校验。
- [x] **多轮工具循环已验证**：`tool_use(python) → 人工批准门 → 内核执行 → tool_result 回喂 → 继续作答`，答案正确（`print((999*888)+77)` → 887189）。
  - 坑1：模型写**裸表达式**（`result` 而非 `print(result)`）时，Science 的 tool_result 只含 stdout（空），模型会瞎猜 → 让模型用 `print()`。
  - 坑2：代码执行是**人工批准门**（`output_data.pending_input_requests`，UI 里点「运行」）。无头驱动：`POST /api/frames/:id/resolve-input`，body `{responses:[{requestId,tool_id,approved:true,action:"allow",scope:"conversation|project|always"}]}`；`--dangerously-skip-approvals` 在此 ant build 被禁用。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SuperJJ007/CSswitch](https://github.com/SuperJJ007/CSswitch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
