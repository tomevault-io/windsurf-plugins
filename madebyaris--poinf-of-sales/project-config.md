---
trigger: always_on
description: Comprehensive business logic patterns for POS System domain understanding, user journeys, and workflow optimization
---


# 🍽️ POS Business Logic & Domain Patterns

## 🎯 Core Business Domain Understanding

### Restaurant Operations Model
The POS system orchestrates complex restaurant operations with multiple stakeholders and intricate workflows:

```typescript
// Domain Model - Core Business Entities
interface RestaurantDomain {
  // Revenue Generation
  orders: OrderLifecycle[]
  payments: PaymentProcessing[]
  inventory: InventoryManagement
  
  // Operations Management  
  tables: TableManagement
  staff: StaffOperations
  kitchen: KitchenWorkflow
  
  // Business Intelligence
  analytics: BusinessAnalytics
  reporting: FinancialReporting
}

// Business Rules Engine
class POSBusinessRules {
  validateOrderCreation(order: CreateOrderRequest): ValidationResult
  calculatePricing(items: OrderItem[]): PricingCalculation
  manageInventory(productId: string, quantity: number): InventoryResult
  optimizeKitchenWorkflow(orders: Order[]): WorkflowOptimization
}
```

## 🔄 Critical User Journeys & Performance Optimization

### 1. Server Journey: Dine-In Order Creation (Target: <30 seconds)
```typescript
// ✅ PERFORMANCE-OPTIMIZED: Server workflow
class ServerWorkflowOptimization {
  // Pre-load critical data for instant access
  private async preloadServerData(): Promise<ServerContext> {
    const [products, categories, tables, activeOrders] = await Promise.all([
      this.productService.getAvailableProducts(), // Cache for 5 minutes
      this.categoryService.getActiveCategories(), // Cache for 1 hour  
      this.tableService.getTableStatus(), // Real-time, 30s cache
      this.orderService.getActiveOrders() // Real-time, 10s cache
    ])
    
    return { products, categories, tables, activeOrders }
  }

  // Optimistic order creation with rollback
  async createOrderOptimistic(orderData: CreateOrderRequest): Promise<Order> {
    // 1. Immediate UI feedback (0ms)
    this.ui.showOrderCreating(orderData)
    
    // 2. Validate business rules locally (5-10ms)
    const validation = await this.validateOrderBusiness(orderData)
    if (!validation.isValid) {
      throw new BusinessRuleError(validation.errors)
    }
    
    // 3. Optimistic update (10-15ms)
    const optimisticOrder = this.generateOptimisticOrder(orderData)
    this.ui.showOrderCreated(optimisticOrder)
    
    // 4. Background server sync (100-200ms)
    try {
      const serverOrder = await this.orderService.createOrder(orderData)
      this.reconcileOptimisticOrder(optimisticOrder, serverOrder)
      return serverOrder
    } catch (error) {
      // Rollback optimistic changes
      this.rollbackOptimisticOrder(optimisticOrder)
      throw error
    }
  }

  // Business rule validation (prevent API round-trips)
  private async validateOrderBusiness(order: CreateOrderRequest): Promise<ValidationResult> {
    const errors: string[] = []
    
    // Table availability check
    if (order.table_id && !this.isTableAvailable(order.table_id)) {
      errors.push('Table is not available')
    }
    
    // Product availability batch check
    const unavailableItems = order.items.filter(item => 
      !this.isProductAvailable(item.product_id, item.quantity)
    )
    if (unavailableItems.length > 0) {
      errors.push(`Products unavailable: ${unavailableItems.map(i => i.product_id).join(', ')}`)
    }
    
    // Business hours validation
    if (!this.isDuringBusinessHours()) {
      errors.push('Orders cannot be created outside business hours')
    }
    
    return { isValid: errors.length === 0, errors }
  }
}
```

### 2. Enhanced Kitchen Journey: As-Ready Service Workflow (Target: <3 seconds per item update)
```typescript
// ✅ REAL-TIME OPTIMIZED: Enhanced kitchen workflow with individual item tracking
class EnhancedKitchenWorkflowEngine {
  private orderPriorityQueue: PriorityQueue<KitchenOrder>
  private preparationTimers: Map<string, Timer>
  private realTimeUpdates: EventEmitter
  private soundNotificationSystem: SoundNotificationSystem

  // Intelligent order prioritization with as-ready service
  async optimizeKitchenQueue(): Promise<KitchenOrder[]> {
    const activeOrders = await this.getActiveKitchenOrders()
    
    // Business logic: Priority calculation with individual item tracking
    return activeOrders
      .map(order => ({
        ...order,
        priority: this.calculateOrderPriority(order),
        estimatedTime: this.estimatePreparationTime(order),
        dependencies: this.findOrderDependencies(order),
        itemProgress: this.calculateItemProgress(order.items), // New: Individual item tracking
        readyItems: order.items.filter(item => item.status === 'ready'),
        servedItems: order.items.filter(item => item.status === 'served')
      }))
      .sort((a, b) => {
        // Enhanced priority: wait time, complexity, table status, ready items
        const aScore = (a.priority * a.waitTime * a.tableUrgency) + (a.readyItems.length * 10)
        const bScore = (b.priority * b.waitTime * b.tableUrgency) + (b.readyItems.length * 10)
        return bScore - aScore
      })
  }

  // As-ready service: Individual item completion
  async markItemReady(orderId: string, itemId: string): Promise<void> {
    await this.updateItemStatus(orderId, itemId, 'ready')

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/poinf-of-sales](https://github.com/madebyaris/poinf-of-sales) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
