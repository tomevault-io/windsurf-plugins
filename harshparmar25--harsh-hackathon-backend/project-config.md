---
trigger: always_on
description: This guide details the implementation patterns for data transformation between different layers of the system.
---

# Data Transformation Guide

This guide details the implementation patterns for data transformation between different layers of the system.

## Core Concepts

Mappers handle data transformation between layers:

1. External data to domain entities
2. Domain entities to DTOs
3. Domain entities to infrastructure format
4. Isolate transformation logic
5. Keep domain entities clean
6. Error handling and validation
7. Type safety and consistency
8. Performance optimization
9. Caching support
10. Logging and monitoring

## Directory Structure

```
mappers/
├── orderMapper.ts
├── orderMapper.test.ts
├── paymentMapper.ts
├── paymentMapper.test.ts
├── customerMapper.ts
├── customerMapper.test.ts
├── cache/
│   ├── orderCache.ts
│   └── paymentCache.ts
└── utils/
    ├── validation.ts
    └── formatting.ts
```

## Mapper Best Practices

### Interface Guidelines

1. **Clear Contracts**: Define explicit input/output types
2. **Single Responsibility**: Each mapper handles one entity type
3. **Error Handling**: Define error types and handling strategies
4. **Type Safety**: Use proper types and interfaces
5. **Documentation**: Document all methods and parameters
6. **Testing**: Include test interfaces
7. **Validation**: Validate input data
8. **Caching**: Support caching strategies

### Implementation Guidelines

1. **Error Handling**: Implement proper error handling
2. **Logging**: Log important operations and errors
3. **Monitoring**: Monitor mapper performance
4. **Validation**: Validate all inputs
5. **Caching**: Implement caching strategies
6. **Performance**: Optimize transformation operations
7. **Testing**: Write comprehensive tests
8. **Documentation**: Document implementation details

### Performance Guidelines

1. **Caching**: Use caching for expensive transformations
2. **Batching**: Support batch transformations
3. **Lazy Loading**: Implement lazy loading where appropriate
4. **Memory Management**: Optimize memory usage
5. **Async Operations**: Support async transformations
6. **Streaming**: Support streaming for large datasets
7. **Validation**: Optimize validation logic
8. **Monitoring**: Monitor performance metrics

## Implementation Examples

### Basic Mapper

```typescript
// mappers/orderMapper.ts

import { Order } from '../domain/entities/order';
import { OrderDto } from '../dtos/orderDto';
import { Logger } from '../utils/logger';
import { CacheService } from '../services/cacheService';
import { MonitoringService } from '../services/monitoringService';
import { ValidationError } from '../utils/errors';
import { z } from 'zod';

const orderSchema = z.object({
  id: z.string(),
  customerId: z.string(),
  items: z.array(z.object({
    productId: z.string(),
    price: z.number().positive(),
    quantity: z.number().int().positive()
  })),
  status: z.enum(['pending', 'processing', 'shipped', 'delivered', 'cancelled']),
  total: z.number().nonnegative(),
  createdAt: z.date()
});

export class OrderMapper {
  private static readonly logger = new Logger('OrderMapper');
  private static readonly cacheService: CacheService;
  private static readonly monitoringService: MonitoringService;

  static async toDomain(raw: any): Promise<Order> {
    try {
      this.logger.info('Mapping raw data to Order domain entity');
      this.monitoringService.incrementCounter('order_mapping_to_domain');

      const cachedOrder = await this.cacheService.get<Order>(`order:${raw.id}`);
      if (cachedOrder) {
        return cachedOrder;
      }

      const orderProps = {
        id: raw.id || raw._id,
        customerId: raw.customer_id || raw.customerId,
        items: raw.items.map((item: any) => ({
          productId: item.product_id || item.productId,
          price: item.price,
          quantity: item.quantity
        })),
        status: raw.status || 'pending',
        total: raw.total || 0,
        createdAt: raw.created_at ? new Date(raw.created_at) : new Date()
      };

      orderSchema.parse(orderProps);
      
      const order = Order.create(orderProps);
      await this.cacheService.set(`order:${order.id}`, order, 3600); // Cache for 1 hour
      
      return order;
    } catch (error) {
      this.logger.error('Failed to map raw data to Order domain entity', { error });
      this.monitoringService.incrementCounter('order_mapping_to_domain_error');
      throw new ValidationError('Invalid order data', error);
    }
  }
  
  static async toDto(order: Order): Promise<OrderDto> {
    try {
      this.logger.info('Mapping Order domain entity to DTO');
      this.monitoringService.incrementCounter('order_mapping_to_dto');

      const cachedDto = await this.cacheService.get<OrderDto>(`order_dto:${order.id}`);
      if (cachedDto) {
        return cachedDto;
      }

      const orderDetails = order.toObject();
      
      const dto = {
        id: orderDetails.id,
        customerId: orderDetails.customerId,
        items: orderDetails.items.map(item => ({
          productId: item.productId,
          price: this.formatPrice(item.price),
          quantity: item.quantity,
          subtotal: this.formatPrice(item.price * item.quantity)
        })),
        status: this.mapStatus(orderDetails.status),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HarshParmar25) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
