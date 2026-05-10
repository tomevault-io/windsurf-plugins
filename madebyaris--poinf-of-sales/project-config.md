---
trigger: always_on
description: Complete authentication, security, and debugging patterns for React + Go POS system
---


# 🔐 Authentication & Security Patterns

## 🚀 Essential Authentication Architecture

### JWT-Based Authentication Flow
```typescript
// Complete authentication workflow
class APIClient {
  constructor() {
    const apiUrl = import.meta.env?.VITE_API_URL || 'http://localhost:8080/api/v1';
    console.log('🔧 API Client baseURL:', apiUrl);
    
    this.client = axios.create({
      baseURL: apiUrl,
      timeout: 30000,
      headers: { 'Content-Type': 'application/json' }
    });

    // Auto-attach token from localStorage
    this.loadStoredAuth();
  }

  private loadStoredAuth(): void {
    const token = localStorage.getItem('pos_token');
    if (token) {
      this.client.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    }
  }

  setAuthToken(token: string): void {
    localStorage.setItem('pos_token', token);
    this.client.defaults.headers.common['Authorization'] = `Bearer ${token}`;
  }

  clearAuth(): void {
    localStorage.removeItem('pos_token');
    localStorage.removeItem('pos_user');
    delete this.client.defaults.headers.common['Authorization'];
  }

  isAuthenticated(): boolean {
    return !!localStorage.getItem('pos_token');
  }
}
```

## 🏗️ React Authentication Components

### Protected Route Pattern (Avoid Infinite Redirects)
```typescript
function HomePage() {
  // ✅ ALL HOOKS AT TOP LEVEL - NEVER after returns
  const [user, setUser] = useState<User | null>(null);
  const [isLoadingAuth, setIsLoadingAuth] = useState(true); // Critical: Start true

  const { isLoading: isVerifying, error } = useQuery({
    queryKey: ['currentUser'],
    queryFn: () => apiClient.getCurrentUser(),
    enabled: false, // Control when to verify
    retry: 1,
  });

  // Load auth state from localStorage FIRST
  useEffect(() => {
    const loadAuthState = async () => {
      const token = localStorage.getItem('pos_token');
      const storedUser = localStorage.getItem('pos_user');
      
      console.log('🔍 Loading auth - token:', token ? 'exists' : 'missing');
      console.log('🔍 Loading auth - user:', storedUser ? 'exists' : 'missing');
      
      if (storedUser && token) {
        try {
          const parsedUser = JSON.parse(storedUser);
          setUser(parsedUser);
          console.log('✅ Auth loaded - user role:', parsedUser.role);
        } catch (error) {
          console.error('❌ Invalid stored auth data, clearing');
          apiClient.clearAuth();
        }
      }
      
      setIsLoadingAuth(false);
    };
    
    loadAuthState();
  }, []);

  // ✅ CRITICAL: Wait for localStorage loading before auth checks
  if (isLoadingAuth) {
    return <LoadingSpinner message="Loading authentication..." />;
  }

  // Only check auth AFTER loading is complete
  if (!apiClient.isAuthenticated() || !user) {
    console.log('🔄 Not authenticated, redirecting to login');
    return <Navigate to="/login" replace />;
  }

  // Render protected content with user context
  return <RoleBasedLayout user={user} />;
}
```

### Login Component Pattern
```typescript
function LoginPage() {
  const [error, setError] = useState<string | null>(null);
  
  // Redirect if already authenticated
  if (apiClient.isAuthenticated()) {
    return <Navigate to="/" replace />;
  }

  const loginMutation = useMutation({
    mutationFn: async (credentials: LoginRequest) => {
      console.log('🔄 Attempting login...');
      return await apiClient.login(credentials);
    },
    onSuccess: (data) => {
      console.log('✅ Login success:', data.success);
      
      if (data.success && data.data) {
        // Set auth token first
        apiClient.setAuthToken(data.data.token);
        
        // Store user data
        localStorage.setItem('pos_user', JSON.stringify(data.data.user));
        
        console.log('✅ Auth stored - role:', data.data.user.role);
        
        // Brief delay prevents race conditions
        setTimeout(() => {
          router.navigate({ to: '/' });
        }, 100);
      }
    },
    onError: (error: any) => {
      console.error('❌ Login failed:', error.message);
      setError(error.message || 'Login failed');
    },
  });

  return (
    <LoginForm 
      onSubmit={loginMutation.mutate}
      isLoading={loginMutation.isPending}
      error={error}
    />
  );
}
```

## 🛡️ Backend Security Patterns

### Go JWT Middleware
```go
// JWT authentication middleware
func AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        tokenString := c.GetHeader("Authorization")
        if tokenString == "" {
            c.JSON(http.StatusUnauthorized, models.APIResponse{
                Success: false,
                Message: "Authorization header required",
                Error:   stringPtr("missing_auth_header"),
            })
            c.Abort()
            return
        }

        // Remove "Bearer " prefix
        if len(tokenString) > 7 && tokenString[:7] == "Bearer " {
            tokenString = tokenString[7:]
        }

        token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
            if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
                return nil, fmt.Errorf("unexpected signing method")
            }
            return []byte(os.Getenv("JWT_SECRET")), nil
        })


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/poinf-of-sales](https://github.com/madebyaris/poinf-of-sales) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
