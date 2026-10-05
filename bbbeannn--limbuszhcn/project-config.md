---
trigger: always_on
description: 本文件只保留当前必须遵守的架构边界和调试入口。详细背景拆到:
---

# Limbus Company 汉化器 - 协作约定

本文件只保留当前必须遵守的架构边界和调试入口。详细背景拆到:

- `docs/project-architecture.md`: 模块、数据流、补丁管线
- `docs/container-runtime-notes.md`: VirtualApp 接入和设备实测
- `docs/runtime-debugging-notes.md`: 运行时排障记录
- `docs/commercial-container-reference.md`: 商业容器参考结论

方案级变更先更新本文件或对应 docs, 再改代码。

## 项目边界

- 目标游戏包名: `com.ProjectMoon.LimbusCompany`
- 汉化器包名: `com.example.limbuszhcn`
- 主路径: VirtualApp 容器模式。同步 Google Play 已安装游戏 split APK 到容器, 从容器内启动游戏, 再写容器内 `Localize/en/`。
- 禁止修改游戏 APK, 禁止 Frida 注入, 禁止 Shizuku 生产路径。
- App 模块: `:app`; 容器模块: `:virtualapp` -> `third_party/virtualapp-upstream/lib`。
- UI 不直接访问真实 `Android/data`; 安装、卸载、复原必须走 `GameStorage`。
- JSON 补丁只能结构化修改明确的显示文本字段。基础字段为
  `content/dialog/dlg/teller/name/nameWithTitle/desc/description/title/summary/flavor/place`;
  `story/relatedChapterText/openConditionNumber` 等新增显示字段也须加入共享的
  `TranslationTextPolicy.DISPLAY_FIELDS` 白名单,并同步到 native 对象字段接管层后才能启用。
  不改 `id`、`personalityid`、`voicefile`、`usage`、`model`、图标键等元数据。
- Token、签名口令、账号信息等敏感配置不得写死进源码或文档；汉化包只允许从公开的 GitHub Releases 获取。

## 容器硬约束

- 容器启动必须走 `VirtualCore.getLaunchIntent(packageName, 0)` + `VActivityManager.startActivity(intent, 0)`, 不得回退宿主 launcher。
- Limbus 可见 `ApplicationInfo` / `LoadedApk` 的 `sourceDir`、`splitSourceDirs`、`nativeLibraryDir`、`dataDir` 必须与 Google Play 真实安装源和容器重定向一致。
- Limbus 通用 PackageManager 查询默认隔离宿主环境; 宿主 Google 包只允许按白名单精确暴露。
- PlayCore asset pack 必须通过注入 `local_testing_dir` 走 `FakeAssetPackService`, 不得绑定宿主 Play Store `AssetModuleService`。
- GMS broker 请求必须通过 `ServiceConnectionDelegate` 改写残留调用包名, 避免 `Unknown calling package name 'com.ProjectMoon.LimbusCompany'`。该改写同时适用于宿主 GMS 和容器内安装的 microG,不得因 `com.google.android.gms` 已虚拟安装而跳过。
- Limbus Credentials `HiddenActivity` extras 含 GMS Parcelable 和 Binder, 不得经过 VAMS server 解包。必须在游戏进程内包装为当前 vpid 的 host stub Intent 并调用系统 `ActivityTaskManager`; 目标 Activity 仍由 `AppInstrumentation` 使用游戏 classloader 创建。该瞬态 Activity 没有 VAMS `ActivityRecord`, create/resume/finish/destroy 不得登记到 VAMS 任务栈, 但系统 token 和 Activity result 链路必须保留。
- 容器内 microG Services/FakeStore 与 Limbus 一样不得进入旧式 ART `jmethodID` 内存改写 VM hook；它们所需的 IO 重定向、Binder 身份和 Activity result 兼容均由独立层完成。Firebase Apple/验证码浏览器回调只允许由宿主精确接管 `genericidp://firebase.auth/` 与 `recaptcha://firebase.auth/`,再显式转发到容器内 Limbus 对应 Activity；不得把其它 URI 或浏览器 extras 带入容器。
- Limbus 原生 DNS 与 socket 连接必须绕过 VirtualApp 域名/IP 策略。允许有限日志诊断, 不得修改测试机 VPN、代理或路由。

## Native / AppSealing 约束

