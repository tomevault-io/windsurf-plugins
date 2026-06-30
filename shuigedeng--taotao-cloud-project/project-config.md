---
trigger: always_on
description: 本项目采用 **领域驱动设计（DDD）** 分层架构，严格遵循六边形架构思想。
---

# SpringBoot DDD 架构开发规范

## 项目架构概述

本项目采用 **领域驱动设计（DDD）** 分层架构，严格遵循六边形架构思想。

### 包结构规范（必须遵守）
com.company.product/
├── domain/ # 领域层（核心业务逻辑）
│ ├── model/ # 领域模型
│ │ ├── aggregate/ # 聚合根
│ │ │ └── order/
│ │ │ ├── Order.java # 聚合根
│ │ │ ├── OrderItem.java # 实体
│ │ │ └── OrderStatus.java # 值对象
│ │ ├── valueobject/ # 值对象
│ │ │ ├── Money.java
│ │ │ ├── Address.java
│ │ │ └── Email.java
│ │ └── event/ # 领域事件
│ │ ├── OrderCreatedEvent.java
│ │ └── OrderPaidEvent.java
│ ├── service/ # 领域服务
│ │ └── PricingService.java
│ ├── repository/ # 仓储接口
│ │ └── OrderRepository.java
│ ├── specification/ # 规格模式
│ │ └── OrderSpecification.java
│ └── factory/ # 工厂
│ └── OrderFactory.java
├── application/ # 应用层（用例编排）
│ ├── service/ # 应用服务
│ │ ├── OrderApplicationService.java
│ │ └── dto/ # 应用层DTO
│ │ ├── CreateOrderCommand.java
│ │ └── OrderDto.java
│ ├── port/ # 端口（入站/出站）
│ │ ├── inbound/ # 入站端口（API接口）
│ │ │ └── OrderUseCase.java
│ │ └── outbound/ # 出站端口（SPI接口）
│ │ ├── PaymentPort.java
│ │ └── NotificationPort.java
│ └── handler/ # 事件处理器
│ └── OrderEventHandler.java
├── infrastructure/ # 基础设施层
│ ├── repository/ # 仓储实现
│ │ ├── JpaOrderRepository.java
│ │ └── mapper/ # ORM映射
│ │ └── OrderMapper.java
│ ├── config/ # 配置
│ │ ├── BeanConfiguration.java
│ │ └── JpaConfiguration.java
│ └── adapter/ # 适配器
│ ├── payment/ # 支付适配器
│ │ └── AlipayAdapter.java
│ └── notification/ # 通知适配器
│ └── SmsAdapter.java
└── interfaces/ # 接口层
├── rest/ # REST API
│ ├── OrderController.java
│ └── dto/ # REST DTO
│ ├── OrderRequest.java
│ └── OrderResponse.java
└── message/ # 消息监听
└── OrderMessageListener.java

text

## DDD 核心原则

### 1. 聚合设计原则
- **原子性**: 聚合内保证事务一致性，聚合间使用最终一致性
- **小聚合**: 一个聚合根只包含必要实体，避免性能问题
- **ID引用**: 跨聚合通过ID引用，而非对象引用
- **不变性**: 聚合根负责维护内部不变量

```java
// 聚合根示例
@AggregateRoot
@Entity
@Table(name = "orders")
public class Order {
    @Id
    private OrderId id;  // 值对象作为ID
    
    @Embedded
    private CustomerId customerId;  // 跨聚合引用
    
    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
    private List<OrderItem> items = new ArrayList<>();
    
    @Embedded
    private Money totalAmount;
    
    private OrderStatus status;
    
    // 聚合根行为方法
    public void addItem(ProductId productId, Money price, int quantity) {
        // 业务规则校验
        if (status != OrderStatus.PENDING) {
            throw new DomainException("只能向待支付订单添加商品");
        }
        
        OrderItem item = new OrderItem(productId, price, quantity);
        items.add(item);
        recalculateTotal();
        
        // 注册领域事件
        registerEvent(new OrderItemAddedEvent(id, productId, quantity));
    }
    
    public void pay() {
        if (items.isEmpty()) {
            throw new DomainException("空订单不能支付");
        }
        this.status = OrderStatus.PAID;
        registerEvent(new OrderPaidEvent(id, totalAmount));
    }
    
    private void recalculateTotal() {
        this.totalAmount = items.stream()
            .map(OrderItem::getSubtotal)
            .reduce(Money.ZERO, Money::add);
    }
    
    // 无参构造器（JPA要求）
    protected Order() {}
    
    // 工厂方法
    public static Order create(CustomerId customerId, Address shippingAddress) {
        Order order = new Order();
        order.id = new OrderId(IdGenerator.next());
        order.customerId = customerId;
        order.status = OrderStatus.PENDING;
        order.registerEvent(new OrderCreatedEvent(order.id));
        return order;
    }
}
2. 值对象设计
不可变性: 所有字段final，无setter方法

无标识: 通过属性值判断相等性

自验证: 构造时验证数据合法性

java
@ValueObject
@Embeddable
public class Money {
    private final BigDecimal amount;
    private final Currency currency;
    
    public Money(BigDecimal amount, Currency currency) {
        if (amount == null || amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new DomainException("金额不能为负数");
        }
        this.amount = amount;
        this.currency = currency;
    }
    
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new DomainException("货币类型不匹配");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }
    
    // 只有getter，无setter
    public BigDecimal getAmount() { return amount; }
    public Currency getCurrency() { return currency; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Money)) return false;
        Money money = (Money) o;
        return amount.compareTo(money.amount) == 0 &&
               currency.equals(money.currency);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(amount, currency);
    }
}
3. 领域服务
无状态: 不持有状态，只提供行为

跨聚合: 协调多个聚合的业务逻辑

业务价值: 不属于任何单一聚合

java
@DomainService
@Service
@Slf4j
public class PricingService {
    private final ProductRepository productRepository;
    private final DiscountPolicy discountPolicy;
    
    public PricingService(ProductRepository productRepository, 
                          DiscountPolicy discountPolicy) {
        this.productRepository = productRepository;
        this.discountPolicy = discountPolicy;
    }
    
    /**
     * 计算订单总价（跨聚合逻辑）
     */
    public Money calculateOrderPrice(Order order, CustomerType customerType) {
        Money subtotal = Money.ZERO;
        
        for (OrderItem item : order.getItems()) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shuigedeng/taotao-cloud-project](https://github.com/shuigedeng/taotao-cloud-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
