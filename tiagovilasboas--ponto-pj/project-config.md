---
trigger: always_on
description: Este arquivo contém todas as boas práticas e padrões arquiteturais implementados neste projeto. Use estas regras como guia para manter a qualidade e consistência do código.
---

# 🚀 Ponto PJ - Cursor Rules & Best Practices

Este arquivo contém todas as boas práticas e padrões arquiteturais implementados neste projeto. Use estas regras como guia para manter a qualidade e consistência do código.

## 🏗️ Arquitetura & Padrões

### Clean Architecture
- **Separação de responsabilidades**: Cada camada tem uma função específica
- **Presentation Layer**: React components, hooks, stores
- **Business Logic Layer**: Services, validation, error handling
- **Data Access Layer**: Repositories, cache, external APIs

### Repository Pattern
- **BaseRepository**: Classe abstrata com funcionalidades comuns
- **CachedRepository**: Extensão com sistema de cache inteligente
- **Especialização**: Cada entidade tem seu próprio repository
- **Validação**: Integrada nos métodos create, update, upsert

### SOLID Principles
- **S** - Single Responsibility: Cada classe tem uma única responsabilidade
- **O** - Open/Closed: Extensível sem modificação
- **L** - Liskov Substitution: Repositories intercambiáveis
- **I** - Interface Segregation: Interfaces específicas
- **D** - Dependency Inversion: Dependências injetadas

## 📁 Estrutura de Arquivos

### Organização por Responsabilidade
```
src/
├── components/     # UI Components (apenas apresentação)
├── hooks/         # Custom hooks (lógica de UI)
├── services/      # Business logic (orquestração)
├── repositories/  # Data access (comunicação com APIs)
├── stores/        # State management (apenas estado)
├── lib/           # Utilities (funções puras)
├── types/         # TypeScript definitions
└── test/          # Testes organizados por tipo
```

### Convenções de Nomenclatura
- **Components**: PascalCase (ex: `WorkSessionCard.tsx`)
- **Hooks**: camelCase com prefixo `use` (ex: `useWorkSession.ts`)
- **Services**: camelCase com sufixo `Service` (ex: `workSessionService.ts`)
- **Repositories**: camelCase com sufixo `Repository` (ex: `workSessionRepository.ts`)
- **Stores**: camelCase com sufixo `Store` (ex: `workSessionStore.ts`)
- **Types**: PascalCase (ex: `WorkSession.ts`)
- **Constants**: UPPER_SNAKE_CASE (ex: `API_ENDPOINTS.ts`)

## 🎯 Padrões de Código

### TypeScript
- **Tipagem forte**: Sempre use tipos explícitos
- **Interfaces**: Para estruturas de dados
- **Types**: Para unions, intersections, mapped types
- **Generics**: Para componentes e funções reutilizáveis
- **Strict mode**: Sempre habilitado

### React Components
```typescript
// ✅ Padrão correto
interface ComponentProps {
  data: WorkSession
  onAction: (id: string) => void
  loading?: boolean
}

export function Component({ data, onAction, loading = false }: ComponentProps) {
  // Lógica do componente
  return <div>...</div>
}

// ❌ Evitar
export function Component(props: any) {
  // Sem tipagem
}
```

### Custom Hooks
```typescript
// ✅ Padrão correto
export function useWorkSession(userId: string) {
  const [data, setData] = useState<WorkSession | null>(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  // Lógica do hook
  return { data, loading, error, refetch }
}

// ❌ Evitar
export function useWorkSession() {
  // Sem parâmetros tipados
  // Sem retorno tipado
}
```

### Services
```typescript
// ✅ Padrão correto
export class WorkSessionService {
  async getSessionByDate(date: string): Promise<WorkSession | null> {
    try {
      const userId = await workSessionRepository.getCurrentUserId()
      return await workSessionRepository.findByUserAndDate(userId, date)
    } catch (error) {
      console.error('Erro ao buscar sessão por data:', error)
      throw error
    }
  }
}

// ❌ Evitar
export class WorkSessionService {
  async getSessionByDate(date: any) {
    // Sem tipagem
    // Sem tratamento de erro
    return await supabase.from('work_sessions').select('*')
  }
}
```

### Repositories
```typescript
// ✅ Padrão correto
export class WorkSessionRepository extends CachedRepository {
  protected tableName = 'work_sessions'

  async findByUserAndDate(userId: string, date: string): Promise<WorkSession | null> {
    const cacheKey = this.generateCacheKey('session', userId, date)
    
    return this.getCached(cacheKey, async () => {
      const { data, error } = await this.getSupabase()
        .from(this.tableName)
        .select('*')
        .eq('user_id', userId)
        .eq('date', date)
        .single()

      if (error && error.code !== 'PGRST116') {
        this.handleError(error, 'findByUserAndDate')
      }

      return data
    }, { ttl: 2 * 60 * 1000 })
  }
}

// ❌ Evitar
export class WorkSessionRepository {
  async findByUserAndDate(userId: any, date: any) {
    // Sem cache
    // Sem tratamento de erro padronizado
    // Sem tipagem
  }
}
```

## 🔒 Validação & Tratamento de Erros

### Sistema de Validação
```typescript
// ✅ Usar validadores específicos
const validation = WorkSessionValidator.validateCreateData(data)
if (!validation.isValid) {
  throw new Error(`validation.error: ${validation.errors.join(', ')}`)
}

// ❌ Evitar validação manual
if (!data.user_id) {
  throw new Error('User ID is required')
}
```

### Tratamento de Erros
```typescript
// ✅ Usar ErrorHandler centralizado

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tiagovilasboas/ponto-pj](https://github.com/tiagovilasboas/ponto-pj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
