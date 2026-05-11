---
trigger: always_on
description: Comprehensive React component development guidelines for LobeHub Editor
---


# React Component Guidelines

## Component Architecture

LobeHub Editor uses React components in two main contexts:

1. **High-level components** in [src/react/](mdc:src/react/) - Main user-facing components
2. **Plugin components** in [src/plugins/\*/react/](mdc:src/plugins/) - Plugin-specific React implementations

### Main Components

Primary components in [src/react/](mdc:src/react/):

- [Editor](mdc:src/react/Editor/) - Core editor with plugin system
- [ChatInput](mdc:src/react/ChatInput/) - Chat interface container
- [ChatInputActions](mdc:src/react/ChatInputActions/) - Action management
- [ChatInputActionBar](mdc:src/react/ChatInputActionBar/) - Action layout
- [SendButton](mdc:src/react/SendButton/) - Send functionality
- [CodeLanguageSelect](mdc:src/react/CodeLanguageSelect/) - Language selection

## Component Development Pattern

### Standard Component Definition

```typescript
import { memo, useCallback, useState, ReactNode, CSSProperties } from 'react';
import { LexicalEditor } from 'lexical';
import { createDebugLogger } from '@/utils/debug';

export interface ComponentNameProps {
  /** Required prop with JSDoc description */
  requiredProp: string;
  /** Optional prop with default value */
  optionalProp?: boolean;
  /** Event handler callback */
  onEvent?: (data: EventData) => void;
  /** Children for composition */
  children?: ReactNode;
  /** Style customization */
  className?: string;
  style?: CSSProperties;
}

export const ComponentName = memo<ComponentNameProps>(({
  requiredProp,
  optionalProp = false,
  onEvent,
  children,
  className,
  style,
  ...rest
}) => {
  // Hooks and state
  const [state, setState] = useState(initialValue);
  const { styles, cx } = useStyles();
  const logger = createDebugLogger('react', 'component-name');
  
  // Event handlers with useCallback
  const handleEvent = useCallback((data: EventData) => {
    logger.debug('Event triggered:', data);
    onEvent?.(data);
  }, [onEvent, logger]);
  
  // Render
  return (
    <div 
      className={cx(styles.container, className)}
      style={style}
      {...rest}
    >
      {children}
    </div>
  );
});

ComponentName.displayName = 'ComponentName';
```

### Plugin Component Pattern

```typescript
export interface ReactPluginNameProps {
  /** Editor instance for command dispatch */
  editor?: LexicalEditor;
  /** Plugin configuration */
  config?: PluginConfig;
  /** Plugin state management */
  state?: PluginState;
  /** Plugin event callbacks */
  onPluginEvent?: (event: PluginEvent) => void;
}

export const ReactPluginName = memo<ReactPluginNameProps>(({
  editor,
  config,
  state,
  onPluginEvent,
}) => {
  const logger = createDebugLogger('plugin', 'plugin-name');
  
  // Command dispatch to editor
  const handleCommand = useCallback((command: Command, payload: any) => {
    logger.debug('Dispatching command:', command, payload);
    editor?.dispatchCommand(command, payload);
  }, [editor, logger]);
  
  // Plugin-specific event handling
  const handlePluginEvent = useCallback((event: PluginEvent) => {
    logger.debug('Plugin event:', event);
    onPluginEvent?.(event);
  }, [onPluginEvent, logger]);
  
  return (
    <div className="plugin-container">
      {/* Plugin UI implementation */}
    </div>
  );
});

ReactPluginName.displayName = 'ReactPluginName';
```

## Editor Integration Patterns

### Main Editor Usage

```typescript
import { Editor, useEditor } from '@lobehub/editor/react';
import { 
  ReactSlashPlugin, 
  ReactMentionPlugin,
  ReactCodeblockPlugin 
} from '@lobehub/editor';

function MyEditorComponent() {
  const editor = useEditor();
  
  const handleChange = useCallback((editor: IEditor) => {
    const content = editor.getDocument('markdown');
    // Handle content changes
  }, []);
  
  return (
    <Editor
      editorRef={editor}
      placeholder="Start typing..."
      plugins={[
        ReactSlashPlugin,
        ReactMentionPlugin,
        ReactCodeblockPlugin,
      ]}
      slashOption={{
        items: slashCommands,
      }}
      mentionOption={{
        items: mentionItems,
      }}
      onChange={handleChange}
    />
  );
}
```

### Chat Interface Pattern

```typescript
import { ChatInput, ChatInputActionBar, ChatInputActions, SendButton } from '@lobehub/editor/react';

function ChatInterface() {
  const handleSend = useCallback(() => {
    // Handle send action
  }, []);
  
  return (
    <ChatInput
      header={<ChatHeader />}
      footer={
        <ChatInputActionBar
          left={<ChatInputActions items={leftActions} />}
          right={<SendButton onSend={handleSend} />}
        />
      }
    >
      <Editor {...editorProps} />
    </ChatInput>
  );
}
```

## Styling Standards

### Style Hooks with antd-style

```typescript
import { createStyles } from 'antd-style';

const useStyles = createStyles(({ css, token, isDarkMode }) => ({
  container: css`
    padding: ${token.padding}px;
    border-radius: ${token.borderRadius}px;
    background: ${isDarkMode ? token.colorBgElevated : token.colorBgContainer};
    border: 1px solid ${token.colorBorder};
    
    &:hover {
      border-color: ${token.colorPrimary};
    }
  `,
  
  active: css`
    background: ${token.colorPrimary};
    color: ${token.colorTextLightSolid};
  `,
  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lobehub/lobe-editor](https://github.com/lobehub/lobe-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
