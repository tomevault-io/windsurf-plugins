---
trigger: always_on
description: 本项目是 Yaklang 的 AI 知识库，用于训练和教学 Yaklang DSL 的使用方法。Yaklang 是一个专为安全研究和渗透测试设计的 DSL，底层库使用 Golang 编写，文件扩展名为 `.yak`。
---

# Yaklang AI Training Materials - Cursor Rules

## 项目概述

本项目是 Yaklang 的 AI 知识库，用于训练和教学 Yaklang DSL 的使用方法。Yaklang 是一个专为安全研究和渗透测试设计的 DSL，底层库使用 Golang 编写，文件扩展名为 `.yak`。

## Yaklang 基础信息

### 库的导出机制
- 所有库通过 `yaklang.Import(库名, 导出表)` 从 yak 仓库中导出
- 可以在 `/Users/v1ll4n/Projects/yaklang/common/yak/script_engine.go` 中查找所有可用库
- 使用 `grep -r "yaklang.Import" /Users/v1ll4n/Projects/yaklang` 来搜索库导出

### 可用库列表（来自 yaklang.Import）
基础库：str, math, os, file, fileparser, excel, filesys, re, re2, regen, env, sync, io, bufio, context, time, timezone, codec, log
网络库：http, httpserver, httpool, tcp, udp, tls, mitm, dns
安全工具：poc, csrf, risk, report, dnslog, nuclei, nasl
扫描库：synscan, finscan, servicescan, subdomain, ping, traceroute, spacengine, brute
协议库：ssh, smb, ldap, redis, rdp, t3, iiop
爬虫库：crawler, crawlerx
漏洞库：cve, cwe, yso, java
数据处理：json, yaml, xml, xpath, zip, gzip, jwt
高级功能：hook, dyn, db, judge, facades, bot, simulator, ja3, sca, git, bin, ssa, syntaxflow, openapi, sandbox
AI相关：ai, aiagent, liteforge, jsonschema, aireducer, rag
系统工具：exec, cli, fuzz, fuzzx, hids, systemd, mmdb, pcapx
容器和工具：container, orderedmap, twofa, mfa, omnisearch, memeditor, amap, fp, mimetype, imageutils, ffmpeg, whisper, pandoc, toolbox, diff
数据库：yakit, x（funk utilities）
其他：rpa, suricata, pprof, tools, dictutil, xhtml

### 引擎使用
- 使用 `which yak` 找到引擎位置
- 使用 `yak filename.yak` 执行脚本验证
- 脚本必须在 10 秒内完成，避免死循环

### 调试工具
- 使用 `desc(instance)` 查看对象结构和方法
- 使用 `assert` 验证关键结果
- 使用 `log.info()`, `log.error()` 等进行日志输出

## 编码规范

### 代码风格
1. **严肃性**：保持代码严肃，不使用 emoji，只使用 ASCII 字符、中文和必要标点符号
2. **注释语言**：
   - 代码注释可以使用中文
   - log 输出必须使用英文
   - 字符串内容必须使用英文
   - 中文只允许出现在注释中
3. **函数定义**：优先使用 `func functionName() {}` 形式定义函数
4. **错误处理**：
   - 使用 `~` 波浪操作符自动处理错误（WavyCall）
   - 使用 `assert` 验证关键结果
   - 使用 `try-catch-finally` 或 `defer recover()` 处理复杂错误

### 关键词注释规范
**重要**：在代码的关键位置（而非文件头部）添加关键词注释，用于 AI 搜索优化和 grep 索引：

```yak
// 基础HTTP请求测试
// 关键词: poc.HTTP, 基础HTTP请求
func testBasicHTTPRequest() {
    log.info("test basic HTTP request functionality")
    
    // 构造基础HTTP请求
    // 关键词: poc.HTTP, 原始HTTP请求包
    rawRequest = `GET / HTTP/1.1
Host: example.com
`
    
    // 发送HTTP请求
    // 关键词: poc.HTTP, HTTP请求发送
    rsp, req, err = poc.HTTP(rawRequest, poc.timeout(10))
    
    assert err == nil, f"basic HTTP request should not fail: ${err}"
}
```

### 关键词注释的位置
- **正确**：在功能代码块之前
- **正确**：在关键函数调用之前
- **错误**：只在文件头部添加
- **原因**：关键位置的注释让 grep 能快速定位到具体实现

## library-usage 目录规范

### 目录结构
- 每个库必须创建独立文件夹（不允许独立文件）
- 文件夹名与库名一致
- 主练习文件命名为 `{库名}-practice.yak`
- 可以包含辅助文件如 `{库名}-simple.yak`, `{库名}-advanced.yak`

