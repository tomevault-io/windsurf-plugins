---
trigger: always_on
description: This file defines React patterns and best practices for any future web UI components in the Code-Index-MCP project.
---

# React Rules for Code-Index-MCP

## Overview
This file defines React patterns and best practices for any future web UI components in the Code-Index-MCP project.

## Component Structure

### Functional Components with TypeScript
```tsx
import React, { useState, useCallback, memo } from 'react';

interface CodeSearchProps {
  onSearch: (query: string) => Promise<void>;
  placeholder?: string;
  className?: string;
}

export const CodeSearch = memo<CodeSearchProps>(({ 
  onSearch, 
  placeholder = "Search code...",
  className = ""
}) => {
  const [query, setQuery] = useState('');
  const [isSearching, setIsSearching] = useState(false);

  const handleSearch = useCallback(async (e: React.FormEvent) => {
    e.preventDefault();
    if (!query.trim()) return;

    setIsSearching(true);
    try {
      await onSearch(query);
    } finally {
      setIsSearching(false);
    }
  }, [query, onSearch]);

  return (
    <form onSubmit={handleSearch} className={className}>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder={placeholder}
        disabled={isSearching}
      />
      <button type="submit" disabled={isSearching}>
        {isSearching ? 'Searching...' : 'Search'}
      </button>
    </form>
  );
});

CodeSearch.displayName = 'CodeSearch';
```

## Custom Hooks

### Data Fetching Hook
```tsx
import { useState, useEffect, useCallback } from 'react';

interface UseApiOptions<T> {
  initialData?: T;
  onError?: (error: Error) => void;
}

function useApi<T>(
  apiCall: () => Promise<T>,
  options: UseApiOptions<T> = {}
) {
  const [data, setData] = useState<T | undefined>(options.initialData);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const execute = useCallback(async () => {
    setLoading(true);
    setError(null);

    try {
      const result = await apiCall();
      setData(result);
      return result;
    } catch (err) {
      const error = err instanceof Error ? err : new Error(String(err));
      setError(error);
      options.onError?.(error);
      throw error;
    } finally {
      setLoading(false);
    }
  }, [apiCall, options.onError]);

  return { data, loading, error, execute };
}

// Usage
function SymbolViewer({ symbolName }: { symbolName: string }) {
  const { data, loading, error, execute } = useApi(
    () => client.getSymbolDefinition(symbolName),
    { onError: (err) => console.error('Failed to load symbol:', err) }
  );

  useEffect(() => {
    execute();
  }, [execute]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!data) return null;

  return <SymbolDetails symbol={data} />;
}
```

### Code Editor Integration Hook
```tsx
interface UseCodeNavigationOptions {
  onNavigate?: (location: CodeLocation) => void;
}

interface CodeLocation {
  file: string;
  line: number;
  column: number;
}

function useCodeNavigation(options: UseCodeNavigationOptions = {}) {
  const navigate = useCallback((location: CodeLocation) => {
    // Integrate with VS Code or other editors
    if (window.vscode) {
      window.vscode.postMessage({
        command: 'openFile',
        file: location.file,
        line: location.line,
        column: location.column
      });
    }
    
    options.onNavigate?.(location);
  }, [options.onNavigate]);

  return { navigate };
}
```

## State Management Patterns

### Context for Global State
```tsx
interface CodeIndexContextType {
  client: MCPClient;
  currentProject: string;
  setCurrentProject: (project: string) => void;
}

const CodeIndexContext = React.createContext<CodeIndexContextType | null>(null);

export function CodeIndexProvider({ 
  children, 
  apiKey,
  baseUrl = 'http://localhost:8000'
}: { 
  children: React.ReactNode;
  apiKey: string;
  baseUrl?: string;
}) {
  const [currentProject, setCurrentProject] = useState('');
  const client = useMemo(
    () => new MCPClient(baseUrl, apiKey),
    [baseUrl, apiKey]
  );

  return (
    <CodeIndexContext.Provider value={{
      client,
      currentProject,
      setCurrentProject
    }}>
      {children}
    </CodeIndexContext.Provider>
  );
}

export function useCodeIndex() {
  const context = useContext(CodeIndexContext);
  if (!context) {
    throw new Error('useCodeIndex must be used within CodeIndexProvider');
  }
  return context;
}
```

## Performance Optimization

### Virtual Scrolling for Large Lists
```tsx
import { FixedSizeList } from 'react-window';

interface SearchResultsProps {
  results: SearchResult[];
  onItemClick: (item: SearchResult) => void;
}

function SearchResults({ results, onItemClick }: SearchResultsProps) {
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => {
    const item = results[index];
    
    return (
      <div 
        style={style} 
        onClick={() => onItemClick(item)}
        className="search-result-item"
      >
        <div className="file-path">{item.file}</div>
        <div className="line-number">Line {item.line}</div>
        <pre className="code-snippet">{item.content}</pre>
      </div>
    );
  };

  return (
    <FixedSizeList
      height={600}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Consiliency/Code-Index-MCP](https://github.com/Consiliency/Code-Index-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
