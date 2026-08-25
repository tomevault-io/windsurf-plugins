---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

给 Antigravity 2.0 做界面汉化。核心思路是**运行时注入**：以 `--remote-debugging-port` 启动
Antigravity，连 Chrome DevTools 协议（CDP）把一段 MutationObserver 翻译引擎注入渲染页面。
**不修改任何安装文件**（不碰 `Antigravity.exe`、`app.asar`、`product.json`），因此没有还原逻辑，
官方更新也不会破坏安装。

依赖：Python 3 + `websockets`（本机实测 Python 3.14.7 / websockets 16.0）。无构建、无测试框架。

### 为什么不能沿用 1.x 的做法

上游 `github.com/cshitian/antigravity_chinese` 针对 Antigravity 1.x：向
`resources\app\out\vs\...\workbench.html` 注入 `<script>` 并同步 `product.json` checksum。
2.0 把 `resources/app` 打包成了 `app.asar`，界面改由本地 HTTPS 服务渲染
（`https://127.0.0.1:<随机端口>/`），那个注入点在文件系统层面已不存在，且 `inject_html()`
对不存在的文件**静默返回 False**。上游 Issue #4 记录了这个失效，作者无回应。

## 常用命令

```bash
python hanhua_v2.py                      # 自动探测 + 启动 + 注入（qidong.bat / qidong_mac.sh 就是调它）
python hanhua_v2.py --no-launch          # 只注入到已带调试端口运行的实例（开发时最常用）
python hanhua_v2.py --watch 0            # 关掉守护，注入完立刻退出
python hanhua_v2.py --install-dir "D:\Antigravity" --port 9333
python hanhua_v2.py --install-dir "/Applications/Antigravity.app"
python caiji.py                          # 抓当前页面未翻译英文 → pending.json
```

改完代码后的自查（没有测试框架，靠这几条）：

```bash
python -c "import ast,io;[ast.parse(io.open(f,encoding='utf-8').read()) for f in ['hanhua_v2.py','caiji.py']]"
python -c "import json,io;[json.load(io.open(f,encoding='utf-8')) for f in ['dicts/common.json','dicts/ui_v2.json']]"
# 引擎 JS 必须单独校验：它是 Python 字符串拼出来的，语法错在注入时才会暴露
python -c "import importlib.util,io;s=importlib.util.spec_from_file_location('h','hanhua_v2.py');h=importlib.util.module_from_spec(s);s.loader.exec_module(h);io.open('/tmp/e.js','w',encoding='utf-8').write(h.build_engine_js(h.load_dictionary()))"
node --check /tmp/e.js
python -c "b=open('qidong.bat','rb').read();assert not [x for x in b if x>127],'bat 混入非 ASCII'"
```

## 架构

```
hanhua_v2.py     主脚本：加载字典 → 生成引擎 JS → 启动 Antigravity → CDP 注入 → 守护
  ├ load_dictionary()   合并 dicts/*.json，对 key 做 normalize_text()
  ├ build_engine_js()   把字典嵌进一段 IIFE 模板字符串（整个翻译引擎都在这里面）
  ├ wait_for_debug_port() 轮询 /json 找主界面页
  └ cdp_inject()        等就绪 → 注册 document-start 脚本 → 注入 → 守护补注
caiji.py         界面文本采集器（CDP 抓页面英文 → pending.json；--misses 拉取引擎未命中日志 → misses.json）
qidong.bat           Windows 一键启动器（taskkill → 调 hanhua_v2.py）
qidong_mac.sh        macOS 一键启动器（killall → 调 hanhua_v2.py）
qidong_mac.command   macOS 双击入口
dicts/*.json         字典，格式 {"英文原文": "中文译文"}
```

**注意 `build_engine_js()` 的返回值是字符串**：整个翻译引擎（约 300 行 JS）以 Python 三引号
字符串形式存在于 `hanhua_v2.py` 里，靠 `DICT_PLACEHOLDER` 占位替换注入字典。改 JS 时
Python 的字符串转义层会介入，见下文「转义陷阱」。

**平台分支**：`candidate_paths()` / `launch_antigravity()` / `is_antigravity_running()` 三处按
`sys.platform` 分 Windows / macOS 两支，加平台相关逻辑时三处要同步改。macOS 支持来自
PR #1（贡献者真机验证），维护侧没有 Mac：改 darwin 分支时用「伪造 `sys.platform` + 假 `.app`
目录树」验证纯逻辑，Windows 侧则必须真机跑完整流程确认无回归。

### 注入时机 —— 冷启动能否成功的关键

**端口开放 ≠ 界面就绪。** 冷启动时 CDP 端口比 SPA 挂载早得多，此时页面 `title` 还是空的，
直接注入会被随后的加载/导航**整个冲掉**，现象是脚本打印「[成功]」但界面全是英文。
`cdp_inject()` 因此分三步：

