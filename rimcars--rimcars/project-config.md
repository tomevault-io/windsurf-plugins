---
trigger: always_on
description: Based on the centralized type system documented in [CODEBASE_ANALYSIS.md](mdc:CODEBASE_ANALYSIS.md), follow these TypeScript patterns:
---

# TypeScript Patterns & Best Practices

## 🔧 Type System Organization

Based on the centralized type system documented in [CODEBASE_ANALYSIS.md](mdc:CODEBASE_ANALYSIS.md), follow these TypeScript patterns:

### Central Type Exports
All types should be exported from `src/types/index.ts` for consistent imports across the codebase.

```typescript
// ✅ Always import from centralized types
import type { DatabaseUser, CarListing, ListingFormData } from "@/types";

// ❌ Don't import from scattered type files
import type { User } from "../features/auth/types/user";
```

### Database Type Integration
The project uses auto-generated Supabase types as the source of truth:

```typescript
// ✅ Current pattern - Use database schema types
import type { Tables } from "@/types/database.types";

export type DatabaseUser = Tables<'users'>;
export type DatabaseListing = Tables<'listings'>;

// ✅ For forms and extended types
export interface ListingFormData extends Omit<DatabaseListing, 'id' | 'created_at' | 'updated_at'> {
  images: File[]; // File objects for upload
}

export interface ExtendedListing extends DatabaseListing {
  seller?: DatabaseUser; // Joined data
}
```

## 🎯 Component Type Patterns

### Props Interface Definitions
```typescript
// ✅ Clear interface naming and optional props
interface UserMenuProps {
  user: DatabaseUser | null;
  className?: string;
  onSignOut?: () => void;
}

// ✅ Generic component props with constraints
interface CardProps<T = any> {
  data: T;
  onSelect?: (item: T) => void;
  className?: string;
}

// ✅ Event handler types
interface CarCardProps {
  car: DatabaseListing;
  onFavoriteToggle?: (carId: string) => void;
  onView?: (car: DatabaseListing) => void;
}
```

### Server Action Type Safety
```typescript
// ✅ Server action return types
interface ActionResult<T = any> {
  success: boolean;
  data?: T;
  error?: string;
}

// ✅ Server action definitions
export async function createListing(
  formData: ListingFormData
): Promise<ActionResult<DatabaseListing>> {
  try {
    // Implementation
    return { success: true, data: newListing };
  } catch (error) {
    return { success: false, error: "فشل في إنشاء الإعلان" };
  }
}

// ✅ Client-side usage with proper typing
const handleSubmit = async (data: ListingFormData) => {
  const result = await createListing(data);
  if (result.success && result.data) {
    // TypeScript knows result.data is DatabaseListing
    router.push(`/cars/${result.data.id}`);
  } else {
    toast.error(result.error);
  }
};
```

## 🔄 State Management Types

### Zustand Store Types
```typescript
// ✅ Favorites store typing (client-side state)
interface FavoritesState {
  favorites: string[];
  addFavorite: (carId: string) => void;
  removeFavorite: (carId: string) => void;
  isFavorite: (carId: string) => boolean;
  toggleFavorite: (carId: string) => void;
  clearFavorites: () => void;
}

export const useFavoritesStore = create<FavoritesState>()(
  persist(
    (set, get) => ({
      favorites: [],
      addFavorite: (carId) => {
        const { favorites } = get();
        if (!favorites.includes(carId)) {
          set({ favorites: [...favorites, carId] });
        }
      },
      // ... other methods
    }),
    { name: "car-favorites" }
  )
);
```

### Form Validation Types
```typescript
// ✅ Zod schema with TypeScript integration
import { z } from "zod";

export const createListingSchema = z.object({
  car_name: z.string().min(1, "اسم السيارة مطلوب"),
  description: z.string().min(10, "الوصف يجب أن يكون 10 أحرف على الأقل"),
  price: z.number().min(1, "السعر مطلوب"),
  make: z.string().min(1, "الماركة مطلوبة"),
  model: z.string().min(1, "الموديل مطلوب"),
  year: z.number().min(1990).max(new Date().getFullYear() + 1),
  mileage: z.number().min(0),
  condition: z.enum(["new", "used", "certified"]),
  transmission: z.enum(["manual", "automatic"]),
  fuel_type: z.enum(["gasoline", "diesel", "hybrid", "electric"]),
  location: z.string().min(1, "الموقع مطلوب"),
});

// ✅ Infer TypeScript type from Zod schema
export type CreateListingData = z.infer<typeof createListingSchema>;
```

## 🌐 API & Server Types

### Next.js App Router Types
```typescript
// ✅ Page component types
interface PageProps {
  params: { id: string };
  searchParams: { [key: string]: string | string[] | undefined };
}

export default async function CarDetailPage({ params }: PageProps) {
  const car = await getPublicListingById(params.id);
  // TypeScript knows car is DatabaseListing | null
}

// ✅ Layout component types
interface LayoutProps {
  children: React.ReactNode;
  params?: { [key: string]: string };
}

export default async function PublicLayout({ children }: LayoutProps) {
  const user = await getUserWithProfile();
  return (
    <>
      <Header user={user} />
      {children}
    </>
  );
}
```

### Server Action Parameter Types
```typescript
// ✅ Form action types with proper validation
export async function updateUserProfile(
  userId: string,
  data: {
    name?: string;
    phone_number?: string;
    avatar_url?: string;
  }
): Promise<ActionResult<DatabaseUser>> {
  // Implementation with type safety
}

// ✅ File upload types
export async function uploadCarImages(
  listingId: string,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rimcars) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
