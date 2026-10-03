---
trigger: always_on
description: 把 [deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) (dsh) 搬到安卓上的项目, 用 Miuix 界面库, 自建虚拟屏来控制应用, 把屏幕能力做成 dsh 原生工具交给模型
---

# LittleWhale

把 [deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) (dsh) 搬到安卓上的项目, 用 Miuix 界面库, 自建虚拟屏来控制应用, 把屏幕能力做成 dsh 原生工具交给模型

**原来的八步计划与界面那一轮都已落地并在真机上验过**, 仓库公开在 `Miuzarte/LittleWhale` (`main` 是压缩后的单个提交, 开发历史在本地 `dev` 上)

## 文档地图

| 文档 | 放什么 |
| :-- | :-- |
| `AGENTS.md` (本文) | 现状、决策、约束、怎么操作 |
| `README.md` | 对外的门面: 能力、已知问题、构建、Credits |
| `docs/step2-record.md` … `docs/step8-record.md` | 每一步的清单、实测数字、踩坑过程 |
| `docs/ui-record.md` | 界面那一轮: 设置页缩进与名字、过渡风格、状态栏图标、截图那两条滑块 |
| `docs/host-build.md` | 构建事实: 随 APK 发的二进制、pack / zip、被否的方案、依赖缺口、工具链版本 (**平时不用读**) |
| `B:\Git\deepseek-harness\patch.md` | fork 相对上游的**全部**改动, 唯一权威 |
| `third_party/deepseek-harness/docs/` | dsh 自己的文档 (subsystems / cookbook / user) |

「第 N 步」的对应: 2 = dsh 上安卓, 3 = 特权通道, 4 = 自建虚拟屏, 5 = dsh 原生工具, 6 = 主屏与触摸刹车, 7 = 无障碍读屏, 8 = 端侧 OCR

## 仓库形态

dsh 以 **git submodule** 挂在 `third_party/deepseek-harness/`, 指向 `https://github.com/Miuzarte/deepseek-harness.git` (Miuzarte 的 fork, 不是上游)

**dsh 的构建产物不进 git**: submodule 里只有源码, `pnpm install` 与 `build:official` 都在**构建 APK 时**跑 (Gradle task), 产物直接喂给打包步骤, 这样 submodule 保持干净、能跟上游 rebase

构建机需要 Node + pnpm, 升级 dsh 就是 `git submodule update --remote` 之后重新构建。**pin 必须是 fork 上推过的提交**: 只活在 submodule 工作区里的改动, 克隆本仓的人看不见

## 架构

LittleWhale 自己是一台**远程 dsh 服务器 + 一个安卓控制端**, 两件事共用一个进程:

1. **dsh host** — APK 里的 Node 跑 dsh, 监听回环地址, 界面是它的 Web GUI (装进 WebView)
2. **安卓控制端** — Miuix 界面 + Shizuku / root 特权通道 + 自建虚拟屏, 把屏幕与输入能力做成 dsh 原生工具交给模型

**常驻方式是前台服务** (Node / host / WebView 都在里面), 否则一切后台就被系统收掉

**host 以「上游自带的 pack → 平铺 `npm install`」的形态进 APK**, 不用源码 + tsx —— 代价是改完 dsh 要重新 build + pack 才能进 APK, 没有"机内改源码立刻生效"

dsh 这个 fork 的定位也是**部署在远程服务器上, 从任意浏览器访问** (主题与字号存浏览器 `localStorage`, 手机与 PC 各一套), 移植时**不要把这些改动弄丢**

**监听地址没有写死 `127.0.0.1`**: 设置页「网络」的开关让 host 以 `--host 0.0.0.0 --allow-lan` 启动 (fork 加的第二个旗标, 安全默认一个字没变), 别的设备用浏览器打开 LAN URL; `DshHost.remoteUrl` 从就绪行的 `(LAN: …)` 后缀里取 URL, **不自己枚举网卡**, 这样显示的地址与 host 认的 browser-trust 栅栏是同一个

**安卓侧两个已知的坑**: `os.cpus().length` 返回 **0**, dsh 里任何按 CPU 数并行的地方都要能容忍 0; 随包发的 node 有一批**写死的 Termux 路径**, `OPENSSL_CONF` / `SHELL` / `TMPDIR` 三个少一个都起不来 (见 `docs/host-build.md`)

## 术语与参考仓库

- **dsh** — deepseek-harness, 被移植的对象, 以 submodule 挂在 `third_party/deepseek-harness/`, 是 **Miuzarte 的 fork** (不是上游)
- **SFA / ScrcpyForAndroid** — `B:\Git\ScrcpyForAndroid`, 同为 Miuzarte 的项目, **只当 Miuix 界面的参考**; 它的 scrcpy 层与 `new_display=` / `display_id=` / `scrcpy_%08x` socket / 控制报文表那套**全部作废**, 别再翻
- **MAA-Meow** — `B:\Git\MAA-Meow`, 自建虚拟屏 / 输入注入 / 读触摸 + Shizuku 与 root 双通道的**做法**参考 (**AGPL-3.0, 只看做法别抄代码**)
- **host / client 侧** — dsh 的术语, host 侧跑在 Node 里 (发构建产物 `lib/`), client 侧是浏览器产物 (每个 client 包的 `lib/client.js` + `@deepseek-ai/dsh-web-frontend/dist`)

