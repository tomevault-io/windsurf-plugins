---
trigger: always_on
description: User journey optimization patterns for all POS roles with performance, UX, and business outcome focus
---


# 👥 User Journey Optimization & Role-Specific Patterns

## 🎯 Journey-First Design Philosophy

### Performance Targets by Role
```typescript
interface RolePerformanceTargets {
  admin: {
    dashboardLoad: '< 2 seconds',
    reportGeneration: '< 5 seconds',
    userManagement: '< 1 second per action',
    systemOverview: '< 1.5 seconds'
  },
  server: {
    orderCreation: '< 30 seconds total',
    productSelection: '< 5 seconds per item',
    tableAssignment: '< 3 seconds',
    customerInteraction: 'seamless, no delays'
  },
  counter: {
    paymentProcessing: '< 10 seconds',
    orderTypeSwitch: '< 2 seconds',
    receiptGeneration: '< 3 seconds',
    queueManagement: 'real-time updates'
  },
  kitchen: {
    statusUpdates: '< 1 second',
    orderPrioritization: 'real-time',
    workflowOptimization: 'continuous',
    communicationDelay: '< 2 seconds'
  }
}
```

## 👑 Admin Journey Optimization

### 1. Executive Dashboard Experience
```typescript
// ✅ ADMIN-OPTIMIZED: Executive dashboard with business intelligence
class AdminDashboardOptimization {
  // Intelligent data aggregation for C-level insights
  async loadExecutiveDashboard(): Promise<ExecutiveDashboard> {
    // Parallel data loading for instant insights
    const [
      realtimeMetrics,
      financialSummary,
      operationalHealth,
      staffPerformance,
      customerSatisfaction,
      systemAlerts
    ] = await Promise.all([
      this.getRealtimeBusinessMetrics(), // Revenue, orders/hour, avg ticket
      this.getFinancialSummary(), // Daily/weekly/monthly trends
      this.getOperationalHealth(), // Kitchen efficiency, table turnover
      this.getStaffPerformance(), // Individual and team metrics
      this.getCustomerSatisfaction(), // Wait times, order accuracy
      this.getSystemAlerts() // Technical and business alerts
    ])

    // Business intelligence: Automatic insights generation
    const insights = this.generateBusinessInsights({
      metrics: realtimeMetrics,
      trends: financialSummary,
      operations: operationalHealth
    })

    return {
      kpis: this.createKPIDashboard(realtimeMetrics),
      trends: this.createTrendAnalysis(financialSummary),
      alerts: this.prioritizeAlerts(systemAlerts),
      recommendations: insights.recommendations,
      quickActions: this.generateQuickActions(insights)
    }
  }

  // Predictive business insights
  private generateBusinessInsights(data: DashboardData): BusinessInsights {
    const insights: BusinessInsight[] = []

    // Revenue optimization insights
    if (data.metrics.averageTicket < data.historical.averageTicket * 0.95) {
      insights.push({
        type: 'revenue_optimization',
        severity: 'medium',
        title: 'Average Ticket Size Declining',
        description: 'Consider implementing upselling strategies or menu optimization',
        actionable: true,
        quickActions: [
          { label: 'View Menu Performance', action: 'navigate_to_menu_analytics' },
          { label: 'Staff Upselling Training', action: 'create_training_task' }
        ]
      })
    }

    // Operational efficiency insights
    if (data.operations.kitchenEfficiency < 0.85) {
      insights.push({
        type: 'operational_efficiency',
        severity: 'high',
        title: 'Kitchen Efficiency Below Target',
        description: 'Kitchen preparation times are impacting customer satisfaction',
        actionable: true,
        quickActions: [
          { label: 'View Kitchen Analytics', action: 'navigate_to_kitchen_dashboard' },
          { label: 'Optimize Kitchen Workflow', action: 'open_workflow_optimizer' }
        ]
      })
    }

    return {
      insights,
      recommendations: this.generateActionableRecommendations(insights),
      predictedImpact: this.calculatePredictedBusinessImpact(insights)
    }
  }
}

// Admin interface switching optimization
class AdminInterfaceSwitching {
  // Seamless role interface switching with context preservation
  async switchToRoleInterface(targetRole: UserRole, preserveContext: boolean = true): Promise<void> {
    // Pre-load target interface data
    const targetData = await this.preloadRoleData(targetRole)
    
    if (preserveContext) {
      // Preserve admin context for quick return
      this.preserveAdminContext({
        currentDashboard: this.getCurrentDashboardState(),
        activeReports: this.getActiveReports(),
        notifications: this.getPendingNotifications()
      })
    }

    // Optimized transition with loading states
    this.showTransitionLoading(`Switching to ${targetRole} interface...`)
    
    // Load role-specific optimizations
    const roleOptimizations = await this.loadRoleOptimizations(targetRole)
    
    // Smooth transition with preserved user experience
    this.transitionToRoleInterface(targetRole, targetData, roleOptimizations)
  }

  // Role-specific data preloading
  private async preloadRoleData(role: UserRole): Promise<RoleData> {
    const preloadStrategies = {
      server: () => Promise.all([
        this.menuService.getAvailableProducts(),
        this.tableService.getAvailableTables(),
        this.orderService.getActiveOrders()
      ]),
      counter: () => Promise.all([
        this.orderService.getPendingPayments(),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/poinf-of-sales](https://github.com/madebyaris/poinf-of-sales) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
