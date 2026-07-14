---
trigger: always_on
description: 锤子时钟 7.1.1 的现代 Android 复刻项目。目标是在保留原版视觉、动画、手势和机械触感的前提下，使用当前稳定 Android 工具链实现可维护、可测试、可在现代设备运行的版本。
---

# Smartisan Clock Revived

锤子时钟 7.1.1 的现代 Android 复刻项目。目标是在保留原版视觉、动画、手势和机械触感的前提下，使用当前稳定 Android 工具链实现可维护、可测试、可在现代设备运行的版本。

## 开发约束

- 源码全部使用 Kotlin，主包为 `com.smartisan.clock`，目录固定为 `app/src/main/kotlin/com/smartisan/clock/`。
- UI 使用 XML Layout、Android View 和自定义 View；不要引入 Jetpack Compose 或 Material 组件重写页面。
- 视觉、尺寸、状态机、动画和触摸参数必须以坚果 R2 原厂时钟 APK、对应系统框架、资源表及反编译证据为准；现代化集中在生命周期、时间基准、Insets、性能与安全边界。
- 新项目 namespace 为 `com.smartisan.clock`，applicationId 为 `app.smartisanclock.revived`。原版包名 `com.smartisanos.clock` 仅用于逆向对照，不得作为新源码包。
- 这是无历史包袱的新项目，不需要兼容原版数据库、SharedPreferences、签名权限或 Smartisan OS 私有服务。
- 不复制原版 OEM 特权权限、导出组件、云同步和系统覆盖层。新增系统能力时使用现代公开 Android API，并坚持最小权限与默认不导出。
- 新增或升级依赖前先查阅 Android/Kotlin 官方文档，只使用最新稳定版；版本统一维护在 `gradle/libs.versions.toml`。
- 保持竖屏原版画布与现有多 Activity 结构，除非原版证据或明确需求要求调整。

## 项目结构