| 路径 | 用途 |
| :-- | :-- |
| `third_party/deepseek-harness/` | **submodule**, 被移植的 dsh, 唯一允许改的第三方代码 |
| `B:\Git\deepseek-harness` | dsh fork 的开发克隆, 在 submodule 之外单独放一份方便比对和推分支 |
| `B:\Git\ScrcpyForAndroid` | Miuix 界面的参考 |
| `B:\Git\MAA-Meow` | 虚拟屏 / 注入 / 读触摸 + 双通道的参考 |

后两个仓库当**只读参考**用, 不要在里面改代码; dsh 不是参考而是**被移植的对象**

## 测试设备

一台随便用的真机, 小米 13, 已连接:

| 项 | 值 |
| :-- | :-- |
| 局域网 IP | `192.168.1.103` |
| adb | `adb connect 192.168.1.103:5555` |
| Termux ssh | `ssh -p 8022 192.168.1.103` (用户 `u0_a441`, 免密 key 已配好) |
| root | KernelSU, `su -c '...'` (`context=u:r:ksu:s0`) |
| 型号 / 代号 | `2211133C` / `fuxi` |
| Android | 16 (API 36) |
| ABI | `arm64-v8a` |
| 内存 | 11 GB |
| 已装 | Termux (含 clang, git, ssh, curl), Shizuku (`moe.shizuku.privileged.api`), KernelSU |

**这是开发机上唯一的一台, 可以随便装东西、重启服务、改配置**, 但 Termux 是 `targetSdk 28`, 别把它升级成 Play 版本; `adb` 在 `B:\Software\AndroidSDK\platform-tools\adb.exe`

## 代码风格

照抄 SFA 的 `AGENTS.md`:

- 注释中英文都行, 中英文/数字之间留空格 (盘古之白), **不要用全角标点**, 用半角 `, . ( ) /`
- **注释里不用句号**, 该断句的地方用逗号, 句末直接结束
- 这条同样管本文档的正文, 不只是代码注释
- UI 字符串同时进 `res/values/strings.xml` (en) 与 `res/values-zh/strings.xml`, **设置页也不例外**; 设置页的键以 `settings_` 开头, 措辞照 SFA 的 `values-zh/strings.xml`, 两份文件**键与顺序都保持一致**, 占位符 (`%1$s`) 一一对上
- 不要用 `;` 把本该分行的语句挤在一行
- 改代码优先小步修改, 不要整文件重写
- 引用符号先 `import` 再用短名, 不要写全限定名
- 多出来的 import 不用手动清, 格式化器会处理

## 界面

**上面原生渲染虚拟屏预览, 下面 `weight(1f)` 的 WebView 渲染 dsh Web GUI, 会话界面不重写** —— 那界面本来就有 ~40 个 client 包 (`packages/client/ui-*`) 且用的是**内部协议** (Host 生成 descriptors + codecs, 不是公开 API), 原生重写等于永久追上游, 详见 `docs/subsystems/web-client.md`

几条要记住的:

- **画面在 `TopAppBar` 下面**而不是页面最顶 (那样会顶进状态栏 inset 里); 手势层贴在 `SurfaceView` 的 modifier 上, 这样 `size` 就是画面本身
- **在看画面时顶栏整条不画** (`SmallTopAppBar` 的 `CollapsedHeight = 52.dp`, 标题空着也一样高, 想省这 52dp 只能不画), 菜单按钮改成浮在画面右上角, 静 3 秒淡出, **菜单开着时不淡出**
- **缩进与段间距统一由脚手架的 `LazyColumn` 给** (`scaffolds/LazyColumn.kt`: 页面左右 12dp + `itemSpacing` 12dp + 横屏限宽 + overscroll + 滚到底触感), **Card 不写水平外边距**; Miuix 的 `Card` 不带内边距, 带内边距的是设置项自己 (16dp), 所以**别再套一层 `padding(16.dp)`** (那就是 32dp); 按钮一律 `fillMaxWidth()`
- 过渡风格只有 `Miuix` / `AOSP` 两项, **没有 "无"**; AOSP 那套手感是搬来的 `ui/CrossActivityTransition.kt` (Miuix 0.9.4 的 `NavTransitions` 里没这个预设), 选中时 `cornerClipMode` 跟着换成 `All`
- 系统栏图标深浅由 `theme/SystemBars.kt` 按**实际渲染出来的配色**定, 在 `MiuixTheme` 里调一次; **顶栏没有模糊也没有那个选项** (画面自己不透明, 糊了没人看得见)
- **设置页右上角有一个 ⋮**: 要重启 host 才生效的改动 (工作区授权 / 局域网开关) 全收在那一个菜单里, 以后加选项就是往那个 `items` 里再加一条; 注意 **material3 不是本项目的依赖** (只有 `material3-window-size-class`), 没有 `androidx.compose.material3.DropdownMenu` 可用

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Miuzarte/LittleWhale](https://github.com/Miuzarte/LittleWhale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
