---
trigger: always_on
description: ├── index.ts          # 导出所有stores
---

# 状态管理与API服务规范

## Pinia 状态管理架构

### Store 文件组织
```
src/stores/
├── index.ts          # 导出所有stores
├── app.ts           # 应用全局状态
├── project.ts       # 项目相关状态
├── component.ts     # 组件管理状态
├── user.ts          # 用户状态
├── cache.ts         # 缓存管理
└── settings.ts      # 用户设置
```

### Store 定义规范

#### 基础Store结构
```typescript
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { ProjectInfo, ComponentConfig } from '@/types'

export const useProjectStore = defineStore('project', () => {
  // State - 使用 ref 定义响应式状态
  const currentProject = ref<ProjectInfo | null>(null)
  const projects = ref<ProjectInfo[]>([])
  const selectedComponent = ref<ComponentConfig | null>(null)
  const draggedComponent = ref<ComponentConfig | null>(null)
  const loading = ref(false)

  // Getters - 使用 computed 定义计算属性
  const hasCurrentProject = computed(() => currentProject.value !== null)
  const projectCount = computed(() => projects.value.length)
  const selectedComponentId = computed(() => selectedComponent.value?.id)

  // Actions - 定义修改状态的方法
  const setCurrentProject = (project: ProjectInfo | null) => {
    currentProject.value = project
  }

  const addProject = (project: ProjectInfo) => {
    projects.value.push(project)
  }

  const updateProject = (id: string, updates: Partial<ProjectInfo>) => {
    const index = projects.value.findIndex(p => p.id === id)
    if (index !== -1) {
      projects.value[index] = { ...projects.value[index], ...updates }
    }
  }

  const removeProject = (id: string) => {
    const index = projects.value.findIndex(p => p.id === id)
    if (index !== -1) {
      projects.value.splice(index, 1)
    }
  }

  const selectComponent = (component: ComponentConfig | null) => {
    selectedComponent.value = component
  }

  // 异步操作
  const loadProjects = async () => {
    loading.value = true
    try {
      const response = await projectApi.getProjects()
      projects.value = response.data
    } catch (error) {
      console.error('加载项目失败:', error)
      throw error
    } finally {
      loading.value = false
    }
  }

  const saveProject = async (project: ProjectInfo) => {
    loading.value = true
    try {
      if (project.id) {
        await projectApi.updateProject(project.id, project)
        updateProject(project.id, project)
      } else {
        const response = await projectApi.createProject(project)
        addProject(response.data)
      }
    } catch (error) {
      console.error('保存项目失败:', error)
      throw error
    } finally {
      loading.value = false
    }
  }

  // 清理方法
  const reset = () => {
    currentProject.value = null
    projects.value = []
    selectedComponent.value = null
    draggedComponent.value = null
    loading.value = false
  }

  return {
    // State
    currentProject,
    projects,
    selectedComponent,
    draggedComponent,
    loading,
    // Getters
    hasCurrentProject,
    projectCount,
    selectedComponentId,
    // Actions
    setCurrentProject,
    addProject,
    updateProject,
    removeProject,
    selectComponent,
    loadProjects,
    saveProject,
    reset
  }
})
```

#### 应用全局状态
```typescript
// src/stores/app.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'

export const useAppStore = defineStore('app', () => {
  // 主题设置
  const theme = ref<'light' | 'dark'>('light')
  const language = ref<'zh-CN' | 'en-US'>('zh-CN')
  
  // 界面状态
  const sidebarCollapsed = ref(false)
  const loading = ref(false)
  const fullscreen = ref(false)
  
  // 通知和消息
  const notifications = ref<Array<{
    id: string
    type: 'info' | 'success' | 'warning' | 'error'
    title: string
    message: string
    timestamp: number
  }>>([])

  // Actions
  const toggleTheme = () => {
    theme.value = theme.value === 'light' ? 'dark' : 'light'
    // 保存到本地存储
    localStorage.setItem('theme', theme.value)
  }

  const setLanguage = (lang: 'zh-CN' | 'en-US') => {
    language.value = lang
    localStorage.setItem('language', lang)
  }

  const toggleSidebar = () => {
    sidebarCollapsed.value = !sidebarCollapsed.value
  }

  const setLoading = (state: boolean) => {
    loading.value = state
  }

  const addNotification = (notification: Omit<typeof notifications.value[0], 'id' | 'timestamp'>) => {
    const newNotification = {
      ...notification,
      id: Date.now().toString(),
      timestamp: Date.now()
    }
    notifications.value.unshift(newNotification)
    
    // 自动清理旧通知（保留最新50条）
    if (notifications.value.length > 50) {
      notifications.value = notifications.value.slice(0, 50)
    }
  }

  const removeNotification = (id: string) => {
    const index = notifications.value.findIndex(n => n.id === id)
    if (index !== -1) {
      notifications.value.splice(index, 1)
    }
  }

  // 初始化方法
  const initializeApp = () => {
    // 从本地存储恢复设置
    const savedTheme = localStorage.getItem('theme') as 'light' | 'dark'
    if (savedTheme) {
      theme.value = savedTheme
    }
    
    const savedLanguage = localStorage.getItem('language') as 'zh-CN' | 'en-US'
    if (savedLanguage) {
      language.value = savedLanguage
    }
  }

  return {
    theme,
    language,
    sidebarCollapsed,
    loading,
    fullscreen,
    notifications,
    toggleTheme,
    setLanguage,
    toggleSidebar,
    setLoading,
    addNotification,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suyancc/easy_window](https://github.com/suyancc/easy_window) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