```text
app/src/main/kotlin/com/smartisan/clock/
├── ClockActivity.kt                  # 四页面宿主、生命周期与页面切换
├── ClockApplication.kt               # 闹钟/计时提醒依赖图与应用级协程作用域
├── AlarmEditorActivity.kt            # 新建/编辑闹钟页面
├── AlarmRingingActivity.kt           # 独立锁屏响铃任务与关闭/贪睡操作
├── TimerRingingActivity.kt           # 计时结束时承载原版居中提醒框的独立任务
├── CityPickerActivity.kt             # 世界时钟城市搜索、索引与选择
├── RingtonePickerActivity.kt         # 使用系统公开铃声目录的独立选择页
├── ClockTab.kt                       # 底部标签及资源映射
├── SystemBars.kt                     # 统一 edge-to-edge 与系统栏 Insets
├── custom/
│   ├── AnalogClockHandsView.kt       # 原版表盘、指针及分层进出场动画
│   ├── AdaptiveFrameSequenceView.kt  # 经典拉环回弹序列的 VSYNC 自适应播放
│   ├── AlarmEarAnimationView.kt      # 响铃卡片铃耳序列的高刷自适应播放
│   ├── AlarmRingingPanelView.kt      # 原版响铃卡片进出场、阻尼与上滑关闭
│   ├── AlarmRepeatDaysView.kt        # 星期多选、快速拖选与节假日设置
│   ├── Classic680RulerView.kt        # 经典计时器竖向拉环、回弹与手势
│   ├── CompactAlarmClockView.kt      # 原版响铃浮层的紧凑机械表盘
│   ├── QuickBarEx.kt                 # 城市列表字母索引及展开网格
│   ├── SmallWorldClockView.kt        # 世界时钟列表的小表盘
│   ├── SmartisanSwitchExView.kt      # 原版位图滑块、拖动动画与触觉反馈
│   ├── SmartisanSwitchView.kt        # 锤子风格闹钟开关
│   ├── SmartisanTimePickerView.kt    # 闹钟滚轮选择器
│   ├── TimerRulerView.kt             # 0–180 分钟刻度、阻尼、惯性与吸附
│   └── WorldClockListView.kt         # 城市时钟列表的拖动与选择承载
├── data/
│   ├── AlarmRepository.kt            # 多闹钟 DataStore 与旧数据迁移
│   ├── DirectBootAlarmStore.kt       # 解锁前可读的最小调度快照
│   ├── ActiveAlarmStore.kt           # 当前响铃 occurrence 与短时 lease
│   ├── TimerAlertStore.kt             # 计时提醒 session 与 Direct Boot 状态
│   ├── TimerPreferencesRepository.kt # 计时器样式的 DataStore 持久化
│   ├── WorldCityRepository.kt        # 城市 TSV 与公开 ZoneId 数据读取
│   └── WorldClockStore.kt            # 已添加城市及顺序持久化
├── model/
│   ├── Alarm.kt                      # 闹钟领域模型及系统调度 identity
│   ├── AlarmOccurrenceCalculator.kt  # 星期、时区与 DST 下次触发计算
│   ├── AlarmRepeat.kt                # 周一至周日 bitmask 与本地化展示
│   ├── ClockStates.kt                # 秒表与倒计时单调时间状态机
│   ├── TimerModels.kt                # 计时器样式、运行态与界面状态
│   └── WorldCity.kt                  # 城市搜索、排序与选择规则
├── ui/
│   ├── AlarmPage.kt                  # 多闹钟页面、编辑态与能力提示
│   ├── AlarmViewModel.kt             # 列表状态、最近闹钟与用户意图
│   ├── AlarmListAdapter.kt           # 稳定 ID 的多闹钟列表绑定
│   ├── ClockPage.kt                  # 页面生命周期接口
│   ├── ClockPages.kt                 # 世界时钟与秒表控制逻辑
│   ├── TimerPage.kt                  # 共享表盘、双样式切换与生命周期
│   ├── TimerSurface.kt               # 计时器样式界面契约
│   ├── TimerViewModel.kt             # 单调倒计时状态机与进程恢复
│   ├── ModernTimerSurface.kt         # 横向刻度计时器界面
│   ├── Classic680TimerSurface.kt     # 经典拉环计时器界面
│   ├── Classic680TimerSoundPlayer.kt # 经典拉环机械音效
│   └── WorldClockListAdapter.kt      # 世界时钟列表绑定与原版行布局
├── alarm/                            # 闹钟调度、恢复、通知、Receiver 与响铃服务
├── timer/                            # 计时结束调度、恢复、提醒通知与循环音服务
└── widget/
    ├── ClockBottomBar.kt             # 原版四标签底栏
    ├── ClockContentFrame.kt          # 480dp 手机内容画布与居中约束
    ├── SmartisanMenuDialog.kt        # R2 MenuDialog 风格底部操作面板
    └── SmartisanModalDialog.kt       # Revone 风格居中弹窗公共外壳

app/src/main/res/
├── layout/                           # Activity 与四个主页面 XML
├── drawable*/                        # 原版 PNG、selector 与本地矢量/shape
├── anim/                             # 闹钟编辑页转场
└── font/                             # 恢复的时钟字形

app/src/main/assets/world_cities.tsv  # 城市、国家、别名与 IANA 时区数据

docs/clock-7.1.1-baseline.md           # 原版证据、参数和首批验收基线
docs/alarm-architecture.md             # 多闹钟、Direct Boot 与系统调度不变量
reverse/r2-stock/
├── app/Clock_7.1.1.apk                # 坚果 R2 原厂时钟，核心/最高优先级基准
├── framework/                         # 同机 Android 11 / Smartisan OS 框架证据
    ├── framework-res.apk
    ├── framework.jar
    ├── smartisanos.jar
    └── smartisanos_11.apk
├── decompiled/                        # R2 原厂 JADX/Vineflower/apktool 输出
├── tooling/apktool-frameworks/        # 项目私有的 framework 解码缓存
└── analysis/README.md                 # 逆向目录、证据顺序与关键入口索引

reverse/apk/Clock_7.1.1.apk            # 第三方移植版，仅保留为历史对照
```

## 当前架构与工具链

- **UI**：纯 Kotlin + XML/View，`ClockActivity` 承载四个主页面；城市、闹钟编辑和铃声选择保持独立 Activity。
- **世界时钟**：城市目录来自本地 TSV，时区计算使用 `java.time.ZoneId`；已选城市和排序由 `WorldClockStore` 保存。
- **弹窗**：重复和标签使用 `SmartisanModalDialog` 的 Revone 外壳，内容结构、尺寸和提交语义以原版 Clock 为准；不要退回平台默认 `AlertDialog`。
- **铃声**：使用公开 `RingtoneManager(TYPE_ALARM)` 和 Activity Result API，不恢复移植包中被硬编码的假铃声列表。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mangi-11/SmartisanClock-Revived](https://github.com/Mangi-11/SmartisanClock-Revived) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
