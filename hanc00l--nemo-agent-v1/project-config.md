---
trigger: always_on
description: 基于 Claude Code 的自动化渗透测试 Agent，达到中高级网络安全专家水平。
---

# CLAUDE.md

## 项目概述

基于 Claude Code 的自动化渗透测试 Agent，达到中高级网络安全专家水平。

**设计哲学**: "Intent → Code → Execute → Record → Result"

## 标准作业流程

```
0. 读取笔记 → note.get_notes_summary(challenge_code)
1. 手动侦察 → 浏览器访问、源码分析
2. 主动侦察 → nmap、katana、observer_ward、whatweb、fscan
3. 查询知识库（任何工具识别到应用时）
   ├─ 识别到应用指纹？
   │  ├─ observer_ward → "信呼OA"、"泛微OA"
   │  ├─ whatweb → "Apache Struts"
   │  ├─ fscan title → "Tomcat"、"Nginx"
   │  ├─ nuclei → CVE 编号
   │  ├─ 手动发现 → 页面特征、响应头
   │  └─ 任何来源？
   │     ├─ 是 → 查询 vulnerability-wiki
   │     │  ├─ 找到？ → 获取详情 → 继续测试
   │     │  └─ 未找到 → nuclei 扫描 → WebSearch 兜底
   │     └─ 否 → 跳过此步骤
4. 漏洞测试 → XSS/SQLi/IDOR/SSTI/命令注入/SSRF
5. 漏洞利用 → 获取 FLAG
   ├─ SSRF 确认？ → auxiliary/cloud/cloud-metadata → 云凭证
   ├─ 云指标命中？ → 云安全工具（lc/cloudsword/cf）
   ├─ AI 指标命中？ → auxiliary/ai-security/ 技能（提示注入）
   └─ 常规 Web 利用
6. 立即提交 → competition.submit_answer()
7. 保存结果 → note.append_note("result", flag)
```

## 核心概念

### challenge_code

题目的唯一标识符，关联 Jupyter 会话和笔记存储。来源：竞赛平台 / 用户指定 / URL 生成

### Note 笔记系统

| 类型 | 文件 | 用途 |
|------|------|------|
| info | `{code}-info.md` | 信息收集 |
| infer | `{code}-infer.md` | 推理分析 |
| result | `{code}-result.md` | 最终结果 |

API:
- `get_notes_summary(code)` - 读取摘要
- `append_note(code, type, content)` - 追加笔记

笔记存储路径由 `NOTE_PATH` 环境变量配置，容器内默认 `/opt/notes`。

### Competition 平台 API

| 函数 | 用途 |
|------|------|
| `get_challenges()` | 获取所有挑战 |
| `get_target_url(code)` | 获取目标 URL |
| `get_hint(code)` | 获取提示（扣分） |
| `submit_answer(code, answer)` | 提交 FLAG（参数名是 answer 不是 flag） |

**频率控制**: 平台限制 ≤3 req/s。`PlatformClient._rate_limit` 内置 0.5s 间隔保护，429 响应自动重试（最多 3 次）。

### Browser 浏览器工具

Playwright 自动化：页面访问、交互、截图、JS 执行

```python
page = await toolset.browser.get_page()
await page.goto("http://target")
content = await page.content()
```

### Terminal 终端工具

基于 tmux 的命令执行：长时间运行、实时输出、超时控制

```python
sid = toolset.terminal.new_session()
toolset.terminal.send_keys(sid, "nmap -sV target", enter=True)
time.sleep(30)
output = toolset.terminal.get_output(sid)
```

## 安全工具

### 信息收集

| 工具 | 来源 | 用途 | 命令 |
|------|------|------|------|
| nmap | apt | 端口扫描 | `nmap -sV -n -T4 --open target` |
| whatweb | apt | 技术栈识别 | `whatweb -a 3 http://target` |
| observer_ward | /opt/workspace | 应用指纹识别 | `observer_ward -t http://target` |
| katana | /opt/workspace | 网页爬取 | `katana -u http://target -d 3 -jc` |
| ffuf | /opt/workspace | 目录发现/模糊测试 | `ffuf -u 'http://target/FUZZ' -w wordlist` |
| fscan | /opt/workspace | 内网综合扫描 | `fscan -h 10.10.1.0/24` |
| lc | /opt/workspace/lc | 多云攻击面资产梳理 | `lc -ep -s` |

### 漏洞利用