1. **等就绪** —— 轮询 `document.readyState === 'complete'` 且 `body.innerText` 超过 100 字符
2. **注册 document-start 脚本** —— `Page.addScriptToEvaluateOnNewDocument`。
   这个注册**只在 CDP 连接存活期间有效**，连接一断浏览器就清掉它，所以它只是加速手段，不是保险
3. **守护补注** —— 每 1.5 秒查 `window.__ag_hanhua_engine__`，被冲掉立刻补注，默认 30 秒。
   这一步才是真正的保险

`main()` 外层还包了 3 次重试：早期 page target 可能在加载途中被丢弃，WebSocket 会直接断开。

### 翻译引擎

**匹配优先级（顺序不能动）**：

1. **整节点精确匹配** —— 优先级最高。句子里提到 Antigravity / MCP / Google AI 等产品名也照翻，
   `PROTECTED` 只管子串替换
2. **大小写归一匹配** —— 限 30 字符以内，防误匹配长句
3. **子串替换 `smartReplace()`** —— 仅 `phraseEntries`（key ≥ 30 字符的完整句子）参与，带词边界检查
4. **正则规则 `REGEX_RULES`** —— 带变量的动态文案（时间、模型名、邮箱、按键提示）

**监听策略**：`MutationObserver` 同时听 `childList` + `characterData` + `attributes`。
`characterData` **必须听**——React 重渲染直接改写文本节点，不听就会被改回英文。
另有每 3 秒一次的全量重扫兜底（下拉菜单等临时弹出内容靠它）。

**性能**：`lastSeen` WeakMap 记住每个文本节点上次处理过的值，没变就跳过。这是关键优化——
没有它，每 3 秒的全量重扫都要对每个节点做一遍昂贵匹配。

实测占用（1125 条词条、667 个 DOM 元素）：JS 堆净增 586 KB（≈ Antigravity 总内存 640 MB 的 0.09%）；
全量重扫 0.4~0.7 ms；11 秒采样期内主线程长任务 0 次。
**量堆增量别直接比前后差值**——应用自身堆在 52~55 MB 间波动，几百 KB 会被噪声淹没；
正确做法是在页面里再构造一份等价结构、前后立刻取样取差值。

### 三条容易改错的不变量

**1. `norm()` 必须与 `normalize_text()` 逐条对应。**
字典 key 在加载时被归一化（压缩空白、弯引号 `’‘“”` 转半角）。页面文本不做同样处理就永远匹配不上。
界面里的 `Don't`、`Let's`、`agent's` 用的是 **U+2019 弯撇号**，不是 ASCII `'`。
曾出过 bug：JS 侧写成 `.replace(/[']/g, "'")`，字符类里只有 ASCII 引号，等于拿直引号换直引号——
纯空操作，所有含撇号的句子静默漏翻。

**2. 禁区类名分两类匹配，选错会造成方向相反的两种事故。**

| 类别 | 匹配方式 | 选错的后果 |
|---|---|---|
| `BLOCKED_CLASS_SUBSTR`（`monaco-editor`、`code-block` 等多词类名） | 子串 | —— 足够特异，不会误伤 |
| `BLOCKED_CLASS_TOKEN`（`terminal`、`preview` 等通用单词） | class token 精确 | 用子串会把 `terminal-settings-row` 这类正常设置容器整块判成禁区，整片界面漏翻 |

`isInBlockedZone()` 向上回溯 **25 层**（不是 12）：Tailwind/React 嵌套很深，12 层够不到上层的
禁区容器，用户的代码/会话内容就会漏出来被翻译。这段只对「值变过」的节点跑，代价可忽略。

会话/项目列表项的识别特征必须够精确：`DIV` + `relative` + `w-full` + `select-none` + `cursor-pointer`。
只判 `select-none cursor-pointer` 会误伤菜单栏按钮（File/View/Window），导致它们不翻译。

**3. `attachShadow` 劫持只做一次，回调动态读全局引擎，不能闭包捕获 observer。**
闭包捕获会让每次重复注入都永久持有一份旧引擎（连同整份字典约 0.6 MB），
而守护期会反复补注，泄漏不断叠加。当前实现用 `__ag_hooked__` 标记只劫持一次，
回调里读 `window.__ag_hanhua_engine__.observe`。实测连注 5 次仅增长 561 KB（≈ 一份字典）。
旧引擎 disconnect 时还要置 `stopped` 标记：document-start 路径下 `startEngine` 尚未成功，

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nianshou555qiansui/antigravity_chinese_v2](https://github.com/nianshou555qiansui/antigravity_chinese_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
