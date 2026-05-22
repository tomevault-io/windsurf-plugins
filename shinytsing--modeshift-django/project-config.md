---
trigger: always_on
description: Shell脚本语言特定规则
---


# Shell脚本语言特定规则

基于awesome-cursorrules的Shell脚本最佳实践。

## 基本规范

### Shebang和编码
```bash
#!/bin/bash
# 使用bash作为解释器
# 设置UTF-8编码

set -euo pipefail
# -e: 遇到错误立即退出
# -u: 使用未定义变量时报错
# -o pipefail: 管道中任何命令失败都会导致整个管道失败
```

### 变量定义和使用
```bash
# 使用大写字母定义常量
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly LOG_FILE="/var/log/${SCRIPT_NAME}.log"

# 使用小写字母定义变量
local_variable="value"
another_variable="${local_variable}_suffix"

# 使用引号包围变量
echo "Hello, ${USER}!"
echo "Current directory: ${PWD}"
```

## 函数定义

### 函数结构
```bash
# 函数定义
function log_info() {
    local message="$1"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[${timestamp}] INFO: ${message}" | tee -a "${LOG_FILE}"
}

function log_error() {
    local message="$1"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[${timestamp}] ERROR: ${message}" >&2
    echo "[${timestamp}] ERROR: ${message}" >> "${LOG_FILE}"
}

# 带返回值的函数
function check_file_exists() {
    local file_path="$1"
    if [[ -f "${file_path}" ]]; then
        return 0
    else
        return 1
    fi
}
```

### 参数处理
```bash
function process_arguments() {
    local help_flag=false
    local verbose_flag=false
    local input_file=""
    local output_file=""
    
    while [[ $# -gt 0 ]]; do
        case $1 in
            -h|--help)
                help_flag=true
                shift
                ;;
            -v|--verbose)
                verbose_flag=true
                shift
                ;;
            -i|--input)
                input_file="$2"
                shift 2
                ;;
            -o|--output)
                output_file="$2"
                shift 2
                ;;
            *)
                echo "Unknown option: $1" >&2
                exit 1
                ;;
        esac
    done
    
    # 验证必需参数
    if [[ -z "${input_file}" ]]; then
        echo "Error: Input file is required" >&2
        exit 1
    fi
}
```

## 错误处理

### 错误捕获和处理
```bash
function cleanup() {
    local exit_code=$?
    log_info "Cleaning up..."
    
    # 清理临时文件
    if [[ -n "${TEMP_DIR:-}" && -d "${TEMP_DIR}" ]]; then
        rm -rf "${TEMP_DIR}"
        log_info "Temporary directory removed: ${TEMP_DIR}"
    fi
    
    # 停止后台进程
    if [[ -n "${BACKGROUND_PID:-}" ]]; then
        kill "${BACKGROUND_PID}" 2>/dev/null || true
        log_info "Background process stopped: ${BACKGROUND_PID}"
    fi
    
    exit "${exit_code}"
}

# 设置清理陷阱
trap cleanup EXIT INT TERM

# 错误处理函数
function handle_error() {
    local line_number="$1"
    local error_code="$2"
    log_error "Error on line ${line_number}: exit code ${error_code}"
    exit "${error_code}"
}

trap 'handle_error ${LINENO} $?' ERR
```

### 重试机制
```bash
function retry_command() {
    local max_attempts="$1"
    local delay="$2"
    shift 2
    local command=("$@")
    
    local attempt=1
    while [[ ${attempt} -le ${max_attempts} ]]; do
        log_info "Attempt ${attempt}/${max_attempts}: ${command[*]}"
        
        if "${command[@]}"; then
            log_info "Command succeeded on attempt ${attempt}"
            return 0
        else
            log_error "Command failed on attempt ${attempt}"
            if [[ ${attempt} -lt ${max_attempts} ]]; then
                log_info "Waiting ${delay} seconds before retry..."
                sleep "${delay}"
            fi
        fi
        
        ((attempt++))
    done
    
    log_error "Command failed after ${max_attempts} attempts"
    return 1
}
```

## 文件操作

### 安全的文件操作
```bash
function create_backup() {
    local source_file="$1"
    local backup_dir="${2:-./backups}"
    
    if [[ ! -f "${source_file}" ]]; then
        log_error "Source file does not exist: ${source_file}"
        return 1
    fi
    
    # 创建备份目录
    mkdir -p "${backup_dir}"
    
    # 生成备份文件名
    local timestamp=$(date '+%Y%m%d_%H%M%S')
    local backup_file="${backup_dir}/$(basename "${source_file}").${timestamp}.bak"
    
    # 创建备份
    if cp "${source_file}" "${backup_file}"; then
        log_info "Backup created: ${backup_file}"
        echo "${backup_file}"
    else
        log_error "Failed to create backup"
        return 1
    fi
}

function safe_file_operation() {
    local source="$1"
    local destination="$2"
    
    # 检查源文件
    if [[ ! -f "${source}" ]]; then
        log_error "Source file not found: ${source}"
        return 1
    fi
    
    # 创建目标目录
    local dest_dir=$(dirname "${destination}")
    mkdir -p "${dest_dir}"
    
    # 创建临时文件
    local temp_file=$(mktemp)
    
    # 执行操作
    if cp "${source}" "${temp_file}" && mv "${temp_file}" "${destination}"; then
        log_info "File operation successful: ${source} -> ${destination}"
        return 0
    else
        log_error "File operation failed"
        rm -f "${temp_file}"
        return 1
    fi
}
```

## 进程管理

### 后台进程管理
```bash
function start_background_process() {
    local command="$1"
    local log_file="${2:-/dev/null}"
    
    log_info "Starting background process: ${command}"
    
    # 启动后台进程
    nohup bash -c "${command}" > "${log_file}" 2>&1 &
    BACKGROUND_PID=$!
    
    log_info "Background process started with PID: ${BACKGROUND_PID}"
    
    # 等待进程启动
    sleep 2
    
    # 检查进程是否仍在运行
    if kill -0 "${BACKGROUND_PID}" 2>/dev/null; then

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shinytsing/modeshift_django](https://github.com/shinytsing/modeshift_django) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