| 工具 | 来源 | 用途 | 命令 |
|------|------|------|------|
| sqlmap | apt | SQL 注入 | `sqlmap -u "http://target/page?id=1" --batch` |
| nuclei | /opt/workspace | 模板化漏洞扫描 | `nuclei -u http://target` |
| xray | /opt/workspace/xray | 被动代理漏洞扫描 | `xray webscan --listen 127.0.0.1:7777 --json-output xray.json` |
| msfconsole | apt (omnibus) | 漏洞利用框架 | `msfconsole` |
| hydra | apt | 暴力破解 | `hydra -l user -P pass.txt target ssh` |
| hashcat | apt | 密码破解 | `hashcat -m 0 hash.txt wordlist` |
| cloudsword | /opt/workspace/cloudsword | 云安全综合测试 | `cloudsword` |
| cf | /opt/workspace/cf | 云环境利用框架 | `cf` |

### Java 反序列化

| 工具 | 来源 | 用途 |
|------|------|------|
| JNDIExploit | /opt/workspace/JNDIExploit/ | JNDI 注入利用 |
| JYso | /opt/workspace/JYso/ | Java 反序列化利用 |
| shiro_cli | /opt/workspace/shiro/ | Shiro 反序列化 |
| ysoserial | /opt/workspace/ysoserial/ | Java 原生反序列化 Payload 生成 |
| marshalsec | /opt/workspace/marshalsec/ | Java Marshalling 漏洞 + JNDI/RMI/LDAP 引导服务 |

### 容器与编排

| 工具 | 来源 | 用途 |
|------|------|------|
| docker | /opt/workspace/docker/ | 容器操作（逃逸检测、镜像审计、挂载探测） |
| kubectl | /opt/workspace/kubectl/ | K8s 集群交互（枚举资源、窃取凭证、提权） |

### 内网渗透工具

| 工具 | 来源 | 用途 |
|------|------|------|
| frpc/frps | /opt/workspace/frp/ | 反向代理（首选） |
| chisel | /opt/workspace | HTTP 隧道代理 |
| Stowaway | /opt/workspace/Stowaway/ | 多级节点代理 |
| Neo-reGeorg | /opt/workspace/Neo-reGeorg/ | HTTP 隧道 |
| nxc (NetExec) | /opt/workspace/NetExec/ | 横向移动（SMB/SSH/WinRM） |
| mimikatz | /opt/workspace | Windows 凭证提取 |

### Webshell / 其他

| 工具 | 来源 | 用途 |
|------|------|------|
| weevely | apt | PHP Webshell 生成 |
| wsh | /opt/workspace | Webshell 管理 |
| rem | /opt/workspace/rem | 漏洞利用 |
| proxychains4 | apt | 代理链 |

**字典**: `/opt/workspace/SecLists/Discovery/Web-Content/`

### 外部知识库

| 工具 | 来源 | 用途 |
|------|------|------|
| vulnerability-wiki | skills/pentest/vulnerability-wiki/ | 漏洞知识库（1123+漏洞），本地文件读取 |
| vulhub | skills/pentest/vulhub/ | 317 漏洞环境知识库（本地 JSON 索引） |

**vulnerability-wiki**:
- 位置: `~/.claude/skills/pentest/vulnerability-wiki/`
- 功能: 1123 个漏洞知识库，本地文件读取（无需容器/Web 服务）
- 使用场景: observer_ward 识别出应用后，查询相关漏洞

```python
# 查询函数定义见: skills/pentest/vulnerability-wiki/SKILL.md

# 按应用模糊搜索（返回所有匹配）
results = search_by_app("ThinkPHP")  # 返回列表

# 按 CVE 精确搜索
result = search_by_cve("CVE-2022-22963")  # 返回单条

# 读取漏洞文件
detail = read_vuln_file("framework/ThinkPHP5-5.0.23-远程代码执行漏洞.md")
```

## 赛区策略

> 赛区策略为累积递进，所有赛区均可使用全部工具，赛区仅影响攻击思路和技能优先级。

| 赛区 | 优先攻击 | 附加技能 |
|------|----------|----------|
| Zone 1 | Web 漏洞 | auxiliary/exploit（22种攻击方法）, sqlmap, xray |
| Zone 2 | Zone1 + CVE/云/AI | nuclei, vulhub, vulnerability-wiki, auxiliary/cloud, auxiliary/ai-security |
| Zone 3 | Zone1+2 + 内网 | fscan, netexec, mimikatz, frp, stowaway, auxiliary/lateral, auxiliary/postexploit, 多级代理 |

## 技能索引

### 核心技能

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hanc00l/nemo-agent-v1](https://github.com/hanc00l/nemo-agent-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
