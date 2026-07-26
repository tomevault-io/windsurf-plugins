---
trigger: always_on
description: TypeScript coding standards and type safety guidelines for enterprise applications
---


# 📝 TypeScript 编码标准 - TypeScript Coding Standards

## 🏗️ 类型定义标准 (Type Definition Standards)

### 🎯 接口设计原则

```typescript
// ✅ 良好的接口设计
interface WeatherData {
  readonly location: {
    readonly name: string
    readonly country: string
    readonly coordinates: {
      readonly lat: number
      readonly lon: number
    }
  }
  readonly current: CurrentWeather
  readonly snowDay: SnowDayPrediction
  readonly timestamp: string
}

interface CurrentWeather {
  readonly temperature: number
  readonly feelsLike: number
  readonly humidity: number
  readonly pressure: number
  readonly description: string
  readonly icon: string
  readonly windSpeed: number
  readonly windSpeedKmh: number
  readonly visibility: number
  readonly visibilityKm: number
  readonly snowfall: number
  readonly cloudCover: number
}

interface SnowDayPrediction {
  readonly probability: number
  readonly level: SnowDayLevel
  readonly color: string
  readonly recommendation: string
  readonly factors: SnowDayFactors
}

// 使用字面量类型确保类型安全
type SnowDayLevel = "Very Low" | "Low" | "Moderate" | "High" | "Very High"
type SearchType = "zip" | "city" | "coords"
type LoadingState = "idle" | "loading" | "success" | "error"
```

### 🔧 泛型与实用类型

```typescript
// 通用API响应类型
interface ApiResponse<T> {
  readonly data: T
  readonly success: boolean
  readonly message?: string
  readonly timestamp: string
}

interface ApiError {
  readonly error: string
  readonly code?: number
  readonly details?: Record<string, unknown>
}

// 条件类型用于API状态
type ApiResult<T> = ApiResponse<T> | ApiError

// 实用类型组合
type PartialUpdate<T> = Partial<Pick<T, keyof T>>
type RequiredFields<T, K extends keyof T> = T & Required<Pick<T, K>>

// 状态管理类型
interface AsyncState<T> {
  readonly data: T | null
  readonly loading: boolean
  readonly error: string | null
  readonly lastUpdated: Date | null
}

// Hook返回类型
interface UseAsyncResult<T> extends AsyncState<T> {
  readonly execute: (params?: unknown) => Promise<void>
  readonly reset: () => void
}
```

### 🎨 React组件类型

```typescript
// 组件Props类型定义
interface ButtonProps {
  readonly variant?: "primary" | "secondary" | "danger"
  readonly size?: "sm" | "md" | "lg"
  readonly loading?: boolean
  readonly disabled?: boolean
  readonly children: React.ReactNode
  readonly onClick?: (event: React.MouseEvent<HTMLButtonElement>) => void
  readonly className?: string
  readonly type?: "button" | "submit" | "reset"
}

// 泛型组件类型
interface SelectOption<T = string> {
  readonly value: T
  readonly label: string
  readonly disabled?: boolean
}

interface SelectProps<T> {
  readonly options: SelectOption<T>[]
  readonly value: T | null
  readonly onChange: (value: T) => void
  readonly placeholder?: string
  readonly disabled?: boolean
  readonly multiple?: boolean
}

// 表单相关类型
interface FormFieldProps<T> {
  readonly name: keyof T
  readonly value: T[keyof T]
  readonly onChange: (name: keyof T, value: T[keyof T]) => void
  readonly error?: string
  readonly required?: boolean
  readonly disabled?: boolean
}

// 高阶组件类型
type WithLoadingProps<T> = T & {
  readonly isLoading: boolean
}

type HOC<TOriginalProps, TInjectedProps = {}> = (
  Component: React.ComponentType<TOriginalProps>
) => React.ComponentType<TOriginalProps & TInjectedProps>
```

## 🎯 函数与Hook类型安全

### 🪝 自定义Hook类型

```typescript
// 异步数据获取Hook
function useAsyncData<T>(
  fetcher: () => Promise<T>,
  dependencies: React.DependencyList = []
): UseAsyncResult<T> {
  const [state, setState] = useState<AsyncState<T>>({
    data: null,
    loading: false,
    error: null,
    lastUpdated: null,
  })

  const execute = useCallback(async () => {
    setState((prev) => ({ ...prev, loading: true, error: null }))

    try {
      const data = await fetcher()
      setState({
        data,
        loading: false,
        error: null,
        lastUpdated: new Date(),
      })
    } catch (error) {
      setState((prev) => ({
        ...prev,
        loading: false,
        error: error instanceof Error ? error.message : "Unknown error",
      }))
    }
  }, dependencies)

  const reset = useCallback(() => {
    setState({
      data: null,
      loading: false,
      error: null,
      lastUpdated: null,
    })
  }, [])

  useEffect(() => {
    execute()
  }, [execute])

  return { ...state, execute, reset }
}

// 本地存储Hook
function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T | ((prev: T) => T)) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === "undefined") return initialValue

    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch (error) {
      console.warn(`Error reading localStorage key "${key}":`, error)
      return initialValue
    }
  })

  const setValue = useCallback(
    (value: T | ((prev: T) => T)) => {
      try {
        const valueToStore = value instanceof Function ? value(storedValue) : value
        setStoredValue(valueToStore)

        if (typeof window !== "undefined") {
          window.localStorage.setItem(key, JSON.stringify(valueToStore))
        }
      } catch (error) {
        console.warn(`Error setting localStorage key "${key}":`, error)
      }
    },
    [key, storedValue]
  )


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geekskai/blog](https://github.com/geekskai/blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
