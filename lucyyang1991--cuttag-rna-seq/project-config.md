---
trigger: always_on
description: Cut&Tag项目规范
---

每次修改脚本后，同步更新readme文件，包括增加和减少。

中文回答问题。

# 项目规范（Project Rules）

## 1. 项目概述

本项目是一个Cut&Tag测序数据分析流程，主要用于分析转录因子结合位点。项目包含从原始数据质控、比对到峰值检测和功能注释的完整流程。

## 2. 文件命名规则

- 所有Shell脚本以数字前缀命名（如`00_`，`01_`），表示执行顺序
- 分析步骤脚本命名应当简明扼要地反映其功能（如`01_fastqc.sh`）
- R脚本同样遵循数字前缀命名规则
- 配置文件统一放在`config`目录下
- 结果文件命名应包含样本名、分析步骤和日期信息

## 3. 目录结构规范

```
project_root/
├── CutTag_Linux/             # 原始数据及处理脚本
│   ├── config/               # 配置文件
│   ├── *_*.sh                # 各步骤处理脚本
│   └── merge.file/           # 整理好的原始文件（重要）
├── CutTag_local_analysis/           # 本地数据分析
│   ├── peak_calling/         # 峰值检测结果
│   ├── peak_annotation/      # 峰值注释结果
│   ├── visualization/        # 可视化结果
│   └── *_*.R                 # R分析脚本
└── reference_repos/          # 参考基因组和注释文件
```

