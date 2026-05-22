---
trigger: always_on
description: Reglas de diseño y desarrollo para Fintech Backend
---


# Fintech Backend - Reglas de Desarrollo

## 🏗️ Arquitectura y Estructura

### Clean Architecture
- Mantener separación clara entre capas: handlers, use cases, entities, repositories
- No mezclar lógica de negocio con detalles de implementación
- Dependencias deben apuntar hacia el centro (domain)

### Estructura de Directorios
```
internal/
├── controller/http/v1/     # Handlers HTTP
├── usecase/               # Lógica de negocio
├── entity/                # Entidades de dominio
├── repository/            # Implementaciones de repositorio
└── app/                   # Configuración de la aplicación
```

## 📝 Estándares de Código

### Nomenclatura
- **Handlers**: Terminar con `Handler` (ej: `ExpenseHandler`)
- **Use Cases**: Usar verbos descriptivos (ej: `CreateExpense`, `GetUserBudget`)
- **Entities**: Nombres singulares (ej: `User`, `Expense`, `Budget`)
- **DTOs**: Terminar con el propósito (ej: `CreateExpenseRequest`, `ExpenseResponse`)

### Manejo de Errores
```go
// ✅ CORRECTO: Errores descriptivos
if err != nil {
    return nil, fmt.Errorf("failed to create expense: %w", err)
}

// ✅ CORRECTO: Errores de dominio
var ErrExpenseNotFound = errors.New("expense not found")

// ❌ INCORRECTO: Errores genéricos
if err != nil {
    return nil, err
}
```

### Validación de Datos
```go
// ✅ CORRECTO: Validar en el handler
func (h *ExpenseHandler) CreateExpense(c *gin.Context) {
    var req dto.CreateExpenseRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": "invalid request"})
        return
    }
    
    if err := h.validator.Validate(req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
}
```

## 🔒 Seguridad y Autenticación

### JWT y Middleware
- Siempre validar tokens JWT en endpoints protegidos
- Extraer user ID del contexto autenticado
- No exponer información sensible en logs

```go
// ✅ CORRECTO: Obtener user ID del contexto
userID, exists := c.Get("user_id")
if !exists {
    c.JSON(401, gin.H{"error": "unauthorized"})
    return
}
```

### Validación de Permisos
- Verificar que el usuario tiene acceso a los recursos solicitados
- Validar ownership de datos antes de operaciones CRUD

## 💾 Base de Datos

### Transacciones
```go
// ✅ CORRECTO: Usar transacciones para operaciones complejas
tx := r.db.Begin()
defer func() {
    if r := recover(); r != nil {
        tx.Rollback()
    }
}()

if err := tx.Create(&expense).Error; err != nil {
    tx.Rollback()
    return err
}

if err := tx.Create(&budgetUpdate).Error; err != nil {
    tx.Rollback()
    return err
}

return tx.Commit().Error
```

### Migraciones
- Siempre crear migraciones para cambios de esquema
- Nombrar migraciones con timestamp y descripción clara
- Incluir rollback en todas las migraciones

## 📊 DTOs y Responses

### Estructura Consistente
```go
// ✅ CORRECTO: Response estructurado
type ExpenseResponse struct {
    ID          uint      `json:"id"`
    Amount      float64   `json:"amount"`
    Description string    `json:"description"`
    CategoryID  uint      `json:"category_id"`
    Category    Category  `json:"category"`
    CreatedAt   time.Time `json:"created_at"`
}

// ✅ CORRECTO: Request con validaciones
type CreateExpenseRequest struct {
    Amount      float64 `json:"amount" validate:"required,gt=0"`
    Description string  `json:"description" validate:"required"`
    CategoryID  uint    `json:"category_id" validate:"required"`
}
```

### Paginación Estándar
```go
type PaginatedResponse struct {
    Data       interface{} `json:"data"`
    Total      int64      `json:"total"`
    Page       int        `json:"page"`
    PerPage    int        `json:"per_page"`
    TotalPages int        `json:"total_pages"`
}
```

## 🧪 Testing

### Estructura de Tests
```go
func TestExpenseHandler_CreateExpense(t *testing.T) {
    tests := []struct {
        name           string
        request        dto.CreateExpenseRequest
        expectedStatus int
        expectedError  string
    }{
        {
            name: "valid expense creation",
            request: dto.CreateExpenseRequest{
                Amount:      100.0,
                Description: "Test expense",
                CategoryID:  1,
            },
            expectedStatus: 201,
        },
        // ... más casos
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Test implementation
        })
    }
}
```

### Mocks y Dependencias
- Usar interfaces para todas las dependencias
- Crear mocks para testing unitario
- Tests de integración para endpoints completos

## 📡 API Design

### RESTful Endpoints
```
GET    /api/v1/expenses              # Listar gastos
POST   /api/v1/expenses              # Crear gasto
GET    /api/v1/expenses/:id          # Obtener gasto específico
PUT    /api/v1/expenses/:id          # Actualizar gasto
DELETE /api/v1/expenses/:id          # Eliminar gasto
```

### Status Codes Estándar
- `200` - OK (GET, PUT exitosos)
- `201` - Created (POST exitoso)
- `204` - No Content (DELETE exitoso)
- `400` - Bad Request (validación falló)
- `401` - Unauthorized (no autenticado)
- `403` - Forbidden (no autorizado)
- `404` - Not Found (recurso no existe)
- `500` - Internal Server Error

### Headers de Response
```go
// ✅ CORRECTO: Headers consistentes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nick130920/fintech-backend](https://github.com/nick130920/fintech-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
