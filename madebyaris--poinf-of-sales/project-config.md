---
trigger: always_on
description: Performance optimization patterns for database queries, React components, and API efficiency in POS System
---


# ⚡ Performance Optimization Patterns

## 🎯 Performance Philosophy for POS Systems

### Critical Performance Metrics
- **Order Creation:** < 500ms from click to confirmation
- **Payment Processing:** < 2s for complete transaction
- **Kitchen Updates:** Real-time (< 100ms propagation)
- **Product Search:** < 200ms for instant results
- **Database Queries:** < 100ms for typical CRUD operations

### Performance Monitoring Strategy
```typescript
// Performance monitoring utilities
class PerformanceMonitor {
  static timeOperation<T>(name: string, operation: () => Promise<T>): Promise<T> {
    console.time(name);
    return operation().finally(() => console.timeEnd(name));
  }

  static measureRender(componentName: string) {
    return (Component: React.ComponentType<any>) => {
      return React.memo(Component, (prevProps, nextProps) => {
        const start = performance.now();
        const shouldUpdate = !Object.is(prevProps, nextProps);
        const end = performance.now();
        
        if (end - start > 1) {
          console.warn(`${componentName} render check took ${end - start}ms`);
        }
        
        return !shouldUpdate;
      });
    };
  }
}
```

## 🗄️ Database Performance Patterns

### Optimized Query Patterns
```go
// ✅ CORRECT: Efficient query with proper indexing
func (h *OrderHandler) GetOrdersWithPagination(c *gin.Context) {
    page := getIntParam(c, "page", 1)
    perPage := getIntParam(c, "per_page", 20)
    status := c.Query("status")
    
    // Use indexed columns in WHERE clause
    query := `
        SELECT 
            o.id, o.order_number, o.status, o.total_amount, o.created_at,
            u.username, t.table_number,
            COUNT(*) OVER() as total_count
        FROM orders o
        LEFT JOIN users u ON o.user_id = u.id
        LEFT JOIN dining_tables t ON o.table_id = t.id
        WHERE ($1 = '' OR o.status = $1)
            AND o.created_at >= CURRENT_DATE - INTERVAL '7 days'
        ORDER BY o.created_at DESC
        LIMIT $2 OFFSET $3
    `
    
    offset := (page - 1) * perPage
    rows, err := h.db.Query(query, status, perPage, offset)
    // ... handle results
}

// ✅ CORRECT: Batch insert for order items
func (h *OrderHandler) CreateOrderWithItems(c *gin.Context) {
    tx, err := h.db.Begin()
    if err != nil {
        // handle error
        return
    }
    defer tx.Rollback()

    // Create order
    var orderID string
    err = tx.QueryRow(`
        INSERT INTO orders (customer_name, order_type, status, total_amount)
        VALUES ($1, $2, $3, $4)
        RETURNING id
    `, req.CustomerName, req.OrderType, "pending", req.TotalAmount).Scan(&orderID)

    // Batch insert order items (much faster than individual inserts)
    if len(req.Items) > 0 {
        valueStrings := make([]string, 0, len(req.Items))
        valueArgs := make([]interface{}, 0, len(req.Items)*4)
        
        for i, item := range req.Items {
            valueStrings = append(valueStrings, fmt.Sprintf("($%d, $%d, $%d, $%d)", 
                i*4+1, i*4+2, i*4+3, i*4+4))
            valueArgs = append(valueArgs, orderID, item.ProductID, item.Quantity, item.Price)
        }

        stmt := fmt.Sprintf(`
            INSERT INTO order_items (order_id, product_id, quantity, price)
            VALUES %s
        `, strings.Join(valueStrings, ","))

        _, err = tx.Exec(stmt, valueArgs...)
        if err != nil {
            return // Rollback automatically called
        }
    }

    err = tx.Commit()
    // ... handle success
}
```

### Database Connection Optimization
```go
// ✅ CORRECT: Optimized connection pool
func SetupDatabase() *sql.DB {
    db, err := sql.Open("postgres", dsn)
    if err != nil {
        log.Fatal("Failed to connect to database:", err)
    }

    // Performance tuning for POS workload
    db.SetMaxOpenConns(25)        // Limit concurrent connections
    db.SetMaxIdleConns(10)        // Keep connections alive
    db.SetConnMaxLifetime(5 * time.Minute) // Rotate connections
    db.SetConnMaxIdleTime(2 * time.Minute) // Close idle connections

    return db
}

// ✅ CORRECT: Connection health monitoring
func (h *Handler) healthCheck(c *gin.Context) {
    ctx, cancel := context.WithTimeout(c.Request.Context(), 2*time.Second)
    defer cancel()

    if err := h.db.PingContext(ctx); err != nil {
        c.JSON(http.StatusServiceUnavailable, gin.H{
            "status": "unhealthy",
            "database": "disconnected",
        })
        return
    }

    c.JSON(http.StatusOK, gin.H{
        "status": "healthy",
        "database": "connected",
    })
}
```

### Query Optimization Patterns
```sql
-- ✅ CORRECT: Strategic indexes for POS workload
-- Covering index for order listing (includes all needed columns)
CREATE INDEX CONCURRENTLY idx_orders_status_created_covering 
ON orders (status, created_at DESC) 
INCLUDE (id, order_number, total_amount, customer_name);

-- Partial index for active orders only
CREATE INDEX CONCURRENTLY idx_orders_active 
ON orders (created_at DESC) 
WHERE status IN ('pending', 'confirmed', 'preparing', 'ready');

-- Composite index for order items lookup
CREATE INDEX CONCURRENTLY idx_order_items_order_product 
ON order_items (order_id, product_id);


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/poinf-of-sales](https://github.com/madebyaris/poinf-of-sales) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
