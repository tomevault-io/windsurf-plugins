---
trigger: always_on
description: ArkUI iOS 平台适配层是 ArkUI-X 跨平台框架的 iOS 平台实现，通过 Objective-C/Objective-C++ 桥接机制实现 ArkTS 应用在 iOS 设备上的原生渲染和交互。
---

# ArkUI iOS 平台适配层

## 项目概述

ArkUI iOS 平台适配层是 ArkUI-X 跨平台框架的 iOS 平台实现，通过 Objective-C/Objective-C++ 桥接机制实现 ArkTS 应用在 iOS 设备上的原生渲染和交互。

**代码位置**: `foundation/arkui/ace_engine/adapter/ios/`

## 目录结构

```
adapter/ios/
├── entrance/                           # 入口层 (ObjC/C++)
│   ├── WindowView.h/mm                 # 窗口视图
│   ├── ace_bridge.h/mm                 # ObjC/C++ 桥接
│   ├── virtual_rs_window.h/mm          # 虚拟渲染窗口 (核心)
│   ├── mmi_event_convertor.h/mm        # 事件转换
│   ├── display_info.h/mm               # 显示信息
│   ├── AcePlatformPlugin.h/mm          # 平台插件
│   ├── AceSurfaceHolder.h/mm           # Surface 持有者
│   ├── AceTextureHolder.h/mm           # Texture 持有者
│   ├── WantParams.h/mm                 # 参数封装
│   ├── DownloadManager.h/mm            # 下载管理
│   ├── accessibility/                  # 无障碍
│   │   ├── AccessibilityWindowView.h/mm
│   │   ├── AccessibilityElement.h/mm
│   │   ├── AccessibilityNodeInfo.h/mm
│   │   └── AceAccessibilityBridge.h/mm
│   ├── resource/                       # 资源管理
│   │   ├── AceResourcePlugin.h/m
│   │   ├── AceResourceRegisterDelegate.h
│   │   ├── AceResourceRegisterOC.h/mm
│   │   ├── IAceOnCallResourceMethod.h
│   │   └── IAceOnResourceEvent.h
│   ├── plugin_lifecycle/               # 插件生命周期
│   │   ├── ArkUIXPluginRegistry.h/mm
│   │   ├── IArkUIXPlugin.h
│   │   ├── IPluginRegistry.h
│   │   └── PluginContext.h/mm
│   ├── logIntercept/                   # 日志拦截
│   │   ├── ILogger.h
│   │   ├── Logger.h/mm
│   │   └── LogInterfaceBridge.h/mm
│   ├── interaction/                    # 交互能力
│   │   └── interaction_impl.h/cpp
│   ├── html/                           # HTML 转换
│   │   └── html_to_span.cpp
│   ├── picker/                         # 选择器
│   ├── report/                         # 上报能力
│   ├── udmf/                           # 统一数据管理框架
│   │   └── udmf_impl.h/cpp
│   ├── ui_session/                     # UI 会话管理
│   │   └── ui_session_manager_ios.h
│   └── xcollie/                        # 看门狗
│       └── xcollieInterface_impl.h
├── stage/                              # Stage 模型适配
│   ├── ability/                        # 能力层 (ObjC)
│   │   ├── StageViewController.h/mm    # 主 ViewController
│   │   ├── StageApplication.h/mm       # 应用入口
│   │   ├── StageContainerView.h/mm     # 容器视图
│   │   ├── StageSecureContainerView.h/mm
│   │   ├── StageAssetManager.h/mm      # 资源管理
│   │   ├── StageConfigurationManager.h/mm
│   │   ├── InstanceIdGenerator.h/mm    # 实例 ID 生成
│   │   ├── Stage.h                     # Stage 定义
│   │   ├── AbilityLoader.h/mm          # 能力加载器
│   │   ├── stage_asset_provider.h/mm
│   │   ├── ability_context_adapter.h/mm
│   │   ├── application_context_adapter.h/mm
│   │   ├── window_view_adapter.h/mm
│   │   └── version_printer.h/cpp
│   └── uicontent/                      # UI 内容 (C++)
│       ├── ace_container_sg.h/cpp      # ACE 容器
│       ├── ui_content_impl.h/cpp       # UI 内容实现
│       ├── ace_view_sg.h/cpp           # ACE 视图
│       └── platform_event_callback.h
├── osal/                               # 平台抽象层 (C++)
│   ├── accessibility_manager_impl.h/cpp # 无障碍服务 (核心)
│   ├── subwindow_ios.h/cpp             # 子窗口管理
│   ├── resource_adapter_impl.h/cpp     # 资源适配
│   ├── resource_adapter_impl_v2.h/cpp  # 资源适配 V2
│   ├── system_properties.cpp           # 系统属性
│   ├── display_manager_ios.h/cpp       # 显示器管理
│   ├── image_source_ios.h/cpp          # 图片解码
│   ├── pixel_map_ios.h/cpp             # 位图操作
│   ├── input_method_manager_ios.cpp    # 输入法管理
│   ├── mouse_style_ios.h/cpp           # 鼠标样式
│   ├── navigation_route_ios.h/cpp      # 导航路由
│   ├── image_packer_ios.h/cpp          # 图片打包
│   ├── drawable_descriptor_ios.h/cpp   # drawable 描述
│   ├── resource_theme_style.h/cpp      # 主题样式
│   ├── resource_convertor.h/cpp        # 资源转换
│   ├── resource_path_util.h/mm         # 资源路径 (ObjC++)
│   ├── file_asset_provider.h/cpp       # 文件资源提供
│   ├── file_uri_helper_ios.cpp         # URI 助手
│   ├── frame_trace_adapter_impl.h/cpp  # 帧追踪
│   ├── layout_inspector.cpp            # 布局检查
│   ├── modal_ui_extension_impl.cpp     # 模态扩展
│   ├── drag_window.cpp                 # 拖拽窗口
│   ├── view_data_wrap_impl.cpp         # 视图数据
│   ├── websocket_manager.cpp           # WebSocket
│   ├── system_bar_style_ohos.h/cpp     # 系统栏样式
│   ├── advance/                        # 高级功能适配
│   │   ├── ai_write_adapter.cpp        # AI 写作
│   │   ├── data_detector_adapter.cpp   # 数据检测
│   │   ├── image_analyzer_adapter_impl.cpp # 图片分析
│   │   ├── text_share_adapter.cpp      # 文本分享
│   │   └── text_translation_adapter.cpp # 文本翻译
│   └── mock/                           # 无障碍模拟实现
│       ├── accessibility_element_info.h/cpp
│       ├── accessibility_constants.h/cpp
│       └── accessibility_def.h
├── capability/                         # 平台能力扩展
│   ├── web/                            # WebView 组件
│   │   ├── AceWeb.h/mm                 # 主类 (核心)
│   │   ├── AceWebControllerBridge.h/mm # 控制器桥接
│   │   ├── AceWebResourcePlugin.h/mm   # 资源插件
│   │   ├── AceWebCallbackObjectWrapper.h/cpp
│   │   ├── AceWebPatternBridge.h/cpp   # Pattern 桥接
│   │   ├── AceWebObject.h/cpp          # Web 对象
│   │   ├── AceWebDownloadImpl.h/cpp    # 下载实现
│   │   ├── WebMessageChannel.h/mm      # 消息通道
│   │   └── AceWebMessageExtImpl.h/cpp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daren866/HOi](https://github.com/daren866/HOi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
