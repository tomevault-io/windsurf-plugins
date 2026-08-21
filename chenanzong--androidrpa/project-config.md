---
trigger: always_on
description: 使用python开发自动化脚本，要求ROOT权限或者SHELL权限, 这是脚本引擎的工作进程yyds-android\app\src\main\java\pyengine\Main.java
---

安卓平台RPA脚本开发
使用python开发自动化脚本，要求ROOT权限或者SHELL权限, 这是脚本引擎的工作进程yyds-android\app\src\main\java\pyengine\Main.java
包含安卓App、VSCode开发插件、MCP Server、多设备控制台、Python模版工程

## 项目目录结构

```
RPA/
├── AGENTS.md                           # 项目摘要文档
├── yyds-android/                       # 安卓App主工程
│   └── app/src/main/
│       ├── AndroidManifest.xml         # 清单文件（权限、组件声明）
│       ├── jni/                        # Native层
│       │   ├── CMakeLists.txt          # 构建配置（libai.so + yyds_keep + script_crypto + cpython_bridge）
│       │   ├── keeper.cpp              # yyds.keep native守护进程（独立可执行文件）
│       │   ├── cpython_bridge.cpp      # CPython JNI桥接（嵌入式Python调用）
│       │   ├── script_crypto.c         # 脚本加密白盒密钥派生（AES-256-GCM）
│       │   ├── main.cpp / image.cpp    # 图像处理、OCR、YOLO推理
│       │   └── ncnn/                   # NCNN推理库（OCR/YOLO模型）
│       └── java/
│           ├── pyengine/               # Python脚本引擎核心
│           │   ├── Main.java           # 工作进程入口（ROOT/SHELL权限运行）
│           │   ├── PyEngine.kt         # Python引擎管理（初始化、启动/停止项目、pip路径）
│           │   ├── CPythonBridge.kt    # CPython JNI桥接Kotlin封装（替代Chaquopy）
│           │   ├── PyProcess.kt        # 独立子进程运行单个项目（进程隔离）
│           │   ├── EngineClient.kt     # App进程与工作进程通信客户端（HTTP REST + WebSocket日志流）
│           │   ├── EngineProtocol.kt   # RPC方法常量与数据键定义
│           │   ├── WebSocketAsServer.kt# 工作进程HTTP/WebSocket服务（Ktor CIO, 端口61140, 全部REST API）
│           │   ├── WebSocketAsClient.kt# 连接yyds-con控制台的WebSocket客户端（设备端）
│           │   ├── WebRtcDataChannel.kt# WebRTC DataChannel存根（P2P屏幕流，待启用）
│           │   ├── HandleApiServerConnection.kt # 公网服务器WebSocket RPC处理（旧远程控制）
│           │   ├── YyProject.kt        # 脚本项目数据模型与扫描逻辑
│           │   ├── YyProjectUtil.kt    # 项目ZIP解压工具
│           │   ├── ApkPackageHelper.kt # APK打包助手（脚本→独立APK，V1签名）
│           │   ├── ApkV1Signer.java    # APK V1签名实现
│           │   ├── BinaryXmlEditor.java# Android二进制XML编辑（修改resources.arsc）
│           │   ├── ScriptEncryptor.kt  # 脚本加密器（AES-256-GCM + 白盒密钥 + PBKDF2）
│           │   ├── PyOut.kt            # 日志输出管理（Java侧队列，Python通过JNI回调）
│           │   ├── RpcDataModel.kt     # RPC数据模型（JSON序列化）
│           │   ├── ContextUtil.java    # 系统Context工具 + native库路径注册
│           │   ├── ShareReflectUtil.java # 反射工具
│           │   └── ZipUtility.java     # 压缩解压工具
│           ├── com/tencent/yyds/       # App UI层
│           │   ├── App.kt              # Application
│           │   ├── MainActivity.kt     # 主Activity（导航、权限、侧边栏）
│           │   ├── frag/
│           │   │   ├── HomeFragment.kt     # 首页
│           │   │   ├── ScriptFragment.kt   # 脚本项目列表页
│           │   │   └── RemoteFragment.kt   # 远程控制页
│           │   ├── PackageActivity.kt      # APK打包配置页（应用名、图标、包名、运行行为）
│           │   ├── RunnerActivity.kt       # Runner模式Activity（打包后APK的控制中心）
│           │   ├── PipManagerActivity.kt   # Pip包管理器（搜索PyPI、安装/卸载/升级）
│           │   ├── FileBrowserActivity.kt  # 文件浏览器（ROOT权限浏览设备文件系统）
│           │   ├── LogcatActivity.kt       # 运行日志页
│           │   ├── ProjectConfigActivity.kt# 项目配置页
│           │   ├── FloatingWindowService.kt# 悬浮窗服务（脚本控制+UI布局检查器）
│           │   ├── FloatingLogService.kt   # 悬浮日志控制台服务
│           │   ├── ShizukuUtil.kt          # Shizuku免ROOT启动引擎
│           │   ├── YypListAdapter.kt       # 项目列表RecyclerView适配器
│           │   ├── YypListViewHolder.kt    # 项目列表ViewHolder
│           │   ├── inspector/              # UI布局检查器
│           │   │   ├── UiInspectorView.kt  # 控件树覆盖绘制视图（Canvas）
│           │   │   ├── UiNode.kt           # 控件节点数据模型
│           │   │   └── NodeTreeAdapter.kt  # 控件树列表适配器
│           │   └── widget/
│           │       └── AppBanner.kt        # 应用横幅组件
│           ├── common/                 # 公共组件
│           │   ├── BootService.kt      # 前台核心服务（通知控制、项目切换）
│           │   ├── BootReceiver.kt     # 开机广播接收器
│           │   └── BootProvider.kt     # ContentProvider
│           ├── uiautomator/            # 自动化引擎API
│           │   ├── AppProcess.java     # 进程管理（app_process启动命令、native keeper路径）
│           │   ├── ExportApi.java      # 自动化引擎API导出
│           │   ├── ExportHandle.java   # 引擎通信句柄
│           │   ├── ExportHttp.java     # HTTP接口导出（JSON参数转换）
│           │   ├── ExtSystem.java      # 系统Shell工具
│           │   └── tool/               # 截图、显示控制等工具
│           ├── image/                  # 图像处理（OCR、颜色识别）
│           ├── noadb/                  # 屏幕编解码
│           └── scrcpy/                 # 剪贴板、显示辅助
├── CPython-android/                    # CPython交叉编译与嵌入式桥接
│   ├── scripts/build-cpython.sh       # WSL交叉编译CPython脚本（--enable-shared）
│   ├── build.bat                      # Windows一键编译入口
│   ├── python-shims/                  # Python兼容层模块（打包进APK assets）
│   │   ├── pyengine.py               # PyOut回调shim（JNI _yyds_bridge C扩展）
│   │   ├── entry.py                  # 脚本入口（子进程模式：加载并运行用户项目）
│   │   └── _android_bootstrap.py     # Android适配层（SSL/locale/tempfile/HOME等）
│   ├── runtime/                       # 纯Python运行时模块（替代部分Kotlin逻辑的Python实现）
│   │   ├── server.py                 # aiohttp HTTP+WS服务器（替代Ktor，端口61140）
│   │   ├── project_manager.py        # 项目管理（扫描/启动/停止/运行代码片段）
│   │   ├── auto_engine_proxy.py      # yyds.auto自动化引擎HTTP代理

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChenAnZong/AndroidRPA](https://github.com/ChenAnZong/AndroidRPA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
