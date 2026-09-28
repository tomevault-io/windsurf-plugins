---
trigger: always_on
description: 这是一个基于 MSPM0G3507 的嵌入式 C99 智能车工程，使用 CMake 与 ARM GNU 工具链构建。工程包含菜单、显示、灰度、编码器、电机、舵机、PID、循迹、Flash 参数保存等应用模块。
---

# Repository Guidelines

## 0. 项目速览

这是一个基于 MSPM0G3507 的嵌入式 C99 智能车工程，使用 CMake 与 ARM GNU 工具链构建。工程包含菜单、显示、灰度、编码器、电机、舵机、PID、循迹、Flash 参数保存等应用模块。

> **路径约定：** 除特别说明外，本文件中的 `project/code/...`、`build/...`、
> `cmake --build build/debug`、`flash.sh`/`flash.ps1` 等路径与命令均相对
> **`firmware/`** 目录（仓库根目录下主控固件工程所在位置）执行。
> `vision/`、`report/`、`tools/`、`docs/` 为仓库根目录下的平级子目录。

入口与关键调度：

```text
firmware/project/user/src/main.c        启动入口，只保留初始化和进入运行框架
firmware/project/user/src/isr.c         底层中断入口，调用逐飞 callback 表
firmware/project/code/core/app_runtime.c 主循环调度
firmware/project/code/core/app_interrupts.c 应用层 PIT/EXTI 绑定与中断任务调度中枢
```

`project/code` 下源码不再平铺，必须按职责放入三个子目录：

```text
project/code/
  core/      系统运行、调度、参数、Flash、PID、循迹、黑板等核心应用层
  device/    传感器、执行器、显示、按键、编码器等硬件封装层
  menu/      Easy Menu 框架、用户菜单页和菜单门面
```

典型文件职责：

```text
core/
  app_runtime.c       主循环调度
  app_interrupts.c    应用层 PIT/EXTI 中断绑定中枢
  app_blackboard.c    全局黑板数据
  app_tracking.c      循迹与速度 PID 运行逻辑
  app_balance.c       摄像头小球/步进滑轨位置维持外环
  app_ladrc.c         二阶对象LADRC、TD、LESO和带宽参数化反馈
  app_pid.c           PID算法；参数接口保持float，控制周期使用Q16.16定点内核
  app_param_store.c   可保存参数缓存与 Flash 读写统一入口
  app_flash_store.c   Flash 通用记录读写工具
  app_config.c        系统配置
  app_boot.c          启动日志
  app_system_load.c   系统负载统计
  app_control_uart.c  UART1 控制命令、参数配置、状态查询与遥测
  app_stepper_control.c 步进电机统一控制锁、位置环与速度/位置命令区

device/
  app_display.c       IPS 显示缓存与统一 flush
  app_keys.c          按键扫描与短按事件缓存
  app_beep.c          蜂鸣器非阻塞状态机
  app_motor.c         电机控制
  app_electromagnet.c 电磁铁 PWM API（当前不初始化、不注册菜单，B4 已释放）
  app_servo.c         舵机控制与 steer 查表插值
  app_encoder.c       编码器采样
  app_camera_uart.c   UART2 B15/B16钢球v5二进制协议解析与帧率统计
  app_zdt_stepper.c   UART1 B6/B5 Emm V5.0闭环步进驱动命令与响应
  app_gs08ra.c        八路灰度
  app_attitude.c      姿态调度与 IMU 数据处理
  app_battery.c       电池 ADC
  imu963ra_attitude.c IMU963RA 姿态解算封装

menu/
  app_menu.c          菜单门面，负责物理按键到 Easy Menu 的适配
  Easy_Menu*.c/.h     Easy Menu 框架源码
  Easy_Menu_User.c    所有菜单页面、菜单项、页面回调集中定义
```

当前主菜单注册 `ZDT Stepper` 子菜单，通过UART1 B6/B5控制Emm V5.0闭环步进驱动：

```text
Jog RPM：零点校准点动速度，1..5000 RPM可调，步长100 RPM，默认300 RPM
Jog step deg：零点校准页面的单次相对点动角度，默认100°
Center Cal：唯一校准入口；KEY2/KEY3短距离点动，KEY1把小球恰好静止的位置标定为0°
Sine Speed：KEY1启停±5000 RPM正弦测试，完整周期固定1 s；KEY4强制零速并返回
```

当前已知未解决问题：进入`Sine Speed`后仍可能出现约-2900°的固定中心迁移。
此前删除页面层`-2392°`补偿状态机后现象仍存在，说明根因不只在菜单进入流程，
还可能与Emm驱动器内部残留位置目标、速度模式切换或驱动坐标反馈有关。后续排查前
禁止再次添加经验偏移补偿，也不得把该偏移当成已经校准的机械零点。
零点校准收到驱动器0x0A“当前位置清零”成功应答后必须立即置`calibrated=1`并结束；
禁止再发送绝对位置0°进行二次验证，该多余位置命令会触发约-2.9k°残留目标，
导致校准永不完成，并使统一速度安全层把Sine、Rail/Ball Hold全部输出压成0。

`ZDT Stepper -> Position Hold`包含三个自动运行Show Page：

