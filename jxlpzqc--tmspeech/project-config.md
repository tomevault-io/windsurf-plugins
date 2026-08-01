---
trigger: always_on
description: TMSpeech 是一个基于 .NET 6 的 Windows 实时语音识别和字幕展示应用，采用插件化架构设计。项目使用 Avalonia UI 框架构建跨平台界面，集成 sherpa-onnx 进行离线语音识别。
---

# TMSpeech 架构文档

## 项目概述

TMSpeech 是一个基于 .NET 6 的 Windows 实时语音识别和字幕展示应用，采用插件化架构设计。项目使用 Avalonia UI 框架构建跨平台界面，集成 sherpa-onnx 进行离线语音识别。

注意，尽量不要搜索代码文件，而是找到相关的文件并阅读整个文件。

## 项目结构

项目采用三层架构设计，分为三个主要子项目：

```
TMSpeech/
├── src/
│   ├── TMSpeech.Core/          # 核心业务逻辑层
│   ├── TMSpeech.GUI/           # 用户界面层
│   ├── TMSpeech/               # 应用程序入口
│   └── Plugins/                # 插件实现
│       ├── TMSpeech.AudioSource.Windows/
│       └── TMSpeech.Recognizer.SherpaOnnx/
```

## 三个子项目的关系

### 1. TMSpeech.Core (核心层)

**职责**：提供核心业务逻辑和基础设施

**主要功能**：
- 插件系统基础架构
- 配置管理
- 任务调度
- 资源管理
- 通知服务

**源码文件清单**：

#### 1.1 插件系统 (Plugins/)

| 文件路径 | 作用 | 关键成员 |
|---------|------|---------|
| `Plugins/IPlugin.cs:9-26` | 插件基础接口定义 | GUID, Name, Version, Author, Available, Init(), Destroy(), CreateConfigEditor(), LoadConfig() |
| `Plugins/IRunable.cs:4-8` | 可运行接口，定义生命周期 | Start(), Stop(), ExceptionOccured 事件 |
| `Plugins/IAudioSource.cs:14-19` | 音频源插件接口 | StatusChanged, DataAvailable 事件，继承 IPlugin 和 IRunable |
| `Plugins/IRecognizer.cs:26-36` | 识别器插件接口 | TextChanged, SentenceDone 事件，Feed() 方法接收音频数据 |
| `Plugins/ITranslator.cs` | 翻译器插件接口（未实现） | Translate() 方法 |
| `Plugins/IPluginConfigEditor.cs` | 插件配置编辑器接口 | GetFormItems(), GetAll(), SetValue(), GenerateConfig(), LoadConfigString() |
| `Plugins/PluginConfigFormItem.cs` | 插件配置表单项定义 | PluginConfigFormItemText, PluginConfigFormItemOption, PluginConfigFormItemFile 等多种表单项类型 |
| `Plugins/PluginManager.cs:194-230` | 插件管理器，动态加载和管理插件 | LoadPlugins(), Plugins, AudioSources, Recognizers, Translators 属性，PluginLoadContext 内部类处理程序集加载 |

#### 1.2 配置管理

| 文件路径 | 作用 | 关键成员 |
|---------|------|---------|
| `ConfigManager.cs:70-192` | 配置管理基类和本地实现 | Apply(), Get(), BatchApply(), Load(), Save(), ConfigChanged 事件 |
| `ConfigTypes.cs` | 配置键定义和默认值 | GeneralConfigTypes, AppearanceConfigTypes, NotificationConfigTypes, AudioSourceConfigTypes, RecognizerConfigTypes |

#### 1.3 任务管理

| 文件路径 | 作用 | 关键成员 |
|---------|------|---------|
| `JobManager.cs:55-277` | 任务管理器，协调音频源和识别器的工作流程 | Start(), Pause(), Stop(), Status 属性，TextChanged, SentenceDone 事件，敏感词检测，日志记录 |

#### 1.4 资源管理 (Services/Resource/)

| 文件路径 | 作用 | 关键成员 |
|---------|------|---------|
| `Services/Resource/ResourceManager.cs:50-178` | 资源管理器，管理插件和模型资源 | GetLocalResources(), GetAllResources(), RemoveResource(), 扫描本地和远程资源 |
| `Services/Resource/ModuleInfo.cs:11-109` | 模块元数据定义 | ID, Version, Name, Type, Assemblies, InstallSteps, SherpaOnnxModelPath |
| `Services/Resource/DownloadManager.cs` | 下载和安装管理器 | StartJob(), PauseJob(), DoDownload(), DoExtract(), DoWriteFile()，支持多任务队列 |

