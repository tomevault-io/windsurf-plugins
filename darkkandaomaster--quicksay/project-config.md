---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目

QuickSay：Windows 上的快捷短语工具。Qt 6.5.3 + MinGW，qmake 构建。只支持 Windows 10/11，大量直接调用 Win32 API（`SendInput`、`WH_KEYBOARD_LL` 钩子、剪贴板、注册表 Run 项），不考虑跨平台。

## 构建与运行

工具链固定在 `D:/Programs/DevEnvironments/Qt/6.5.3/mingw_64` + `D:/Programs/DevEnvironments/Qt/Tools/mingw1120_64`。

```powershell
cd QuickSay
$env:PATH="D:\Programs\DevEnvironments\Qt\Tools\mingw1120_64\bin;D:\Programs\DevEnvironments\Qt\6.5.3\mingw_64\bin;$env:PATH"
D:/Programs/DevEnvironments/Qt/6.5.3/mingw_64/bin/qmake.exe QuickSay.pro -spec win32-g++
D:/Programs/DevEnvironments/Qt/Tools/mingw1120_64/bin/mingw32-make.exe -j8
./QuickSay.exe
```

- **PATH 里必须把 `mingw1120_64/bin` 排在最前面**。Makefile 调的是裸 `g++`，而这台机器上还装着另一套 MinGW（`D:/Program Files/Programming/C,C++/mingw64`，gcc 8.1.0）。撞上它的表现是在 `bits/fs_path.h` 里报一大堆 `no match for 'operator!='`、`'value_type' was not declared` 之类看不懂的模板错误——那不是代码问题，是编译器拿错了。
- **链接前先关掉正在运行的 QuickSay.exe**，否则 `ld.exe: cannot open output file QuickSay.exe: Permission denied`。
- **`.pro` 必须保持 CRLF 行尾**。存成 LF 之后 qmake 会把 `include(.\QHotkey-1.5.0\qhotkey.pri)` 解析坏，`INCPATH` 里变成 `-I/include`，编译报 `QHotkey: No such file or directory`。用脚本批量改 `.pro` / `main.cpp` 时，读写都要显式保住 CRLF（`main.cpp` 还带 UTF-8 BOM，也要一起保住）。
- **必须关闭 Shadow build**（`.pro` 里 `CONFIG -= debug_and_release`，Qt Creator 项目页也要取消影子构建）。exe 必须和 `icons/` 同级，否则图标全丢；`config.json` / `data.json` / `tab.json` 也写在 exe 同目录。
- 生成物（`Makefile`、`*.o`、`moc_*`、`ui_*.h`、`QuickSay.exe`、`build/`）全部被 `QuickSay/.gitignore` 忽略，不要提交。
- 没有测试，没有 lint。编译由 Claude 跑，功能验证靠作者手测，见下面「验证与测试」。

### 验证与测试

**编译由 Claude 来跑**：改完代码就照上面那段命令编一遍，别把语法错误留给作者。链接前直接 `taskkill /IM QuickSay.exe /F` 把正在跑的那个关掉——作者认这个代价，重开一下就是了。作者自己也会在 Qt Creator 里编，两边共用同一份 `Makefile` 和 `.o`（Shadow build 是关的）。

**功能验证只能作者手测。** UI 全是手写代码构造的，再加上低级键盘钩子、`SendInput`、剪贴板、UAC 弹窗这些东西，没法脚本化点击，也没有任何自动化测试可写可跑。

Claude 唯一能自己核对的是「进程级事实」——不点界面也能查到的那些。改自启、单实例、提权、持久化这类东西时值得核一遍：

| 想确认什么 | 怎么看 |
|---|---|
| 起没起来、有没有开出第二个实例 | `tasklist` 里 `QuickSay.exe` 有几个、PID 变没变 |
| 有没有弹 UAC | 弹窗那几秒里 `consent.exe` 在不在进程列表里 |
| 设置有没有落盘、老字段有没有迁移 | 直接读 exe 同目录的 `config.json` / `data.json` / `tab.json` |
| 开机自启的真实状态 | `reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v QuickSay`、`Get-ScheduledTask -TaskName 'QuickSay 管理员自启'` |

界面长什么样、快捷键灵不灵、短语输出对不对，一律交给作者手测，别自己下「应该没问题」的结论。

### 开发目录里跑不通的东西

开发目录里 exe 旁边**没有 Qt DLL**，Qt 的 bin 也**不在持久 PATH 里**（只有 Qt Creator、或上面那句 `$env:PATH` 临时注入）。所以凡是「绕过当前 shell、由 Windows 自己去拉起 QuickSay.exe」的功能，在开发目录里一定失败，退出码 `-1073741515`（`0xC0000135` STATUS_DLL_NOT_FOUND）：

- 开机自启（注册表 Run 项、计划任务）
- `ShellExecuteEx` + `runas` 提权重新拉起自己