```text
Rail Zero Hold：不读取小球位置，只以驱动坐标0°为目标产生步进速度，用于排查-2.9k偏移
Ball Zero Hold：小球位置PD产生目标杆角，滑轨位置P再产生步进速度
Deadband 0.1cm：小球误差死区半宽，步长0.1 cm，默认4即±0.4 cm
Max rail deg：静止起滚Boost最大电机轴角，默认2100°，范围100..3000°
Run rail deg：小球已经滚动后的常规控制最大杆角，默认1200°
Max RPM：滑轨位置内环速度上限，默认900 RPM
Rail Kp：滑轨位置内环P增益，默认0.2
+5 / -5 Traverse：从中心±1 cm内自动执行+5 cm折返至-5 cm；正端和负端容差均为±0.8 cm，
                 负端速度≤1 cm/s连续200 ms判定稳定，要求5 s内完成。页面进入即启动，
                 KEY1可在启动条件不足时重试，KEY4立即停止并上锁。
Rail Self Test：进入页面后再次KEY1确认，依次执行0→正峰值→负峰值→0；
                 每次到位鸣笛并停留2 s，最终回零后自动返回
```

位置维持控制位于`app_balance.c`，只复用通用`app_pid_t`算法，不复用面向双轮编码器的
`app_position`业务模块。摄像头控制只在有效帧计数`total_frames`变化时更新，禁止对同一
45 Hz数据帧在100 Hz节拍中重复计算D项；外部`sequence`可能复用，不得作为唯一的新帧依据。
超过100 ms无有效摄像头数据时请求速度必须归零。往返页使用独立整数轨迹PD：起步阶段
使用-2600°非阻塞起滚脉冲，检测到小球运动后切换到P=30的轨迹控制；正程D=10，回程D=30，
滑轨内环Kp=0.3、限幅1500 RPM，滑轨位置内环死区为±5°。端点微调只在误差超过±0.8 cm
且球速不超过0.5 cm/s时使用短脉冲，不允许在容差内持续动作。
由于当前存在约-2.9k°驱动坐标异常，Center Cal和Position Hold页面退出只允许立即
零速、停止并上锁，禁止调用`app_zdt_stepper_return_zero_on_exit()`，否则回零状态长期
无法满足0°到位条件并通过`app_zdt_stepper_input_busy()`锁死包括Sine Speed在内的菜单按键。
Rail Self Test正常完成时仍按其内部状态机回到0°后再退出。
小球控制采用串级结构：外环位置PD输出目标滑轨角，内环以驱动器真实位置反馈输出RPM。
小球零点保持的2026-07-30联调基线为：外环P=90、D=60、死区±0.4 cm，
Boost最大角2600°、常规角度限幅1800°；滑轨内环P=0.25，速度上限1200 RPM。
实测从约+1.99 cm回到+0.20 cm，约4.6 s进入死区，仍有数次衰减振荡，
该组参数作为继续增加D阻尼前的可回退版本。
实测机械中位约±500°不足以起滚，约1500°接近静摩擦阈值，约2100°会快速滚动，
因此控制器采用“静止起滚Boost + 常规低倾角PD”：小球位于死区外至少0.1 cm、速度
不超过0.2 cm/s并持续300 ms时进入Boost。Boost角按位置误差从近端1600°平滑增加，
误差达到3 cm时使用配置的2100°最大角；速度达到0.4 cm/s后立即退出Boost，常规目标角
限制为±1200°。Boost持续2.5 s仍未起滚时不得退出零点维持：先降回常规倾角冷却
0.8 s，再重新执行300 ms静止判定并尝试Boost；每次连续失败把近端Boost角提高200°，
直到配置的最大角，检测到起滚或进入“死区+0.1 cm”迟滞区后清零重试次数。迟滞区
用于避免摄像头噪声在死区边缘反复触发大倾角。只有用户退出、
反馈失效、校准失效或安全边界阻止运动时才允许停止输出；禁止长期保持大倾角。
禁止恢复为“小球PD直接输出RPM”的持续积分结构。
D项使用0.05 s一阶低通滤波。
当摄像头协议`MOTION_VALID`有效时，D项直接使用其`velocity_centi_cm_s`
（0.01 cm/s）作为测量变化率；该位无效时才退回相邻位置帧差分。P项始终使用
`position_centi_cm`位置误差。摄像头只在新帧时更新目标角，滑轨内环每10 ms运行。
UART0支持`@BALL_P:<value>`、`@BALL_D:<value>`、`@BALL_PD:<P>,<D>`立即修改RAM参数，
`@BALL_PD?`查询当前参数。设置成功依次返回原指令、`@OK:BALL_PD`和实际生效的
`@BALL_PD:<P>,<D>`；`@BALL_DB`调整死区，`@BALL_ANGLE`调整Boost角，
`@BALL_RUN_ANGLE`调整常规控制角，`@BALL_MAX`调整内环速度上限，
`@BALL_RAIL_KP`调整滑轨内环P。`@BALL?`末尾包含常规角、内环P、Boost状态和重试等待状态。
当前机构实测负滑轨角使负位置小球向视觉零点运动，因此外环目标角必须反相。
这些参数当前不写Flash，上电恢复代码默认值。

常用串口调试统一使用`tools/ball_tune.py`，依赖见`tools/requirements.txt`：

```sh
python3 tools/ball_tune.py status
python3 tools/ball_tune.py set --kp 30 --kd 30 --boost-angle 2100 --run-angle 1200 --max-rpm 900
python3 tools/ball_tune.py run --seconds 8 --csv temp/ball.csv
python3 tools/ball_tune.py zero
python3 tools/ball_tune.py stop
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [colorcard/26NUEDC-H](https://github.com/colorcard/26NUEDC-H) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
