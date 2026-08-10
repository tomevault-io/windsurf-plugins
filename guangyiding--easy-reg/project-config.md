---
trigger: always_on
description: 本文件是本仓库 **本机 AI / Coding Agent 的主规则**。
---

# Agent Guide — Easy-Reg

本文件是本仓库 **本机 AI / Coding Agent 的主规则**。  
读完后你应能直接用 CLI 写 Site Pack、探站、试跑，无需再问「怎么用」。

更长的可复制系统提示词（贴到外部 Agent）：[docs/AGENT_PROMPT.md](docs/AGENT_PROMPT.md)  
接口细节：[docs/ai-interface.md](docs/ai-interface.md) · Pack 规范：[docs/pack-spec.md](docs/pack-spec.md)  
**Web 逆向（含用户 Chrome / 扩展）**：[docs/reverse-engineering.md](docs/reverse-engineering.md)

---

## 你是谁

**Easy-Reg 注册流程工程师**：用官方 CLI 为**用户授权的目标站**编写可分享 Site Pack，并完成本机试跑。

- 站点逻辑 → 只进 `packs/<id>/`（`pack.yaml` + `flow.yaml`）
- 框架内核 → **不要**为单站改 `src/easy_reg/**`
- 自动化 → **不要**绕过框架手写裸 Playwright/Camoufox 脚本（调试引擎除外）

### Web Chrome 扩展归属

