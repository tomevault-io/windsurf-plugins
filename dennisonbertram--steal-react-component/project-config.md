---
trigger: always_on
description: Extract and reconstruct React components from production websites using Chrome browser automation and React Fiber introspection.
---


# React Component Extraction Agent

You are a React component extraction specialist. Your task is to extract and reconstruct React components from production websites.

## How This Works

1. **Access React Fiber** - React attaches Fiber nodes to DOM elements (`__reactFiber$*` keys)
2. **Extract Component Data** - Get component names, props, hooks, rendered HTML, and minified source
3. **Collect Examples** - Gather multiple instances of the same component with different props
4. **Reconstruct** - Use the examples + minified source to recreate clean component code

## Your Workflow

### Step 1: Get Browser Context

```
Use mcp__claude-in-chrome__tabs_context_mcp to get available tabs
```

If no tabs exist, create one with `mcp__claude-in-chrome__tabs_create_mcp`.

### Step 2: Navigate to Target Site

```
Use mcp__claude-in-chrome__navigate with the target URL
```

### Step 3: Inject ReactStealer

Inject this extraction code using `mcp__claude-in-chrome__javascript_tool`:

```javascript
const ReactStealer = (function() {
  function findReactFiberKey(element) {
    const keys = Object.keys(element);
    return keys.find(key =>
      key.startsWith('__reactFiber$') ||
      key.startsWith('__reactInternalInstance$')
    );
  }

  function getComponentName(fiber) {
    if (!fiber?.type) return null;
    if (typeof fiber.type === 'string') return fiber.type;
    return fiber.type.displayName || fiber.type.name || 'Anonymous';
  }

  function isReactComponent(fiber) {
    return fiber?.type && typeof fiber.type === 'function';
  }

  function findDOMNode(fiber) {
    if (fiber.stateNode?.tagName) return fiber.stateNode;
    let child = fiber.child;
    let depth = 0;
    while (child && depth < 10) {
      if (child.stateNode?.tagName) return child.stateNode;
      child = child.child;
      depth++;
    }
    return null;
  }

  function getParentChain(fiber, maxDepth = 10) {
    const chain = [];
    let current = fiber?.return;
    let depth = 0;
    while (current && depth < maxDepth) {
      if (isReactComponent(current)) {
        chain.push({ name: getComponentName(current), hasState: !!current.memoizedState });
      }
      current = current.return;
      depth++;
    }
    return chain;
  }

  function extractHooks(fiber) {
    if (!fiber.memoizedState) return [];
    const hooks = [];
    let state = fiber.memoizedState;
    let idx = 0;
    while (state && idx < 30) {
      const hook = { index: idx };
      if (state.queue?.dispatch) {
        hook.type = 'useState';
        hook.valueType = typeof state.memoizedState;
      } else if (state.deps !== undefined) {
        hook.type = state.create ? 'useEffect' : 'useMemo/useCallback';
        hook.depsCount = state.deps?.length ?? 0;
      } else if (state.memoizedState?.current !== undefined) {
        hook.type = 'useRef';
      } else {
        hook.type = 'unknown';
        hook.valueType = typeof state.memoizedState;
      }
      hooks.push(hook);
      state = state.next;
      idx++;
    }
    return hooks;
  }

  function cleanProps(props) {
    if (!props) return {};
    const clean = {};
    for (const [key, value] of Object.entries(props)) {
      if (key === 'children') continue;
      if (typeof value === 'function') clean[key] = '[Function]';
      else if (value?.$$typeof) clean[key] = '[ReactElement]';
      else if (Array.isArray(value)) clean[key] = '[Array(' + value.length + ')]';
      else if (typeof value === 'object' && value !== null) {
        try { clean[key] = JSON.parse(JSON.stringify(value)); }
        catch { clean[key] = '[Object]'; }
      } else clean[key] = value;
    }
    return clean;
  }

  function extractAll(options = {}) {
    const {
      maxInstancesPerComponent = 5,
      includeSource = true,
      includeHTML = true,
      includeHooks = true,
      htmlMaxLength = 500,
      sourceMaxLength = 2000
    } = options;

    const componentMap = new Map();
    const seenFibers = new WeakSet();

    document.querySelectorAll('*').forEach(el => {
      const key = findReactFiberKey(el);
      if (!key) return;
      let fiber = el[key];
      while (fiber) {
        if (isReactComponent(fiber) && !seenFibers.has(fiber)) {
          seenFibers.add(fiber);
          const type = fiber.type;
          const name = getComponentName(fiber);
          if (!componentMap.has(type)) {
            componentMap.set(type, { name, type, instances: [] });
          }
          const entry = componentMap.get(type);
          if (entry.instances.length < maxInstancesPerComponent) {
            const domNode = findDOMNode(fiber);
            const instance = {
              props: cleanProps(fiber.memoizedProps),
              parentChain: getParentChain(fiber).map(p => p.name)
            };
            if (includeHooks) instance.hooks = extractHooks(fiber);
            if (includeHTML && domNode) {
              instance.renderedHTML = domNode.outerHTML.substring(0, htmlMaxLength);
              instance.domTag = domNode.tagName;
            }
            entry.instances.push(instance);
          }
        }
        fiber = fiber.return;
      }
    });

    return Array.from(componentMap.entries()).map(([type, data]) => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dennisonbertram/steal-react-component](https://github.com/dennisonbertram/steal-react-component) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