- Limbus native 终止路径只做容器进程内轻量拦截; `SIGPWR` / `SIGXCPU` 必须放行。
- `SIGSEGV` 仅可忽略 `si_code <= 0` 的用户主动投递; 同步内存错误必须交回原 handler。若下游 handler 返回后 PC/SP 均未改变,表示它没有消费当前同步故障,必须恢复默认处理并终止,不得返回原指令形成信号/日志死循环。
- libc hook 必须保持公开 API 返回值语义。例如 `getcwd()` 成功时返回调用方 buffer 指针, 不能暴露 raw syscall 长度。
- native 路径伪装必须覆盖 `readlinkat` / `syscall(__NR_readlinkat)` 和 `/proc/self/fd` 反向路径。
- AppSealing 50040 已确认由 `/proc/self/maps` 审计路径触发; `/sys/module`、`/proc/modules` 走 direct syscall, libc 文件 API 伪装不能覆盖。
- 只允许按反汇编或运行时 backtrace 确认的版本偏移跳过 50040/50048 上报调用; 不得泛化屏蔽 maps 解析、错误策略表或未知 syscall。
- AppSealing 50048 的 `SIGILL` 兜底只允许在 `ILL_ILLOPC`、PC=`base+0x4e564`、
  LR=`base+0x4e550` 同时匹配时跳到 `base+0x4e998`;其余 `SIGILL` 必须转发原 handler。
  该包装器须独立通过原始 `sigaction` 维护,不得扩展已被 inline hook 的 `sigaction` 函数体,
  否则会改变前导指令布局并导致 Android 12 在 `__fix_instructions` 崩溃。

## 当前调试状态

- 设备地址最近变动频繁; 以用户最新给出的 adb 地址为准。
- 用户已明确说明: 不要给测试机设置代理。可观察 VPN/网络状态, 不要修改。
- Redmi/Android 12 + Limbus v1.107.1 已验证容器可进入标题页。
- 历史黑屏根因: arm64 `getcwd` hook 返回 raw syscall 长度 `2`, 导致 il2cpp 执行 `strlen(0x2)`。
- 最近新增 Firebase C++ Auth shim 后, 容器已从黑屏推进到 `PlayVideo -> PlayWarningAnim -> PlayDone` 并显示标题页。
- Google 登录选择器可打开; 选择账号后仍需继续跟踪 Credentials result 回调和 Unity 登录状态。
- vivo/Android 13 的 Issue #1 已确认 Google 登录会启动 `:p1/:p2` microG 辅助进程,随后在 `libv++_64.so -> hookAndroidVM -> NativeEngine.launchEngine` 同步崩溃；Apple 登录同时缺失浏览器 `genericidp` 回调入口。修复后仍须在真机确认账号选择结果和 Apple OAuth 状态确实返回 p0 游戏进程。
- 游戏文本文件在进入主页面登录后才下载, 不要在标题页阶段期待 `Localize/en/` 已完整存在。
- Redmi/Android 12 + Limbus v1.109.1 已验证同步到容器 versionCode 450 后可再次走完
  `PlayVideo -> PlayWarningAnim -> PlayDone`,并恢复 `SetLoginInfo : GOOGLE`。若 APK
  `sourceDir` 已是 450、Virtual PackageManager 仍报告 448,AppSealing 会在启动期报
  `20033`;必须先同步宿主安装,不能把它作为新的终止路径特判。
- Redmi/Android 12 已验证统一 service bind 包装会命中 microG
  `MeasurementService`:依次出现 `bindIsolatedService intercept`、
  `Wrap GMS service broker`、`Created explicit Limbus GMS broker proxy` 和
  `Rewrite Limbus GMS GetServiceRequest`,随后仍可进入 `PlayDone`。其中
  `libcovault-appsec.so +0x4e564` 的 `SIGILL` 被放行后游戏继续运行,不得把它单独作为
  GMS broker 修复失败的判据。Android 13/MIUI 设备仍需用相同四条日志复测。
- 运行时文本索引已切换为日语单源 schema 7。schema 6 的版本 2026071001 曾生成
  93488 条完整映射、19659 条幂等短术语映射和 151950 条源记录;schema 7 在此基础上
  统一编译端/native 显示字段、覆盖 `story/relatedChapterText/openConditionNumber`,并拒绝
  Unicode 替换字符、非法控制字符等确定损坏的译文。Limbus 1.109.1 可解析 149 个 IL2CPP
  assembly。接管必须在访客配置阶段挂接 `dlsym`,只把 Unity 请求的 `il2cpp_init`
  返回值替换为生命周期包装函数;原初始化返回后才解析 domain 和安装文本 hook。不得
  inline patch `il2cpp_init`,不得并发读取半初始化 domain,也不得用固定启动秒数补偿。
- `TMP_Text.set_text`、`SetText(String)`、`SetText(String, Boolean)` 与安全的
  `UnityEngine.UI.Text.set_text` 已安装运行时替换 hook。内置 `ChineseFont.ttf`
  注册为全局 fallback;只有能证明由接管层新建的中文托管字符串,才可在赋值期间
  把当前 TMP 组件切换到该中文主字体,以避免日文主字体与粗体 fallback 逐字混排;
  组件复用于非托管文本时必须恢复原主字体,不得按“含非 ASCII”泛化替换所有 TMP。
  内置主字体使用 Sarasa Gothic SC Regular,动态 SDF 图集必须使用 4096x4096,1024x1024
  已实测会在技能页加载约千条文本后丢失旧字形。切换中文字体时须保存并恢复组件原字体

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BBBEANNN/limbusZhCN](https://github.com/BBBEANNN/limbusZhCN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
