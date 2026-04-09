---
trigger: always_on
description: Pinia Store 开发规范
---


# Pinia Store 规范

## Store 结构

### 使用 Setup Store 模式

```typescript
// ✅ 推荐：Setup Store（类似 Composition API）
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { Torrent } from '@/api/types'
import { torrentApi } from '@/api'

export const useTorrentStore = defineStore('torrent', () => {
  // State
  const torrents = ref<Torrent[]>([])
  const isLoading = ref(false)
  const error = ref<Error | null>(null)
  
  // Getters
  const activeTorrents = computed(() => 
    torrents.value.filter(t => t.state === 'downloading')
  )
  
  const totalSize = computed(() =>
    torrents.value.reduce((sum, t) => sum + t.size, 0)
  )
  
  // Actions
  async function fetchTorrents() {
    isLoading.value = true
    error.value = null
    
    try {
      const data = await torrentApi.getTorrents()
      torrents.value = data
    } catch (e) {
      error.value = e as Error
      console.error('Failed to fetch torrents:', e)
      window.$message?.error('获取种子列表失败')
      throw e
    } finally {
      isLoading.value = false
    }
  }
  
  async function pauseTorrent(hash: string) {
    try {
      await torrentApi.pauseTorrent(hash)
      
      // 更新本地状态
      const torrent = torrents.value.find(t => t.hash === hash)
      if (torrent) {
        torrent.state = 'pausedDL'
      }
      
      window.$message?.success('暂停成功')
    } catch (e) {
      console.error('Failed to pause torrent:', e)
      window.$message?.error('暂停失败')
      throw e
    }
  }
  
  return {
    // State
    torrents,
    isLoading,
    error,
    // Getters
    activeTorrents,
    totalSize,
    // Actions
    fetchTorrents,
    pauseTorrent
  }
})
```

## 命名规范

### Store 命名

```typescript
// ✅ 使用 use + 名词 + Store
export const useTorrentStore = defineStore('torrent', () => { })
export const useSettingStore = defineStore('setting', () => { })
export const useSessionStore = defineStore('session', () => { })
```

### State 命名

```typescript
// ✅ 使用描述性名称
const torrents = ref<Torrent[]>([])
const isLoading = ref(false)
const error = ref<Error | null>(null)
const selectedHashes = ref<Set<string>>(new Set())

// ❌ 避免过于简短
const data = ref([])
const loading = ref(false)
```

### Getter 命名

```typescript
// ✅ 使用形容词或名词
const activeTorrents = computed(() => { })
const downloadingCount = computed(() => { })
const hasError = computed(() => error.value !== null)

// ❌ 避免使用 get 前缀（computed 已经是 getter）
const getActiveTorrents = computed(() => { })
```

### Action 命名

```typescript
// ✅ 使用动词开头
async function fetchTorrents() { }
async function pauseTorrent() { }
async function deleteTorrent() { }
function updateFilter() { }

// ❌ 避免 get/set 前缀（使用更具体的动词）
async function getTorrents() { } // 使用 fetchTorrents
function setFilter() { }         // 使用 updateFilter
```

## 错误处理

### 标准错误处理模式

```typescript
async function fetchTorrents() {
  isLoading.value = true
  error.value = null
  
  try {
    const data = await torrentApi.getTorrents()
    torrents.value = data
  } catch (e) {
    error.value = e as Error
    console.error('Failed to fetch torrents:', e)
    window.$message?.error('获取种子列表失败')
    throw e // 重新抛出以便调用方处理
  } finally {
    isLoading.value = false
  }
}
```

### 乐观更新模式

```typescript
// ✅ 乐观更新：先更新 UI，失败后回滚
async function toggleTorrentPause(hash: string) {
  const torrent = torrents.value.find(t => t.hash === hash)
  if (!torrent) {
    return
  }
  
  // 保存原始状态
  const originalState = torrent.state
  
  // 乐观更新
  torrent.state = originalState === 'downloading' ? 'pausedDL' : 'downloading'
  
  try {
    if (originalState === 'downloading') {
      await torrentApi.pauseTorrent(hash)
    } else {
      await torrentApi.resumeTorrent(hash)
    }
  } catch (e) {
    // 回滚
    torrent.state = originalState
    console.error('Failed to toggle pause:', e)
    window.$message?.error('操作失败')
    throw e
  }
}
```

## State 管理最佳实践

### 1. 避免冗余状态

```typescript
// ❌ 错误：存储可计算的值
const torrents = ref<Torrent[]>([])
const torrentCount = ref(0) // 冗余！

// ✅ 正确：使用 computed
const torrents = ref<Torrent[]>([])
const torrentCount = computed(() => torrents.value.length)
```

### 2. 使用规范化状态

```typescript
// ✅ 对于大型列表，使用 Map 存储
const torrentsMap = ref<Map<string, Torrent>>(new Map())

// Getter 返回数组
const torrents = computed(() => Array.from(torrentsMap.value.values()))

// 快速查找
function getTorrent(hash: string) {
  return torrentsMap.value.get(hash)
}

// 快速更新
function updateTorrent(hash: string, updates: Partial<Torrent>) {
  const torrent = torrentsMap.value.get(hash)
  if (torrent) {
    Object.assign(torrent, updates)
  }
}
```

### 3. 分离加载状态

```typescript
// ✅ 区分不同操作的加载状态
const isLoadingList = ref(false)
const isLoadingDetail = ref(false)
const isDeletingTorrent = ref(false)

// ❌ 避免单一加载状态
const isLoading = ref(false) // 无法区分具体操作
```

## Store 组合

### 在 Store 中使用其他 Store

```typescript
export const useTorrentStore = defineStore('torrent', () => {
  const settingStore = useSettingStore()
  const sessionStore = useSessionStore()
  
  // 使用其他 store 的状态
  const sortedTorrents = computed(() => {
    const { sortBy, sortOrder } = settingStore
    return sortTorrents(torrents.value, sortBy, sortOrder)
  })
  
  return {
    torrents,
    sortedTorrents
  }
})
```

## 持久化

### 使用 pinia-plugin-persistedstate

```typescript
import { defineStore } from 'pinia'


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jianxcao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
