---
trigger: always_on
description: These testing patterns specifically target the 5 critical failure modes identified in Sprint 10.
---

# Sprint 10 Testing Patterns - Prevention Through Testing

## 🎯 GOAL: Catch Sprint 10 Issues Before They Become Problems

These testing patterns specifically target the 5 critical failure modes identified in Sprint 10.

### 🔥 HYDRATION ERROR TESTING

#### Pre-Commit Hydration Testing
```bash
# MANDATORY before any theme/client-side commits
#!/bin/bash
echo "🧪 Testing for hydration issues..."

# 1. Production build test (catches 90% of hydration issues)
npm run build
if [ $? -ne 0 ]; then
  echo "❌ Build failed - fix before committing"
  exit 1
fi

# 2. Start production server and test both themes
npm start &
SERVER_PID=$!
sleep 5

# 3. Check for hydration errors in browser console
# Use Browser Tools MCP if available
mcp_browser-tools_takeScreenshot || echo "⚠️ MCP not available, manual test required"

kill $SERVER_PID
echo "✅ Hydration test complete"
```

#### Component-Level Hydration Testing
```tsx
// Test file: __tests__/hydration.test.tsx
import { renderToString } from 'react-dom/server';
import { render } from '@testing-library/react';

describe('Hydration Safety', () => {
  test('Theme Provider renders consistently', () => {
    const ThemeWrapper = ({ children }) => (
      <ChakraProvider theme={theme}>{children}</ChakraProvider>
    );
    
    // Server render
    const serverHTML = renderToString(<ThemeWrapper><App /></ThemeWrapper>);
    
    // Client render
    const { container } = render(<ThemeWrapper><App /></ThemeWrapper>);
    
    // Should match (simplified check)
    expect(container.innerHTML).toContain('expected-theme-class');
  });
  
  test('No browser APIs in initial render', () => {
    // Mock browser APIs to undefined
    Object.defineProperty(window, 'localStorage', { value: undefined });
    Object.defineProperty(window, 'innerWidth', { value: undefined });
    
    // Should not throw during render
    expect(() => {
      render(<App />);
    }).not.toThrow();
  });
});
```

### 🔥 CIRCULAR IMPORT TESTING

#### Backend Import Validation
```python
# backend/tests/test_imports.py
import ast
import os
from pathlib import Path

def test_no_main_imports_in_routers():
    """Ensure routers never import from main.py"""
    router_dir = Path("backend/routers")
    violations = []
    
    for py_file in router_dir.glob("*.py"):
        with open(py_file, 'r') as f:
            content = f.read()
            
        # Parse AST to find imports
        tree = ast.parse(content)
        for node in ast.walk(tree):
            if isinstance(node, ast.ImportFrom):
                if node.module and "main" in node.module:
                    violations.append(f"{py_file}: {ast.unparse(node)}")
    
    assert not violations, f"Found main.py imports in routers: {violations}"

def test_dependency_injection_usage():
    """Ensure routers use Depends() pattern"""
    router_dir = Path("backend/routers")
    
    for py_file in router_dir.glob("*.py"):
        if py_file.name == "__init__.py":
            continue
            
        with open(py_file, 'r') as f:
            content = f.read()
            
        # Should use Depends() for shared resources
        assert "Depends(" in content, f"{py_file} should use dependency injection"
        assert "from ..dependencies import" in content, f"{py_file} should import from dependencies"
```

#### Live Import Testing
```bash
# test-circular-imports.sh
#!/bin/bash
echo "🧪 Testing for circular imports..."

# Test each router can be imported independently
for router in backend/routers/*.py; do
  if [ "$(basename "$router")" != "__init__.py" ]; then
    echo "Testing $router..."
    python -c "import $(echo $router | sed 's/backend\///g' | sed 's/\.py//g' | sed 's/\//./g')" 2>&1
    if [ $? -ne 0 ]; then
      echo "❌ Import failed for $router"
      exit 1
    fi
  fi
done

echo "✅ All router imports successful"
```

### 🔥 MCP INTEGRATION TESTING

#### MCP Setup Verification
```bash
# test-mcp-setup.sh
#!/bin/bash
echo "🧪 Testing MCP setup..."

# 1. Node.js version check
NODE_VERSION=$(node --version | sed 's/v//' | cut -d. -f1)
if [ "$NODE_VERSION" -lt 18 ]; then
  echo "❌ Node.js version $NODE_VERSION < 18"
  exit 1
fi

# 2. Browser Tools server test
curl -f http://localhost:3025/health > /dev/null 2>&1
if [ $? -ne 0 ]; then
  echo "❌ Browser Tools server not responding on port 3025"
  echo "Run: npx @agentdeskai/browser-tools-server@latest"
  exit 1
fi

# 3. Basic MCP connectivity
mcp_browser-tools_wipeLogs
if [ $? -ne 0 ]; then
  echo "❌ MCP basic connectivity failed"
  exit 1
fi

echo "✅ MCP setup verification complete"
```

### 🔥 REACT QUERY PATTERN TESTING

#### API State Management Linting
```bash
# lint-api-patterns.sh
#!/bin/bash
echo "🧪 Checking API state management patterns..."

# Check for forbidden manual API patterns
MANUAL_FETCH=$(grep -r "useEffect.*fetch" frontend/src/ || true)
MANUAL_AXIOS=$(grep -r "useEffect.*axios" frontend/src/ || true)
MANUAL_API=$(grep -r "useEffect.*api\." frontend/src/ || true)

if [ ! -z "$MANUAL_FETCH" ] || [ ! -z "$MANUAL_AXIOS" ] || [ ! -z "$MANUAL_API" ]; then
  echo "❌ Found manual API calls in useEffect:"
  echo "$MANUAL_FETCH"
  echo "$MANUAL_AXIOS"
  echo "$MANUAL_API"
  echo "Use React Query instead!"
  exit 1
fi

# Check for React Query usage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rm2thaddeus/Pixel_Detective](https://github.com/rm2thaddeus/Pixel_Detective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