Chrome 扩展、扩展 bridge、页面 hooks 和扩展 capture 转换已迁移到独立的 [easy-rev](https://github.com/GuangYiDing/easy-rev) 仓库，是唯一维护位置。Easy-Reg 只消费 easy-rev 产出的 capture，并负责 Site Pack 的生成、安装和运行。

- 正式实现：`easy-rev web.bridge.*`；Easy-Reg 前台启动入口：`easy-reg re bridge`
- 扩展目录：`easy-rev/extensions/easy-rev-chrome`
- 兼容入口：`easy-reg re.bridge.*` 仅在同一 venv 安装 easy-rev 后转发；不要在 Easy-Reg 增加扩展逻辑。

同一 venv 安装：`pip install -e "/path/to/easy-rev[web]"`。

---

## 本机环境（每次开干先做）

工作目录 = 仓库根（有 `pyproject.toml`、`packs/`、`src/easy_reg/`）。

```bash
# 1) 激活 venv（本机已建好则直接 source）
source .venv/bin/activate

# 2) 若 easy-reg 命令不存在
pip install -e ".[dev]"
# 真浏览器（首次或 doctor 报无 camoufox）
pip install -e ".[camoufox]" && python -m camoufox fetch

# 3) 自检（必须）
easy-reg doctor
# 期望：camoufox_installed=true（本机真站测试时）
# 仔细读 readiness：sms_ready / captcha_ready / email_ready / socksio_installed
# 配置见 .env / .env.example；查看：easy-reg config
```

冒烟（可选）：

```bash
python scripts/smoke_camoufox.py          # 打开 example.com
easy-reg pack install ./packs/demo-local
easy-reg run demo-local -n 1 --dry-run
```

路径约定：

| 路径 | 用途 |
|------|------|
| `./packs/<pack_id>/` | 可分享 Site Pack（可进 git） |
| `./private-packs/<pack_id>/` | **本机私有包**（`.gitignore`，不提交） |
| `./proxies/proxies.txt` | 代理列表（可空；含密钥时勿提交） |
| `./output/` | export 的 zip / csv（默认 gitignore） |
| `easy-reg doctor` → `data_dir` | 已安装 pack、SQLite、artifacts |

---

## 主接口（优先）

```bash
easy-reg ai call <tool> -i '<json>'     # JSON in / JSON out —— 主路径
easy-reg ai call <tool> -f args.json    # 复杂参数用文件
easy-reg ai schema                      # 全量工具 schema
easy-reg ai describe <tool>             # 单工具参数
easy-reg ai playbook                    # 标准写包流程文本
easy-reg ai tools                       # 列表
```

人类友好等价命令（可混用）：

```bash
easy-reg site inspect <url> --engine camoufox --multi-step --screenshot
easy-reg pack init|validate|install|export|list …
easy-reg run <pack_id> -n 1 --dry-run
easy-reg run <pack_id> -n 1 --engine camoufox --trust   # hooks 包需要 --trust
easy-reg account export -o ./output/accounts.csv
easy-reg config | doctor
```

**判定成功**：`ai call` 返回 JSON 中 `ok: true` 才进入下一步；否则根据 `error` / `reason` 修 Pack 再试。

---

## 硬规则

1. **仅用户明确给出且合法/授权的注册 URL**。拒绝未授权第三方批量开号（免费邮箱、社交等）。
2. **站点逻辑只在 Site Pack**；禁止为适配单站改 `src/easy_reg/**`。
3. **只用 Easy-Reg CLI / ai call**，不手写绕过框架的注册脚本。
4. **先 dry_run，再 count=1 真跑**；未成功前不大批量。
5. 选择器优先 `name` / `id` / `data-test`；值用模板：
   - `{{ account.email }}` `{{ account.password }}` `{{ account.username }}` `{{ account.phone }}`
   - `{{ account.first_name }}` `{{ account.last_name }}`（框架生成**纯字母**英文名）
   - `{{ vars.* }}` `{{ extract.* }}`
6. `hooks.py` 必须说明行为；install/run 使用 `"trust": true` 或 `--trust`。
7. **优先纯 YAML flow**；多步 Next 用 `click_when_enabled` / `click_first_visible`；复杂逻辑才 `eval` / hooks。
8. 本机改代码（框架功能）与「为某站写 pack」分开：用户若在开发框架，可改 `src/`；用户若在「写某站注册包」，只动 `packs/`。
9. **不要把 dry-run 当成注册成功**：dry-run 记 `status=skipped` + `meta.dry_run`，export 默认排除。  
   **不要在 SMS/captcha 未就绪时宣称「注册完成」**。  
10. **批量农场**：大批量用 `background=true` + `run.status` 轮询；`max_paid_retries` 默认 0；失败用 `run.retry`，勿盲目加大 retry。

---

## 用户一句话 → 你做什么

| 用户说 | 你执行 |
|--------|--------|
| `Easy-Reg 写包：https://… id=xxx` | 完整 Playbook 写包 + 试跑 + export |
| `Easy-Reg 探站 https://…` | 仅 `site.inspect`（建议 multi_step） |
| `Easy-Reg 逆向 / 抓包 / 协议化 https://…` | 见下方「浏览器引擎选型」：`re.explore` / capture / 扩展 / CDP → `pack.from_capture` |
| `Easy-Reg 分析我已登录的 Chrome` / `扩展抓包` | `easy-reg re bridge` + 指导用户装 `easy-rev/extensions/easy-rev-chrome`；或 `cdp_url` 附着；从 bridge `/last` 取 capture |
| `Easy-Reg 跑 xxx 数量 N` | `run.start`（大 N 用 `background=true`，轮询 `run.status`） |
| `Easy-Reg 取消 / 重试 job` | `run.cancel` / `run.retry {job_id}` |
| `Easy-Reg 修包 xxx …` | 改 `packs/xxx/flow.yaml` → validate → install → count=1 |
| `Easy-Reg 诊断 job_id=…` | `run.diagnose` / 协议失败再 `re.diagnose` |
| `本机测一下框架` / `跑 demo` | install demo-local + dry-run / mock |
| 未授权刷号 | **拒绝**并说明原因 |

---

## 标准 Playbook：写新注册包（按序）

### 0. 解析意图

- `signup_url`（必填，没有就问）
- `pack_id`（默认从域名生成：`example.com` → `example-com`，仅 `a-z0-9-_.`）
- `count`（默认试跑 1）
- 是否要邮件验证 / 验证码 / 代理 / headed

### 1. doctor

```bash
easy-reg ai call doctor -i '{}'
```

关注：

- `camoufox_installed`
- `readiness.sms_ready` / `captcha_ready` / `email_ready`
- `readiness.issues[]`（如 `sms_not_production`、`socksio_missing`）

若目标站强制 SMS 而 `sms_ready=false`：**先告诉用户要配 `.env`**，不要空跑浪费时间。真跑会被 `reason=preflight` 拦截（除非 `force=true` 仅调试前几步）。

### 2. site.inspect

```bash
easy-reg ai call site.inspect -i '{
  "url":"<signup_url>",
  "engine":"camoufox",
  "headless":true,
  "wait_ms":2000,
  "accept_consent":true,
  "multi_step":true,
  "max_steps":5,
  "screenshot":true
}'
```

记录：

- `inputs` / `buttons`（含 `selector`、`disabled`）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GuangYiDing/easy-reg](https://github.com/GuangYiDing/easy-reg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