### 文件内容结构（参考 poc-practice.yak 风格）
```yak
/*
库名称和简介
关键词: 核心关键词1, 核心关键词2, ...
*/

// 功能测试1
// 关键词: 具体功能, API名称
func testFeature1() {
    log.info("test feature 1")
    
    // 具体实现
    // 关键词: 细节功能, 使用场景
    result = library.Function()~
    
    assert result != nil, "result should not be nil"
    log.info("feature 1 test passed")
}

// 功能测试2
// 关键词: 具体功能, API名称
func testFeature2() {
    // 实现...
}

// 主函数执行所有测试
func main() {
    log.info("start library testing")
    
    testFeature1()
    testFeature2()
    
    log.info("all tests completed")
}

// 不需要"总结部分"，代码、注释和 assert 验证已经足够详细
```

### 测试要求
1. **超时控制**：所有测试必须在 10 秒内完成
2. **使用 assert**：关键结果必须用 assert 验证
3. **使用 timeout 参数**：网络请求等操作必须设置超时
4. **避免死循环**：循环必须有明确的终止条件
5. **错误处理**：使用 `~` 或 try-catch 处理可能的错误

### 示例：网络请求超时
```yak
// 正确：设置超时
rsp, req, err = poc.Get("http://example.com",
    poc.timeout(5)  // 5秒超时
)

// 错误：没有超时可能导致卡死
rsp, req, err = poc.Get("http://example.com")
```

## 查漏补缺流程

### 检查已有库的覆盖情况
1. 查看 `/Users/v1ll4n/Projects/yaklang/common/yak/script_engine.go` 中的所有 `yaklang.Import`
2. 对比 `library-usage` 目录下已有的库文件夹
3. 找出缺失的库并补充

### 学习库的使用方法
当不知道如何使用某个库时：
1. 搜索 yaklang 仓库中的测试用例：
   ```bash
   grep -r "库名." /Users/v1ll4n/Projects/yaklang --include="*_test.go"
   ```
2. 查看 yaklang 仓库中的示例代码
3. 使用 `desc(库名)` 在 yak 脚本中查看库的方法和属性

### 补充缺失库的步骤
1. 创建库文件夹：`library-usage/{库名}/`
2. 创建主文件：`library-usage/{库名}/{库名}-practice.yak`
3. 参考该库的 Golang 测试代码学习用法
4. 编写完整的功能测试，包含关键词注释
5. 使用 `yak {库名}-practice.yak` 验证脚本可执行
6. 确保所有 assert 通过且在 10 秒内完成

## 常见库的特殊注意事项

### poc 库
- HTTP 请求必须设置 `poc.timeout()`
- 使用 `poc.FixHTTPRequest()` 修复原始请求
- 使用 `poc.GetUrlFromHTTPRequest()` 提取 URL

### codec 库
- 某些函数返回字节数组（如 `codec.Sm3`, `codec.HmacSha256`）
- 需要使用 `codec.EncodeToHex()` 转换为可读字符串
- 注意区分编码解码函数的参数类型

### db 库
- 使用 `db.SetKey()` 和 `db.GetKey()` 进行键值存储
- 使用 `db.CreateTemporaryYakScript()` 创建临时插件
- 记得用 `defer db.DeleteYakScriptByName()` 清理资源

### hook 库
- 使用 `hook.NewMixPluginCaller()` 创建插件调用器
- 插件脚本需要定义回调函数如 `mirrorNewWebsitePath`
- 使用 `caller.LoadPlugin()` 加载插件

### 网络相关库（http, tcp, udp, dns 等）
- 所有网络操作必须设置超时
- 测试时考虑网络环境，允许某些测试失败
- 使用公共测试服务（如 httpbin.org）

### 加密相关库（codec, jwt, tls 等）
- 密钥长度要求严格（AES-128: 16字节，DES: 8字节等）
- 某些算法需要 IV（初始化向量）
- 国密算法（SM2, SM3, SM4）需要特别注明

### AI 相关库（ai, rag, aiagent 等）
- 可能需要外部服务支持
- 测试时注意 API 限流
- 敏感信息不要硬编码

## 文件组织最佳实践

### 一个库多个文件时的组织
```
library-usage/
  codec/
    codec-practice.yak       # 完整功能测试（主文件）
    codec-simple.yak         # 简单示例
    codec-crypto-random.yak  # 特定主题（如随机数生成）
    codec-tls-practice.yak   # 特定主题（如 TLS 证书）
    README.md                # 可选的说明文档
```

### 文件命名规范
- 主练习文件：`{库名}-practice.yak`
- 简单示例：`{库名}-simple.yak`
- 高级示例：`{库名}-advanced.yak`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yaklang/yaklang-ai-training-materials](https://github.com/yaklang/yaklang-ai-training-materials) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
