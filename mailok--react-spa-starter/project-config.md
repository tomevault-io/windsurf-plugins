---
trigger: always_on
description: **This project uses React Router v7 in DATA MODE for SPA development, NOT Framework Mode.**
---

# React Router v7 Data Mode (SPA) - Cursor Rules

## 🚨 CRITICAL: Data Mode vs Framework Mode

**This project uses React Router v7 in DATA MODE for SPA development, NOT Framework Mode.**

- ✅ **Data Mode**: Uses `createBrowserRouter` with manual route configuration  
- ❌ **Framework Mode**: Uses `app/routes.ts` with auto-generated types

## Critical Package Guidelines

### ✅ CORRECT Packages for Data Mode:
- `react-router` - Main package for routing components, hooks, and router creation

### ❌ NEVER Use in Data Mode:
- `@react-router/dev` - Only for Framework Mode
- `@react-router/node` - Only for server-side Framework Mode  
- `@react-router/serve` - Only for Framework Mode
- `react-router-dom` - Legacy package, use `react-router` instead
- `@remix-run/*` - Old packages, replaced by `@react-router/*`

## Essential Data Mode Architecture

### Router Configuration Pattern
```tsx
import { createBrowserRouter, Navigate } from "react-router";
import { Root } from "./root";
import { DashboardPage } from "./dashboard/dashboard.page";

export const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    children: [
      {
        index: true,
        element: <DashboardPage />,
        loader: dashboardLoader,
        handle: {
          crumb: <DashboardBreadcrumb />,
        },
      },
      {
        path: "products/:id",
        element: <ProductPage />,
        loader: productLoader,
        action: productAction,
      },
      {
        path: "products",
        element: <ProductLayout />,
        children: [
          {
            index: true,
            element: <Navigate to="list" replace />,
          },
          {
            path: "list",
            element: <ProductList />,
            loader: productListLoader,
          },
        ],
      },
    ],
  },
  {
    path: "*",
    element: <NotFound />,
  },
]);
```

### Route Module Pattern (Data Mode)
```tsx
import { LoaderFunction, ActionFunction, redirect } from "react-router";
import { useLoaderData } from "react-router";

// Type-safe loader
export const productLoader: LoaderFunction = async ({ params }) => {
  const product = await getProduct(params.id!);
  if (!product) {
    throw new Response("Product Not Found", { status: 404 });
  }
  return { product };
};

// Type-safe action
export const productAction: ActionFunction = async ({ request, params }) => {
  const formData = await request.formData();
  await updateProduct(params.id!, formData);
  return redirect(`/products/${params.id}`);
};

// Component with typed loader data
interface ProductLoaderData {
  product: Product;
}

export default function ProductPage() {
  const { product } = useLoaderData() as ProductLoaderData;
  
  return (
    <div>
      <h1>{product.name}</h1>
      <Form method="post">
        <input name="name" defaultValue={product.name} />
        <button type="submit">Update</button>
      </Form>
    </div>
  );
}
```

### Layout Routes with Outlet
```tsx
import { Outlet, useNavigation } from "react-router";

export default function ProductLayout() {
  const navigation = useNavigation();
  
  return (
    <div className="layout">
      <nav>
        {/* Sidebar or navigation */}
      </nav>
      <main>
        {navigation.state === "loading" && <LoadingSpinner />}
        <Outlet /> {/* ✅ This renders the matching child route */}
      </main>
    </div>
  );
}
```

## Type Safety in Data Mode

### ✅ Manual Type Definitions:
In Data Mode, you define your own types for loader data and form actions:

```tsx
import { LoaderFunction, ActionFunction } from "react-router";

// Define your loader data types
interface DashboardLoaderData {
  user: User;
  stats: DashboardStats;
}

interface ProductLoaderData {
  product: Product;
  reviews: Review[];
}

// Type-safe loaders
export const dashboardLoader: LoaderFunction = async ({ request }) => {
  const user = await getCurrentUser(request);
  const stats = await getDashboardStats(user.id);
  return { user, stats } satisfies DashboardLoaderData;
};

// Type-safe component usage
export default function Dashboard() {
  const { user, stats } = useLoaderData() as DashboardLoaderData;
  
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <StatsDisplay stats={stats} />
    </div>
  );
}
```

### ✅ Form Action Types:
```tsx
interface ProductFormData {
  name: string;
  price: number;
  description: string;
}

export const productAction: ActionFunction = async ({ request, params }) => {
  const formData = await request.formData();
  
  const productData: ProductFormData = {
    name: formData.get("name") as string,
    price: Number(formData.get("price")),
    description: formData.get("description") as string,
  };
  
  await updateProduct(params.id!, productData);
  return redirect(`/products/${params.id}`);
};
```

## Critical Imports & Patterns

### ✅ Correct Imports for Data Mode:
```tsx
import { 
  createBrowserRouter, 
  RouterProvider,
  Link, 
  NavLink, 
  Form, 
  useLoaderData, 
  useActionData,
  useFetcher, 
  useNavigation,
  useNavigate,
  Outlet,
  Navigate,
  redirect,
  LoaderFunction,
  ActionFunction,
  type LoaderFunctionArgs,
  type ActionFunctionArgs,
} from "react-router";
```

### ❌ DON'T Import Framework Mode Features:
```tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mailok) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
