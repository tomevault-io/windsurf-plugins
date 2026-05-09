---
trigger: always_on
description: API 调用和数据处理规范（Transmission RPC）
---


# API 和数据处理规范

针对 Transmission RPC API 的最佳实践

## API 层设计

### API 文件组织
```
api/
├── rpc.ts           # Transmission RPC 核心接口
├── trpc.ts          # tRPC 配置
├── types.ts         # API 类型定义
└── interceptors.ts  # 请求拦截器（可选）
```

### RPC 方法封装
```typescript
// api/rpc.ts
import axios, { AxiosInstance } from 'axios'

class TransmissionRPC {
  private client: AxiosInstance
  private sessionId: string = ''
  
  constructor(baseURL: string) {
    this.client = axios.create({
      baseURL,
      headers: {
        'Content-Type': 'application/json'
      }
    })
    
    this.setupInterceptors()
  }
  
  // ✅ 统一的 RPC 调用方法
  private async call<T = any>(
    method: string, 
    arguments?: Record<string, any>
  ): Promise<T> {
    try {
      const response = await this.client.post('/rpc', {
        method,
        arguments
      }, {
        headers: {
          'X-Transmission-Session-Id': this.sessionId
        }
      })
      
      if (response.data.result !== 'success') {
        throw new Error(response.data.result)
      }
      
      return response.data.arguments as T
    } catch (error) {
      this.handleError(error)
      throw error
    }
  }
  
  // ✅ 具体的 API 方法
  async getTorrents(ids?: number[]): Promise<TorrentInfo[]> {
    const args = ids ? { ids } : {}
    const result = await this.call<{ torrents: TorrentInfo[] }>(
      'torrent-get',
      {
        ...args,
        fields: TORRENT_FIELDS
      }
    )
    return result.torrents
  }
  
  async addTorrent(options: AddTorrentOptions): Promise<TorrentInfo> {
    const result = await this.call<{ torrent-added: TorrentInfo }>(
      'torrent-add',
      options
    )
    return result['torrent-added']
  }
}

export const rpc = new TransmissionRPC(
  import.meta.env.VITE_RPC_URL || '/transmission/rpc'
)
```

## 错误处理

### 统一错误处理
```typescript
// api/errors.ts
export class TransmissionError extends Error {
  constructor(
    message: string,
    public code?: string,
    public statusCode?: number
  ) {
    super(message)
    this.name = 'TransmissionError'
  }
}

export class SessionIdError extends TransmissionError {
  constructor() {
    super('Invalid session ID', 'SESSION_ID_ERROR', 409)
  }
}

export class NetworkError extends TransmissionError {
  constructor(message: string) {
    super(message, 'NETWORK_ERROR')
  }
}

// api/rpc.ts
private handleError(error: any): void {
  // 409 错误：需要更新 Session ID
  if (error.response?.status === 409) {
    const sessionId = error.response.headers['x-transmission-session-id']
    if (sessionId) {
      this.sessionId = sessionId
      throw new SessionIdError()
    }
  }
  
  // 网络错误
  if (!error.response) {
    throw new NetworkError('Network connection failed')
  }
  
  // 其他错误
  throw new TransmissionError(
    error.response?.data?.message || error.message,
    'UNKNOWN_ERROR',
    error.response?.status
  )
}
```

### 组件中的错误处理
```typescript
// ✅ 使用 try-catch 和用户友好的提示
import { useMessage } from 'naive-ui'

const message = useMessage()

async function handleAddTorrent() {
  try {
    await rpc.addTorrent({ filename: magnetLink })
    message.success('种子添加成功')
  } catch (error) {
    if (error instanceof SessionIdError) {
      // 自动重试
      return handleAddTorrent()
    }
    
    if (error instanceof NetworkError) {
      message.error('网络连接失败，请检查连接')
    } else {
      message.error('添加种子失败')
    }
    
    console.error('Failed to add torrent:', error)
  }
}
```

## 数据处理

### JSON 解析
```typescript
// ✅ 始终使用 try-catch 处理 JSON 解析
function parseStoredConfig(): Config {
  try {
    const json = localStorage.getItem('config')
    if (!json) {
      return DEFAULT_CONFIG
    }
    return JSON.parse(json)
  } catch (error) {
    console.error('Failed to parse config:', error)
    return DEFAULT_CONFIG
  }
}

// ✅ 使用类型守卫验证解析结果
function isValidConfig(data: unknown): data is Config {
  return (
    typeof data === 'object' &&
    data !== null &&
    'theme' in data &&
    'language' in data
  )
}

function loadConfig(): Config {
  try {
    const json = localStorage.getItem('config')
    if (!json) return DEFAULT_CONFIG
    
    const data = JSON.parse(json)
    if (isValidConfig(data)) {
      return data
    }
    
    console.warn('Invalid config format, using default')
    return DEFAULT_CONFIG
  } catch (error) {
    console.error('Failed to load config:', error)
    return DEFAULT_CONFIG
  }
}
```

### 数据转换
```typescript
// ✅ 将 API 数据转换为应用数据
interface RpcTorrent {
  id: number
  name: string
  totalSize: number
  downloadedEver: number
  status: number
  rateDownload: number
  rateUpload: number
}

interface TorrentInfo {
  id: number
  name: string
  totalSize: number
  downloadedSize: number
  status: TorrentStatus
  downloadSpeed: number
  uploadSpeed: number
  progress: number
}

// 转换函数
function transformTorrent(rpc: RpcTorrent): TorrentInfo {
  return {
    id: rpc.id,
    name: rpc.name,
    totalSize: rpc.totalSize,
    downloadedSize: rpc.downloadedEver,
    status: getRpcStatus(rpc.status),
    downloadSpeed: rpc.rateDownload,
    uploadSpeed: rpc.rateUpload,
    progress: rpc.totalSize > 0 
      ? (rpc.downloadedEver / rpc.totalSize) * 100 
      : 0
  }
}

// 状态映射
function getRpcStatus(status: number): TorrentStatus {
  const STATUS_MAP: Record<number, TorrentStatus> = {
    0: 'stopped',
    1: 'checking',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jianxcao/transmission-web](https://github.com/jianxcao/transmission-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
