---
trigger: always_on
description: // Saleor Quickstart - Cursor AI Rules
---

// Saleor Quickstart - Cursor AI Rules
// This file provides instructions for Cursor AI to help with Saleor development

// Core technologies used in this project
const technologies = [
  "Next.js",
  "TypeScript",
  "GraphQL",
  "Saleor",
  "Tailwind CSS",
  "Apollo Client"
];

// Best practices for Saleor GraphQL development
const bestPractices = [
  "Use fragments for reusable parts of GraphQL queries",
  "Implement proper error handling and loading states for all GraphQL operations",
  "Follow the Configurator patterns for extending product types and attributes",
  "Use TypeScript types generated from the GraphQL schema",
  "Leverage Apollo Client's caching capabilities for performance",
  "Structure GraphQL operations into queries, mutations and fragments",
  "Implement dynamic form generation based on Saleor's attribute system",
  "Follow the multi-channel architecture pattern",
  "Handle checkout process according to Saleor's step-based approach",
  "Respect price freezing and stock allocation patterns for orders"
];

// Folder structure guide
const folderStructure = `
src/
  components/           # React components
    products/           # Product-related components
    checkout/           # Checkout flow components
    ui/                 # Shared UI components
  graphql/              # GraphQL operations
    fragments/          # Reusable query fragments
    queries/            # GraphQL queries
    mutations/          # GraphQL mutations
  hooks/                # Custom React hooks
  lib/                  # Utility functions
  pages/                # Next.js pages
  styles/               # Global styles
  providers/            # Context providers
  types/                # TypeScript types
`;

// Saleor Core Concepts
const saleorCoreConcepts = `
1. Multi-Channel Architecture
   - Channels represent sales channels (websites, mobile apps, POS systems)
   - Each channel has its own pricing, availability, and shipping options
   - Products can be available in multiple channels with different configurations

2. Pricing System
   - Supports gross and net pricing
   - Handles multiple currencies per channel
   - Implements price lists and discounts
   - Manages tax configurations and calculations

3. Order Management Flow
   - Order states: Unconfirmed → Unfulfilled → Partially Fulfilled → Fulfilled → Completed
   - Supports partial fulfillments and returns
   - Implements payment flow with transaction states
   - Handles refunds and cancellations

4. Meta Fields System
   - Allows storing additional data on most entities
   - Custom fields can be added to orders, users, products, etc.
   - Useful for integrations with external systems

5. Permission System
   - RBAC (Role-Based Access Control)
   - Staff users with assigned permissions
   - Token-based API authentication
   - Permission-based access to API operations

6. Transaction System
   - Supports asynchronous payment flow
   - Transaction states: Pending → Success/Failed
   - Allows for payment refunds and captures
   - Integration with external payment providers via Apps

7. App Framework
   - Extends core functionality with Apps
   - Apps can provide webhooks, payment methods, etc.
   - Apps authenticate with tokens
   - Custom dashboard extensions via Apps
`;

// Saleor-specific patterns
const saleorPatterns = `
1. Configurator-based Product Modeling
   - Define product types with appropriate attributes
   - Use variant attributes for options that create distinct variants
   - Use product attributes for features that don't create variants

2. Multi-Channel Setup
   - Always include channel in GraphQL operations
   - Handle channel-specific pricing and availability

3. Checkout Flow
   - Follow the step sequence: AddItems → ShippingAddress → ShippingMethod → Billing → Payment
   - Handle shipping and billing address validation
   - Implement proper error handling for checkout mutations

4. Dynamic Attribute Handling
   - Render form fields based on attribute input types
   - Use appropriate UI components for each attribute type
   - Generate variant combinations when needed

5. Authentication and Permissions
   - Use JWT tokens for authentication
   - Implement proper permission checks
`;

// GraphQL schema awareness - Extended with deeper schema examples
const schemaPatterns = `
1. Product and Variant Structure
   - Product is the parent entity with general information
   - ProductVariant represents sellable variants with unique SKUs
   - Products belong to ProductTypes which define their attributes
   - Example of fetching a product with variants:
   
   query ProductWithVariants($id: ID!, $channel: String!) {
     product(id: $id, channel: $channel) {
       id
       name
       slug
       description
       category { name, slug }
       collections { edges { node { name, slug } } }
       media { url, alt, type }
       variants {
         id
         name
         sku
         attributes {
           attribute { name, slug }
           values { name, slug }
         }
         pricing {
           price { gross { amount, currency } }
           onSale
           discount { gross { amount } }
         }
         quantityAvailable
       }
     }
   }

2. Order and Checkout Flow
   - Checkout is a temporary cart that becomes an Order

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saleor/quickstart](https://github.com/saleor/quickstart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
