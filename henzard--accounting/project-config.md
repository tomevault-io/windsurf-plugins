---
trigger: always_on
description: **⚠️ CRITICAL**: React Native's `Alert.alert()` does NOT work on web. It silently fails.
---

---
globs:
  - "src/**/*.web.*"
  - "**/*.web.tsx"
alwaysApply: false
---

# Web-Specific Rules

## Alert Compatibility

**⚠️ CRITICAL**: React Native's `Alert.alert()` does NOT work on web. It silently fails.

**Solution**: Always use `showAlert` or `showConfirm` from `@/shared/utils/alert` instead of `Alert.alert` directly.

See [Rule 35: Web Alert Compatibility](35-web-alert-compatibility.mdc) for complete details.

---
 & Best Practices

## Responsive Design

### Mobile-First Approach
```typescript
// src/shared/constants/breakpoints.ts
export const BREAKPOINTS = {
  mobile: 0,       // 0-767px
  tablet: 768,     // 768-1023px
  desktop: 1024,   // 1024-1439px
  wide: 1440,      // 1440px+
} as const;

export function useMediaQuery() {
  const [breakpoint, setBreakpoint] = useState(getBreakpoint());

  useEffect(() => {
    const handleResize = () => setBreakpoint(getBreakpoint());
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return {
    breakpoint,
    isMobile: breakpoint === 'mobile',
    isTablet: breakpoint === 'tablet',
    isDesktop: breakpoint === 'desktop' || breakpoint === 'wide',
  };
}

function getBreakpoint() {
  const width = window.innerWidth;
  if (width >= BREAKPOINTS.wide) return 'wide';
  if (width >= BREAKPOINTS.desktop) return 'desktop';
  if (width >= BREAKPOINTS.tablet) return 'tablet';
  return 'mobile';
}
```

### Responsive Components
```typescript
export const AnimalGrid: React.FC = () => {
  const { breakpoint } = useMediaQuery();
  
  const columns = {
    mobile: 1,
    tablet: 2,
    desktop: 3,
    wide: 4,
  }[breakpoint];

  return (
    <FlatList
      data={animals}
      key={columns} // Force re-render on column change
      numColumns={columns}
      renderItem={({ item }) => <AnimalCard animal={item} />}
    />
  );
};
```

### Fluid Typography
```typescript
// Use viewport units for responsive text
const styles = StyleSheet.create({
  heading: {
    fontSize: 'clamp(24px, 5vw, 48px)', // Min 24px, max 48px
  },
});
```

## SEO Optimization

### Meta Tags
```typescript
// src/shared/utils/seo.web.ts
export interface PageMeta {
  title: string;
  description: string;
  keywords?: string[];
  ogImage?: string;
  canonical?: string;
}

export function setPageMeta(meta: PageMeta): void {
  // Title
  document.title = `${meta.title} | Weighsoft Animal Weigher`;
  
  // Description
  updateMetaTag('description', meta.description);
  
  // Keywords
  if (meta.keywords) {
    updateMetaTag('keywords', meta.keywords.join(', '));
  }
  
  // Open Graph
  updateMetaTag('og:title', meta.title, 'property');
  updateMetaTag('og:description', meta.description, 'property');
  if (meta.ogImage) {
    updateMetaTag('og:image', meta.ogImage, 'property');
  }
  
  // Canonical URL
  if (meta.canonical) {
    updateLinkTag('canonical', meta.canonical);
  }
}

function updateMetaTag(name: string, content: string, attr = 'name') {
  let tag = document.querySelector(`meta[${attr}="${name}"]`);
  if (!tag) {
    tag = document.createElement('meta');
    tag.setAttribute(attr, name);
    document.head.appendChild(tag);
  }
  tag.setAttribute('content', content);
}
```

### Semantic HTML
```typescript
// Use semantic HTML elements when targeting web
export const AnimalCard: React.FC = ({ animal }) => {
  if (Platform.OS === 'web') {
    return (
      <article>
        <header>
          <h2>{animal.name}</h2>
        </header>
        <section>
          <p>{animal.species}</p>
        </section>
      </article>
    );
  }
  
  // Mobile version
  return (
    <View>
      <Text>{animal.name}</Text>
      <Text>{animal.species}</Text>
    </View>
  );
};
```

## Web Routing

### React Router Setup
```typescript
// src/presentation/navigation/web-router.tsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';

export const WebRouter: React.FC = () => (
  <BrowserRouter>
    <Routes>
      {/* Public routes */}
      <Route path="/" element={<HomeScreen />} />
      <Route path="/login" element={<LoginScreen />} />
      
      {/* Protected routes */}
      <Route element={<ProtectedRoute />}>
        <Route path="/animals" element={<AnimalListScreen />} />
        <Route path="/animals/:id" element={<AnimalDetailScreen />} />
        <Route path="/weighing" element={<WeighingScreen />} />
        <Route path="/settings" element={<SettingsScreen />} />
      </Route>
      
      {/* 404 */}
      <Route path="*" element={<NotFoundScreen />} />
    </Routes>
  </BrowserRouter>
);

// Protected route wrapper
const ProtectedRoute = () => {
  const { isAuthenticated } = useAuth();
  return isAuthenticated ? <Outlet /> : <Navigate to="/login" />;
};
```

### URL Parameters & Query Strings
```typescript
// Using react-router hooks
import { useParams, useSearchParams } from 'react-router-dom';

export const AnimalDetailScreen = () => {
  const { id } = useParams<{ id: string }>();
  const [searchParams, setSearchParams] = useSearchParams();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/henzard) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
