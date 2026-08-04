---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Flowix 是一款桌面笔记应用（**Tauri 2 + Rust 后端，React 19 + TS + Tiptap 前端**），内置 AI 代理（`rllm` v1.1，OpenAI / Anthropic / DeepSeek 全部走 `openai_compatible` provider）。


## 命令

```bash
export PATH="$HOME/.cargo/bin:/opt/homebrew/bin:/usr/local/bin:$PATH"
npm run tauri:dev        # 推荐：独立 dev bundle ID (com.flowix.app.dev / "Flowix Dev")，可与生产 app 并存
npm run tauri:dev:win    # Windows 开发启动：使用 app/flowix-desktop/tauri.windows.dev.conf.json
npm run tauri dev        # ⚠️ 走默认 tauri.conf.json，与生产同 bundle ID (com.flowix.app)，已被生产占住时会立刻 exit 0
npm run dev              # 仅前端 (localhost:1420)
npm run tauri build      # 生产构建
npm run cli:build        # 编 CLI sidecar 到 app/flowix-desktop/binaries/（当前 host）
npm run cli:build:all    # CI 用：三平台（linux / macOS ×2 / windows）全编
pkill -f "node.*vite" 2>/dev/null   # 端口冲突时
sudo xcode-select -r                 # 首次运行
```

Rust 测试（在 `app/` 目录跑）：

```bash
cd app
cargo test -p flowix-core <module>::tests           # 跑某 crate 某模块
cargo test -p flowix-core <module>::tests::test_xxx # 跑单个
cargo test --workspace --lib                         # 跑全部
```

## Dev / Prod 并存打包

通过差异化 Tauri 配置，让 dev 版与已安装的生产版同时运行：

- **dev**：`npm run tauri:dev` → `app/flowix-desktop/tauri.conf.dev.json` → bundle ID `com.flowix.app.dev` / `Flowix Dev`
- **生产**：`npm run tauri:build:production` → `tauri.conf.json` + 平台覆盖层 + 签名覆盖层 → 平台专用 `tauri.*.production.local.json` → bundle ID `com.flowix.app` / `Flowix`
- **默认 build**：`npm run tauri:build` → 默认 `tauri.conf.json` → 生产身份（无签名，便于本地试装）

`tauri:dev` 通过 `--config` 指向独立配置，**不要**改 `tauri.conf.json` 的 `identifier` / `productName` / `mainBinaryName` / `bundle.macOS.bundleName` —— 这四个字段是生产身份的锚点。`tauri.conf.production.json` 作为覆盖层被 `tauri build --config` 深合并在 `tauri.conf.json` 之上，因此 dev 配置改动不会污染生产链路。

dev 与生产现使用不同 bundle ID（`com.flowix.app.dev` vs `com.flowix.app`），可同时运行且互不冲突（Tauri `app_data_dir` / `tauri-plugin-single-instance` lock 都按 identifier 派生）。代价：dev 首次运行需要重新授予一次 user-selected folder 授权（TCC 按 identifier 记忆授权），prod 已授予的不会带过来。视觉上仍通过 bundle name / 窗口标题区分（`Flowix Dev` vs `Flowix`）。URL scheme `flowix://` 仍共用，让浏览器深链能落到任一已装实例。

### macOS 本地生产包 ad-hoc 签名

构建 macOS 生产包后，如果没有 Developer ID，也要对 bundle 内 sidecar 和 `.app` 做一次本地 ad-hoc codesign，让 `entitlements.plist` 写进可执行产物；否则 security-scoped bookmarks / user-selected folder 权限相关 entitlement 不会实际生效。

```bash
npm run tauri:build:production

codesign --force --options runtime --sign - \
  --entitlements app/flowix-desktop/entitlements.plist \
  "app/flowix-desktop/target/release/bundle/macos/Flowix.app/Contents/MacOS/flowix-cli"

codesign --force --deep --sign - \
  --entitlements app/flowix-desktop/entitlements.plist \
  "app/flowix-desktop/target/release/bundle/macos/Flowix.app"
```

`--sign -` 是 ad-hoc 签名，只适合本机开发 / 本地试装，不能替代 Developer ID 签名与 notarization。先签 `Contents/MacOS/flowix-cli`，再签外层 `.app`；若实际产物路径不同，以 `target/release/bundle/macos/*.app` 为准。

## macOS 发布流水线（Developer ID 直分发 + Notarization）

production 发版走 Apple Developer ID 直分发（不走 Mac App Store），完整脚本在 `scripts/apple-signing/`：

```
scripts/apple-signing/
├── gen-csr.sh            # 生成 CSR + private key → ~/.flowix-signing/
├── make-p12.sh           # .cer + .key → 可导入 Keychain 的 .p12（legacy 路径）
├── sign-and-notarize.sh  # 完整发版：tauri build → resign → notary → staple
└── README.md             # 流程图 + 5 步走 + 隐私边界
```

### 一次性配置

#### 1. Apple Developer Account
- 账号邮箱（注册时绑定的，**不能改**）：Apple ID 主邮箱
- Membership Details → **Legal Entity Name**（精确复制粘贴）
- notary 凭据的源邮箱必须跟 Apple Developer Account 邮箱一致

#### 2. 生成本地私钥 + CSR
```bash
bash scripts/apple-signing/gen-csr.sh \
  "<Apple ID 邮箱>" \
  "<Common Name，ASCII>" \
  "<Legal Entity Name 精确>" \
  "CN"
```
输出：
- `~/.flowix-signing/devid.key`（private key，**永不丢失**）
- `~/.flowix-signing/devid.csr`（后续可重用来 renewal）

#### 3. Apple Developer Portal 创建 Developer ID Application 证书
- 登 <https://developer.apple.com/account/resources/certificates/list>
- `+` → **Developer ID Application**（**不是** Apple Development / Apple Distribution / Developer ID Installer）→ 上传 `devid.csr`
- 下载 `developerID_application.cer`（Safari 可能保存为 `development.cer`，**以 CN 字段的 `Developer ID Application:` 前缀为准**判断）
- 把 .cer 放进 `~/.flowix-signing/`

#### 4. 合 .p12 + 导入 Keychain

```bash
# 合包（绕过 make-p12.sh 交互式 prompt，一次性 inline）
openssl x509 -in ~/.flowix-signing/development.cer -inform DER -out /tmp/devid.pem
PASS="$(openssl rand -base64 24 | tr -dc 'A-Za-z0-9' | head -c 24)"
printf '%s' "$PASS" > ~/.flowix-signing/.p12pass
chmod 600 ~/.flowix-signing/.p12pass

openssl pkcs12 -export \
  -inkey ~/.flowix-signing/devid.key \
  -in /tmp/devid.pem \
  -out ~/.flowix-signing/devid.p12 \
  -name "$(openssl x509 -in /tmp/devid.pem -noout -subject | sed 's/^subject=//')" \
  -passout "file:$HOME/.flowix-signing/.p12pass"
chmod 600 ~/.flowix-signing/devid.p12
rm -f /tmp/devid.pem

# 导入 login keychain（加 -T 让 codesign 不再要求密码）
PASS="$(cat ~/.flowix-signing/.p12pass)"
security import "$HOME/.flowix-signing/devid.p12" \
  -k "$HOME/Library/Keychains/login.keychain-db" \
  -P "$PASS" \
  -T /usr/bin/codesign \
  -T /usr/bin/security \
  -T /usr/bin/codesign_allocate \
  -T /usr/bin/productbuild \
  -A
```

验证：
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [text2future/flowix](https://github.com/text2future/flowix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