**这不是代码 bug，别去改代码**。注意 `$env:PATH` 只在当前进程有效，提权（经 AppInfo 服务）和计划任务拉起的子进程都拿不到它。要手测这类功能，先 windeployqt 出一份完整副本，在副本上测：

```powershell
$d="$env:TEMP\qs-test"; mkdir $d -Force
copy QuickSay.exe $d; copy icons $d -Recurse
D:\Programs\DevEnvironments\Qt\6.5.3\mingw_64\bin\windeployqt.exe --no-translations --no-opengl-sw "$d\QuickSay.exe"
```

测完记得清干净：注册表 Run 项要还原成开发目录的路径（别留着临时目录的）、计划任务要删掉、临时目录要删掉——这些是真写进系统的，不是沙盒。

## 代码结构

**整个程序就是 [QuickSay/main.cpp](QuickSay/main.cpp) 一个文件（约 3500 行）**，UI 全部手写代码构造，没有 .ui 布局。

`.pro` 里列的 `mainwindow.cpp/h/ui` 是 Qt Creator 新建项目时的模板残留（被 gitignore，不在仓库里，克隆后第一次 qmake 前要自己建空壳，见 README 的复现步骤）。`MainWindow` 类没有任何人用，改代码不要碰它。

`QHotkey-1.5.0/` 是 vendored 的第三方库，通过 `.pri` include 进来，不要改。

### main.cpp 的分层（按行号顺序）

1. **全局状态**：`config`（QJsonObject，全部设置）+ 一堆 `g_` / `p` 裸指针（`pchuangkou` 主窗口、`g_liebiao`、`g_tabBar`、`g_search`、`g_tuding`、`g_keyboardHook`…）。所有窗口和控件都是 `main()` 里的栈对象，靠这些全局指针给自由函数和事件过滤器用。
2. **持久化**：`saveConfig`/`loadConfig`、`saveListToJson`/`loadListFromJson`、`saveTabToJson`/`loadTabFromJson`。
3. **开机自启**：`applyZiqidong` + 计划任务那一套 COM 调用，见下面「开机自启」一节。
4. **输出流水线**：`parseQuickSayOutputActions` → `QVector<QuickSayOutputAction>` → `QuickSayOutputRunner`。
5. **键盘钩子 + 免激活窗口**：`quickSayKeyboardProc` / `handleQuickSayBrowseKey` / `showMainWindowNoActivate` / `enterSearchMode`。
6. **事件过滤器类**：`WindowMoveFilter`（记录窗口位置）、`MyEventFilter`（Esc/回车/左右键）、`PhraseItemToolTipFilter`、`HotkeyEditFilter` / `KjjHotkeyEditFilter`（编辑快捷键时临时注销全局快捷键）、`BadgeDelegate`（画角标）、`MyTabBar`（滚轮切分组）。
7. **`main()`**（2297 行起，行号会漂，用 `grep -n "^int main"` 找）：全局 QSS 样式表 → 主窗口 → 设置窗口 → 添加窗口 → 修改窗口 → 托盘 → 装过滤器 → `adjustAllWindows`。

### 几个必须知道的约定

**短语项数据存在 `Qt::UserRole` 偏移里**，改动任何一处都要同步 `saveListToJson`/`loadListFromJson`：

| 角色 | 内容 | data.json 字段 |
|---|---|---|
| `UserRole` | 短语正文 | `text` |
| `UserRole+1` | 备注（非空则代替短语显示，绿色） | `remark` |
| `UserRole+2` | 所属分组名 | `tab` |
| `UserRole+3` | 短语快捷键字符串 | `hotkey` |
| `UserRole+4` | 角标字符（运行期算出，不落盘） | — |

分组的短语项高度塞在 `QTabBar::tabData()` 里（`tab.json` 的 `item_height`）。

**分组是用名字字符串关联的**，不是 id。改分组名要遍历所有短语项把 `UserRole+2` 一起改（`main.cpp` 2732 行附近，行号会漂，用 `grep -n "UserRole+2,newName"` 找），删分组要连带删该分组下的短语项。

**过滤/角标只有一个入口**：`filterListByTab(liebiao, 当前分组名, 搜索框文字)`。任何会改变可见项集合的操作（切分组、搜索、拖动排序、增删改）之后都必须调它一次，否则角标错位。

**布局是绝对坐标**，全部集中在 `adjustAllWindows()`。加控件就在那个函数里加一行 `move` + `setFixedSize`，不要引入 layout（除了设置窗口的 `QFormLayout`）。

**兼容旧版本的方式**：`loadConfig` 里逐个 `if(!config.contains("xxx")) config["xxx"]=默认值`；启动时 load 完立刻 save 一次把新字段落盘。加新设置项照抄这个模式，同时在 `loadConfig` 的 else 分支（config.json 不存在）里补上默认值——两处都要写。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DarkKandaoMaster/QuickSay](https://github.com/DarkKandaoMaster/QuickSay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