#### 1.5 通知服务 (Services/Notification/)

| 文件路径 | 作用 | 关键成员 |
|---------|------|---------|
| `Services/Notification/INotificationService.cs` | 通知服务接口 | Notify() 方法，NotificationType 枚举 |
| `Services/Notification/NotificationManager.cs` | 通知管理器（单例） | RegistService(), Notify(), SetNotifyLevel() |

#### 1.6 自动更新 (Services/AutoUpdate/)

| 文件路径 | 作用 | 关键成员 |
|---------|------|---------|
| `Services/AutoUpdate/AutoUpdateManager.cs` | 自动更新管理（TODO 未实现） | CheckUpdate() |

#### 1.7 工具类 (Utils/)

| 文件路径 | 作用 | 关键成员 |
|---------|------|---------|
| `Utils/SystemObjectNewtonsoftCompatibleConverter.cs` | JSON 转换器，兼容 Newtonsoft.Json | JsonConverter<object> 实现，处理基本类型和 JsonElement |

**依赖关系**：
- 无依赖其他子项目
- 依赖外部包：Downloader、SharpCompress

### 2. TMSpeech.GUI (界面层)

**职责**：提供用户界面和交互体验

**主要功能**：
- 主窗口（实时字幕显示）
- 配置窗口
- 历史记录窗口
- 托盘菜单
- 插件配置界面动态生成

**源码文件清单**：

#### 2.1 应用程序入口

| 文件路径 | 作用 | 关键成员 |
|---------|------|---------|
| `App.axaml.cs:22-82` | Avalonia 应用程序入口 | Initialize() 初始化配置，OnFrameworkInitializationCompleted() 加载插件和启动识别，UpdateTrayMenu() |
| `App.axaml` | 应用程序资源定义 | 样式、主题、全局资源 |
| `DefaultConfig.cs` | 生成默认配置 | GenerateConfig() 合并各配置类型，设置默认音频源和字体 |

#### 2.2 视图模型 (ViewModels/)

| 文件路径 | 作用 | 关键成员 |
|---------|------|---------|
| `ViewModels/ViewModelBase.cs` | ReactiveUI 视图模型基类 | ViewModelActivator |
| `ViewModels/MainViewModel.cs:99-211` | 主窗口视图模型 | Status, Text, IsLocked, HistoryTexts, PlayCommand, PauseCommand, StopCommand, LockCommand, CaptionStyleViewModel |
| `ViewModels/ConfigViewModel.cs` | 配置窗口视图模型 | GeneralSectionConfig, AppearanceSectionConfig, AudioSectionConfig, RecognizeSectionConfig, NotificationConfig |
| `ViewModels/ResourceManagerViewModel.cs` | 资源管理器视图模型 | Items (ResourceItemViewModel 列表), Loading, LoadCommand |

**ConfigViewModel 包含的配置节视图模型**：
- `SectionConfigViewModelBase`: 配置节基类，实现序列化/反序列化
- `GeneralSectionConfigViewModel`: 通用设置（语言、启动选项、日志路径）
- `AppearanceSectionConfigViewModel`: 外观设置（字体、颜色、阴影、对齐）
- `AudioSectionConfigViewModel`: 音频源配置（动态加载插件）
- `RecognizeSectionConfigViewModel`: 识别器配置（动态加载插件）
- `NotificationConfigViewModel`: 通知设置（通知类型、敏感词）

#### 2.3 视图 (Views/)

| 文件路径 | 作用 | 关键成员 |
|---------|------|---------|
| `Views/MainWindow.axaml.cs:12-139` | 主窗口代码后台 | SetCaptionLock() Win32 API 实现窗口穿透，无边框窗口拖拽和调整大小（BeginResizeDrag） |
| `Views/MainWindow.axaml` | 主窗口布局 | 字幕显示、控制按钮、历史记录 |
| `Views/ConfigWindow.axaml.cs` | 配置窗口代码后台 | 显示版本信息（GitVersionInformation） |
| `Views/ConfigWindow.axaml` | 配置窗口布局 | 多标签页配置界面 |
| `Views/HistoryWindow.axaml.cs` | 历史记录窗口代码后台 | 构造函数接收 MainViewModel |
| `Views/HistoryWindow.axaml` | 历史记录窗口布局 | 列表显示历史识别文本 |
| `Views/ResourceManagerView.axaml.cs` | 资源管理视图代码后台 | 用户控件 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jxlpzqc/TMSpeech](https://github.com/jxlpzqc/TMSpeech) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
