---
trigger: always_on
description: 全自动分析网站加密逻辑，自动提取密钥并生成 mitmproxy 脚本，实现 BurpSuite 与目标服务器之间的**双向透明加解密**。
---

# Web Crypto Proxy Skill

## 描述

全自动分析网站加密逻辑，自动提取密钥并生成 mitmproxy 脚本，实现 BurpSuite 与目标服务器之间的**双向透明加解密**。

## 核心架构

```
目标 URL ──→ AI 自动分析 ──→ 提取密钥 ──→ 生成脚本 ──→ mitmproxy 运行
                                                          ↓
浏览器 → BurpSuite (:8080) → mitmproxy (:8083) → 目标服务器
              ↑                    ↓
         [明文操作]          [自动加解密]
```

## 双向加解密机制

```
┌─────────────────────────────────────────────────────────────────┐
│                      双向透明加解密                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  【请求方向】BurpSuite 明文 → mitmproxy 加密 → 服务器收到密文     │
│                                                                  │
│    BurpSuite 发送:                                               │
│    {"username": "admin", "password": "123456"}                   │
│                         ↓                                        │
│    mitmproxy 检测明文字段，自动加密:                              │
│    {"username": "Z09xqdN...", "password": "Yq+prW..."}           │
│                         ↓                                        │
│    服务器收到加密数据                                            │
│                                                                  │
│  【响应方向】服务器密文 → mitmproxy 解密 → BurpSuite 显示明文     │
│                                                                  │
│    服务器返回:                                                   │
│    {"data": "encrypted_base64_string..."}                        │
│                         ↓                                        │
│    mitmproxy 检测加密数据，自动解密:                              │
│    {"data": {"user": "admin", "role": "admin"}}                  │
│                         ↓                                        │
│    BurpSuite 显示明文数据                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 使用方法

```
skill: web-crypto-proxy https://target.com
```

AI 将**自动执行**以下步骤：

1. 获取网站 HTML 源码
2. 提取并下载所有 JS 文件
3. 分析加密算法和提取密钥
4. 生成完整的 mitmproxy 脚本
5. 测试加解密功能
6. 提供启动命令

## AI 自动执行流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI 自动化分析流程                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  [Step 1] 获取目标 URL，解析域名                                 │
│       ↓                                                          │
│  [Step 2] curl 获取 HTML 源码                                    │
│       ↓                                                          │
│  [Step 3] 正则提取所有 <script src="..."> JS 文件链接           │
│       ↓                                                          │
│  [Step 4] 批量下载 JS 文件到本地                                 │
│       ↓                                                          │
│  [Step 5] 搜索加密关键词:                                        │
│           - encrypt / decrypt                                    │
│           - CryptoJS / JSEncrypt / sm2 / sm4                     │
│           - setPublicKey / setPrivateKey                         │
│           - AES / DES / RSA / SM2 / SM4                          │
│       ↓                                                          │
│  [Step 6] 正则提取密钥:                                          │
│           - RSA 公钥: MFwwDQYJKoZIhvcNAQEB...                    │
│           - RSA 私钥: MIIBVAIBADANBgkqhkiG9w...                  │
│           - AES 密钥/IV: CryptoJS.enc.Utf8.parse(...)            │
│       ↓                                                          │
│  [Step 7] 识别加密算法、模式、填充方式                           │
│       ↓                                                          │
│  [Step 8] 生成 Python 加解密函数代码                             │
│       ↓                                                          │
│  [Step 9] 生成完整 mitmproxy 脚本（包含双向加解密逻辑）          │
│       ↓                                                          │
│  [Step 10] 测试加解密功能是否正常                                │
│       ↓                                                          │
│  [Step 11] 输出分析报告和启动命令                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## AI 执行的具体命令

### 获取 HTML

```bash
curl -s -k "https://target.com/" -o index.html
```

### 提取 JS 文件链接

```bash
grep -oE 'src="[^"]+\.js[^"]*"' index.html
grep -oE '<script[^>]+src="[^"]+"' index.html
```

### 下载 JS 文件

```bash
mkdir -p js
# 对每个 JS URL
curl -s -k "$JS_URL" -o "js/$(basename $JS_URL)"
```

### 搜索加密关键词

```bash
grep -rn "encrypt\|decrypt\|CryptoJS\|JSEncrypt" js/
grep -rn "setPublicKey\|setPrivateKey\|AES\|RSA\|SM2\|SM4" js/
```

### 提取 RSA 密钥

```bash
# RSA 公钥 (512-bit)
grep -o 'MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJB[A-Za-z0-9+/=\n]*' js/*.js

# RSA 公钥 (1024-bit)
grep -o 'MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC[A-Za-z0-9+/=\n]*' js/*.js

# RSA 公钥 (2048-bit)
grep -o 'MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA[A-Za-z0-9+/=\n]*' js/*.js

# RSA 私钥
grep -o 'MIIB[A-Za-z0-9+/=\n]*' js/*.js | grep -E '(PRIVATE|AgEAAkEA)'
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arg3Sea/web-crypto-proxy](https://github.com/Arg3Sea/web-crypto-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
