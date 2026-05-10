---
trigger: always_on
description: Complete React component patterns, hooks, and state management with shadcn/ui for POS System
---


# ⚛️ React Component Patterns & Best Practices

## 🏗️ Component Architecture

### shadcn/ui Base Components Pattern
```typescript
// Follow the established shadcn/ui pattern from components/ui/
import * as React from 'react'
import { cn } from '@/lib/utils'
import { cva, type VariantProps } from 'class-variance-authority'

// ✅ CORRECT: Use cva for variant-based styling
const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md font-medium transition-colors",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        outline: "border border-input bg-background hover:bg-accent",
        ghost: "hover:bg-accent hover:text-accent-foreground",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 px-3",
        lg: "h-11 px-8",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean
}

// ✅ CORRECT: Always use forwardRef for reusable components
const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    return (
      <button
        ref={ref}
        className={cn(buttonVariants({ variant, size, className }))}
        {...props}
      />
    )
  }
)
Button.displayName = "Button"
```

### Business Component Pattern
```typescript
// POS-specific business components
import { useState, useCallback } from 'react'
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { Badge } from '@/components/ui/badge'
import apiClient from '@/api/client'
import type { Product, CartItem } from '@/types'

interface ProductCardProps {
  product: Product
  onSelect: (product: Product) => void
  isSelected?: boolean
  isInCart?: boolean
  cartQuantity?: number
  className?: string
}

// ✅ CORRECT: Typed component with proper props interface
export const ProductCard: React.FC<ProductCardProps> = ({
  product,
  onSelect,
  isSelected = false,
  isInCart = false,
  cartQuantity = 0,
  className
}) => {
  // ✅ CORRECT: Use useCallback for event handlers
  const handleSelect = useCallback(() => {
    onSelect(product)
  }, [product, onSelect])

  // ✅ CORRECT: Early returns for loading/error states
  if (!product.is_available) {
    return (
      <Card className={cn("opacity-50", className)}>
        <CardContent className="p-4 text-center">
          <p className="text-muted-foreground">Unavailable</p>
        </CardContent>
      </Card>
    )
  }

  return (
    <Card 
      className={cn(
        "cursor-pointer transition-all hover:shadow-md",
        isSelected && "ring-2 ring-primary",
        className
      )}
      onClick={handleSelect}
    >
      <CardHeader className="pb-2">
        <div className="flex items-center justify-between">
          <CardTitle className="text-lg">{product.name}</CardTitle>
          {isInCart && (
            <Badge variant="secondary">{cartQuantity}</Badge>
          )}
        </div>
      </CardHeader>
      <CardContent>
        <p className="text-sm text-muted-foreground mb-2">
          {product.description}
        </p>
        <p className="text-xl font-bold">${product.price.toFixed(2)}</p>
      </CardContent>
    </Card>
  )
}
```

## 🎣 Custom Hooks Patterns

### Data Fetching Hook
```typescript
// Custom hook for API data fetching
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { toastHelpers } from '@/lib/toast-helpers'
import apiClient from '@/api/client'
import type { Product, CreateProductRequest } from '@/types'

export const useProducts = (categoryId?: string) => {
  return useQuery({
    queryKey: ['products', categoryId],
    queryFn: () => apiClient.getProducts({ category_id: categoryId }),
    select: (data) => data.data || [], // Transform response
    staleTime: 5 * 60 * 1000, // 5 minutes
  })
}

export const useCreateProduct = () => {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (product: CreateProductRequest) => 
      apiClient.createProduct(product),
    onSuccess: (data) => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['products'] })
      toastHelpers.success('Product created successfully!')
    },
    onError: (error: any) => {
      toastHelpers.error(`Failed to create product: ${error.message}`)
    },
  })
}
```

### Business Logic Hook (Shopping Cart)
```typescript
// Custom hook for cart management
import { useState, useCallback, useMemo } from 'react'
import type { Product, CartItem } from '@/types'

export const useCart = () => {
  const [items, setItems] = useState<CartItem[]>([])

  // ✅ CORRECT: Memoized calculations
  const total = useMemo(() => {
    return items.reduce((sum, item) => sum + (item.product.price * item.quantity), 0)
  }, [items])

  const itemCount = useMemo(() => {
    return items.reduce((count, item) => count + item.quantity, 0)
  }, [items])


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/poinf-of-sales](https://github.com/madebyaris/poinf-of-sales) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