### 特殊目录说明
- **CutTag_Linux/merge.file/**: 此目录包含整理好的原始文件，是数据分析的主要输入来源

## 4. 代码风格指南

### 通用规范
- **最小改动原则**：每次修改代码时应尽可能做最少量的改动，减少引入新错误的风险
- **单一职责**：每次改动应专注于解决单一问题或实现单一功能
- **改动前测试**：在修改代码前，确保了解现有代码的行为和依赖关系
- **改动后验证**：每次改动后，必须进行适当的测试验证，确保功能正常
- **作者署名**：
  - 在每个脚本文件开头注释中明确标注所有作者信息
  - 如果是合作编写（如与AI助手合作），应同时标注人类作者和AI助手
  - 署名格式示例：
    ```bash
    # 作者：张三
    # 合作者：Claude AI Assistant
    # 创建日期：2024-01-20
    # 最后修改：2024-01-21
    ```
  - 对脚本有实质性贡献的所有参与者都应被列入作者名单
  - 建议注明每位作者的具体贡献内容

### Shell脚本规范
- 每个脚本开头应包含注释，说明脚本功能、输入、输出和依赖
- 每个脚本开头必须包含使用方式说明，包括命令行参数解释和使用示例
  ```bash
  # 使用方式：
  # sh 脚本名.sh [参数1] [参数2] ...
  #   参数1    参数1的功能说明
  #   参数2    参数2的功能说明
  # 示例：
  #   sh 脚本名.sh 参数值1             # 示例1说明
  #   sh 脚本名.sh 参数值1 参数值2     # 示例2说明
  ```
- 使用`00_config.sh`集中管理路径和参数
- 使用有意义的变量名称
- 添加适当的日志输出
- 添加错误处理机制
- **脚本自包含原则**：脚本应当实现所有所需功能，包括激活conda环境和退出conda环境，避免依赖外部手动环境管理
- **环境管理**：所有脚本应在开头使用`source activate cuttag`激活环境，必要时在结束时使用`conda deactivate`退出环境
- **自动化优先**：尽量减少人工干预，脚本应能够自动完成从输入到输出的全部过程
- **SGE作业脚本标准格式**：
  - 所有提交到SGE调度系统的脚本应遵循以下格式：
    ```bash
    #!/bin/bash
    #$ -S /bin/bash
    #$ -N job_name        # 作业名
    #$ -cwd               # 使用当前工作目录
    #$ -j y               # 合并标准输出和错误输出
    #$ -o logs/$JOB_NAME.o$JOB_ID  # 输出文件路径
    #$ -l h_vmem=8G       # 申请内存资源
    #$ -pe smp 4          # 申请CPU线程数
    
    # 脚本正文开始
    source activate cuttag
    
    # 脚本逻辑...
    ```
  - 根据任务资源需求调整内存(-l h_vmem)和CPU线程(-pe smp)参数
  - 确保logs目录存在，或在脚本中创建该目录
  - 作业名应能清晰反映脚本功能和处理的样本
- **SGE作业依赖管理**：
  - 对于有依赖关系的作业，使用`-hold_jid`参数指定依赖的作业ID
  - 示例：`#$ -hold_jid previous_job_name`
  - 复杂流程应创建作业依赖链，确保按正确顺序执行
- **依赖检查**：
  - 在脚本开头添加软件依赖检查代码，确保所有必要工具已正确安装
  - 如发现依赖缺失，提供清晰的错误信息并退出脚本
  - 依赖检查代码示例：
    ```bash
    # 检查必要的软件依赖
    check_dependencies() {
        local missing_tools=()
        
        for tool in fastqc cutadapt bowtie2 samtools seqtk; do
            if ! command -v $tool &> /dev/null; then
                missing_tools+=("$tool")
            fi
        done
        
        if [ ${#missing_tools[@]} -ne 0 ]; then
            echo "错误: 以下必要工具未安装或未添加到PATH中:"
            for tool in "${missing_tools[@]}"; do
                echo "  - $tool"
            done
            echo "请确保已激活正确的conda环境(cuttag)，或安装缺失的工具。"
            exit 1
        fi
    }
    
    # 执行依赖检查
    check_dependencies
    ```
  - 应在激活conda环境后立即执行依赖检查
  - 根据脚本功能调整所需检查的具体工具列表
- **功能单一原则**：
  - 每个脚本应专注于完成单一的分析任务
  - 不同的分析步骤应拆分为独立的脚本（如质控、比对、峰值检测等）
  - 避免在一个脚本中混合多个独立的分析任务
  - 示例：
    - ✓ `01_fastqc.sh`：仅执行质控分析
    - ✓ `02_trimming.sh`：仅执行接头去除
    - ✗ `01_fastqc_and_trimming.sh`：不推荐在一个脚本中同时执行质控和接头去除
  - 好处：
    - 提高代码可读性和可维护性
    - 方便调试和错误定位
    - 支持灵活的任务组合和重用
    - 便于并行执行不同任务

### R脚本规范
- 遵循PEP8编码规范
- 函数和变量命名使用snake_case格式
- 添加详细的注释说明代码逻辑
- 使用相对路径引用数据文件
- 使用tidyverse风格进行数据处理

## 5. 工作环境与文件夹用途

### 环境划分
- **服务器环境**：用于执行计算密集型任务，如测序数据比对、峰值检测等
- **本地环境**：用于数据可视化、统计分析和结果整合等

### 运行环境规范
- **严格分离原则**：CutTag_Linux目录下的脚本**只能在服务器环境运行**，不得在本地环境执行
- **环境特异性**：服务器脚本依赖特定的计算资源和环境配置，在本地运行可能导致错误或资源耗尽
- **本地脚本限制**：CutTag_local_analysis目录下的脚本只应在本地环境运行，不应在服务器上执行
- **跨环境调用**：如需在不同环境间传递数据，应使用专门的数据传输脚本，不应直接调用其他环境的脚本

### conda环境配置
- **服务器端环境名称**：`cuttag`
- **环境激活方式**：所有服务器脚本中使用 `source activate cuttag` 激活环境
- **环境主要组件**：
  - 质控工具：fastqc、multiqc、cutadapt等
  - 比对工具：bowtie2、samtools等
  - 峰值检测：macs2、seacr等
  - 文件处理：bedtools、deeptools等
  - 并行工具：GNU parallel
- **环境管理**：
  - 脚本开头统一添加环境激活命令
  - 确保所有依赖工具在环境中正确安装
  - 环境变量统一在配置文件中管理

### 文件夹用途说明
- **CutTag_Linux/**: 
  - 包含在本地撰写但在**服务器上运行**的脚本
  - 处理原始测序数据的工作流程
  - 包含配置文件和各步骤的shell脚本
  
- **reference_repos/**: 
  - 存放从外部获取的**参考分析流程**
  - 作为学习和参考用途，不直接参与实际分析
  - 用于比较和改进当前分析流程 

- **CutTag_local_analysis/**: 
  - 专门用于**本地执行**的分析脚本和结果
  - 包含R脚本进行下游分析、可视化和统计
  - 处理从服务器获取的中间结果文件

## 6. 并行处理规范

> **注意**: 以下并行处理规范**仅适用于CutTag_Linux目录下的脚本**，这些脚本在服务器上执行。local_analysis目录下的本地分析脚本不适用此规范。

### 基本原则
- 所有计算密集型任务均应使用SGE调度系统进行并行处理
- 根据任务特性选择合适的并行方式
- 避免在登录节点上直接运行计算密集型任务

### SGE并行实现方式
1. **数组作业方式**
   - 适用于处理多个独立样本的相同任务
   - 使用`#$ -t 1-N`指定作业数组大小，N为样本数量
   - 使用`SGE_TASK_ID`环境变量访问当前任务ID
   - 在脚本中通过任务ID关联到对应样本
   - 示例：
     ```bash
     #!/bin/bash
     #$ -S /bin/bash
     #$ -N fastqc_array
     #$ -cwd
     #$ -j y
     #$ -o logs/fastqc_$TASK_ID.log
     #$ -t 1-24  # 处理24个样本
     
     # 获取样本列表
     source ./config/00_config.sh
     
     # 获取当前任务对应的样本
     current_sample=${SAMPLES[$SGE_TASK_ID-1]}
     
     # 针对当前样本执行处理
     process_sample ${current_sample}
     ```

2. **样本拆分作业方式**
   - 为每个样本创建单独的作业脚本
   - 使用脚本生成器自动创建每个样本的作业脚本
   - 批量提交所有样本的作业
   - 示例：
     ```bash
     #!/bin/bash
     # 作业生成脚本
     
     source ./config/00_config.sh
     
     # 创建作业脚本目录

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lucyyang1991/Cuttag_RNA-seq](https://github.com/Lucyyang1991/Cuttag_RNA-seq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
