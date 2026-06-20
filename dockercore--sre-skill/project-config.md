---
trigger: always_on
description: Server Operations Toolkit — Monitoring, Log Analysis, Service Management, Docker (服务器运维工具集 — 监控巡检、日志分析、服务管理、Docker容器管理)
---


# ops-toolkit — Server Operations Toolkit / 服务器运维工具集

A comprehensive Linux/macOS server operations skill covering 4 modules: Monitoring, Log Analysis, Service Management, and Docker Management.
全面的 Linux/macOS 服务器运维 skill，覆盖监控巡检、日志分析、服务管理、Docker 容器管理四大模块。

---

## Module 1: Server Monitoring & Health Check / 模块一：服务器监控与巡检

### Quick Health Check / 一键健康检查

Run the built-in health check script to automatically inspect CPU, memory, disk, network, processes, services, and Docker:
运行内置巡检脚本，自动检查 CPU、内存、磁盘、网络、进程、服务、Docker 状态：

```bash
# Quick mode (default) — for daily checks / 快速模式（默认）—— 适合日常巡检
bash ~/.hermes/skills/devops/ops-toolkit/scripts/health-check.sh --quick

# Full mode — includes network interfaces, container details / 完整模式 —— 包含网络接口详情、容器详情等
bash ~/.hermes/skills/devops/ops-toolkit/scripts/health-check.sh --full

# Show help / 显示帮助
bash ~/.hermes/skills/devops/ops-toolkit/scripts/health-check.sh --help
```

**Expected Output / 期望输出：**
```
=========================================
  服务器健康巡检  2026-04-14 14:36:20
  系统: Darwin
=========================================

[系统信息]
  主机名:   zailing
  内核:     25.4.0
  运行时间: up 7 days, 23:51
  当前用户: dockercore

[CPU]
  [OK]   CPU 使用率 51%
  CPU 核心数: 8
  负载均值:   10.75 8.35 7.25

[内存]
  [OK]   内存使用率 8% (1340M/16384M)
  可用内存:   694M
  [WARN] Swap 使用率 95% (6836M/7168M)

[磁盘]
  [OK]   挂载: /  大小: 460Gi  已用: 12Gi  可用: 46Gi  使用率: 21%
  [WARN] 挂载: /System/Volumes/Data  大小: 460Gi  已用: 375Gi  可用: 46Gi  使用率: 89%

[网络]
  [OK]   监听端口数: 20
  端口列表:   80 3306 5000 8080 ...

[进程]
  [OK]   进程总数: 565
  [OK]   无僵尸进程

[Docker]
  [OK]   Docker 守护进程运行中
  运行容器:   5
  镜像数:     12

=========================================
  巡检完成
=========================================
```

**Output Legend / 输出图例：**
- `[OK]` = Normal / 正常
- `[WARN]` = Warning, needs attention / 警告，需要关注
- `[FAIL]` = Critical, must fix immediately / 严重，必须立即处理

### CPU Monitoring / CPU 监控

```bash
# Real-time CPU usage (1-second sample) / 实时 CPU 使用率（采样1秒）
# Linux:
top -bn1 | grep "Cpu(s)" | awk '{print "User: "$2", System: "$4", Idle: "$8}'
# macOS:
top -l 1 -n 0 | grep "CPU usage"

# CPU core count / CPU 核心数
nproc                              # Linux
sysctl -n hw.ncpu                  # macOS

# Load average (1min/5min/15min) / 负载均值（1分钟/5分钟/15分钟）
# Load average = average number of processes waiting for CPU
# 负载均值 = 等待 CPU 的平均进程数
# Rule of thumb: load < core count = healthy
# 经验法则：负载 < 核心数 = 健康
cat /proc/loadavg                  # Linux
sysctl -n vm.loadavg               # macOS

# Per-core CPU usage / 按核心查看 CPU 使用率
mpstat -P ALL 1 1                  # Requires sysstat package / 需要 sysstat 包

# Top 10 CPU-consuming processes / 高 CPU 进程 Top 10
ps aux --sort=-%cpu | head -11     # Linux
ps aux -r | head -11               # macOS
```

**What is Load Average? / 什么是负载均值？**
- The three numbers represent 1-minute, 5-minute, and 15-minute averages
- 三个数字分别代表 1分钟、5分钟和 15分钟的平均值
- If you have 4 cores, load of 4.0 means 100% busy
- 如果你有 4 个核心，负载 4.0 表示 100% 繁忙
- Load > core count = processes are waiting / 负载 > 核心数 = 进程在排队

### Memory Monitoring / 内存监控

```bash
# Memory overview (MB) / 内存使用概览（MB）
free -m                            # Linux
vm_stat                            # macOS (pages, multiply by 4096 for bytes)

# Memory details / 内存使用详情
cat /proc/meminfo | head -20       # Linux
sysctl -n hw.memsize               # macOS: total physical memory

# Top 10 memory-consuming processes / 内存 Top 10 进程
ps aux --sort=-%mem | head -11     # Linux
ps aux -m | head -11               # macOS

# Continuous monitoring (refresh every 2s) / 持续监控内存（每2秒刷新）
watch -n2 free -m                  # Press Ctrl+C to stop / 按 Ctrl+C 停止
```

**Understanding Memory: / 理解内存：**
- `used` = memory currently in use / 当前使用的内存
- `free` = completely unused memory / 完全未使用的内存
- `available` = memory available for new programs (includes cache that can be freed)
- `available` = 可分配给新程序的内存（含可释放的缓存）
- `buffers/cache` = disk cache, automatically freed when needed
- `buffers/cache` = 磁盘缓存，需要时自动释放
- Don't panic if `free` is low — Linux uses free memory for cache
- `free` 很低不要慌 — Linux 会把空闲内存用作缓存

### Disk Monitoring / 磁盘监控

```bash
# Disk usage overview / 磁盘使用概览
df -hT -x tmpfs -x devtmpfs        # Linux
df -h                              # macOS

# Inode usage (filesystem can run out of inodes even with free space)
# Inode 使用率（即使空间有余，Inode 耗尽也会报错）
df -i -x tmpfs -x devtmpfs         # Linux only

# Top 10 largest directories / 大目录 Top 10
du -ah /path 2>/dev/null | sort -rh | head -10

# Mount details / 挂载详情
findmnt                            # Linux
mount                              # macOS

# Disk I/O statistics / 磁盘 I/O 统计
iostat -xz 1 3                     # Requires sysstat / 需要 sysstat 包
```

**What are Inodes? / 什么是 Inode？**
- Every file uses one inode. If you create millions of tiny files, inodes run out before disk space does.
- 每个文件占用一个 inode。如果创建大量小文件，inode 会比空间先耗尽。

### Network Monitoring / 网络监控

```bash
# Listening ports / 监听端口
ss -tlnp                           # Linux
lsof -iTCP -sTCP:LISTEN -P -n      # macOS

# Connection statistics / 网络连接统计
ss -s                              # Linux
netstat -s                         # macOS

# Connections by state / 各状态连接数

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dockercore/sre-skill](https://github.com/dockercore/sre-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
