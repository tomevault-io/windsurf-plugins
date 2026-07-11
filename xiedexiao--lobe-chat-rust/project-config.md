---
trigger: always_on
description: 本文档详细说明了 LobeChat 项目中 Zustand Action 的组织方式、命名规范和实现模式，特别关注乐观更新与后端服务的集成。
---

# LobeChat Zustand Action 组织模式

本文档详细说明了 LobeChat 项目中 Zustand Action 的组织方式、命名规范和实现模式，特别关注乐观更新与后端服务的集成。

## Action 类型分层

LobeChat 的 Action 采用分层架构，明确区分不同职责：

### 1. Public Actions
对外暴露的主要接口，供 UI 组件调用：
- 命名：动词形式（`createTopic`, `sendMessage`, `updateTopicTitle`）
- 职责：参数验证、流程编排、调用 internal actions
- 示例：[src/store/chat/slices/topic/action.ts](mdc:src/store/chat/slices/topic/action.ts)

```typescript
// Public Action 示例
createTopic: async () => {
  const { activeId, internal_createTopic } = get();
  const messages = chatSelectors.activeBaseChats(get());
  
  if (messages.length === 0) return;
  
  const topicId = await internal_createTopic({
    sessionId: activeId,
    title: t('defaultTitle', { ns: 'topic' }),
    messages: messages.map((m) => m.id),
  });
  
  return topicId;
},
```

### 2. Internal Actions (`internal_*`)
内部实现细节，处理核心业务逻辑：
- 命名：`internal_` 前缀 + 动词（`internal_createTopic`, `internal_updateMessageContent`）
- 职责：乐观更新、服务调用、错误处理、状态同步
- 不应该被 UI 组件直接调用

```typescript
// Internal Action 示例 - 乐观更新模式
internal_createTopic: async (params) => {
  const tmpId = Date.now().toString();
  
  // 1. 立即更新前端状态（乐观更新）
  get().internal_dispatchTopic(
    { type: 'addTopic', value: { ...params, id: tmpId } },
    'internal_createTopic',
  );
  get().internal_updateTopicLoading(tmpId, true);
  
  // 2. 调用后端服务
  const topicId = await topicService.createTopic(params);
  get().internal_updateTopicLoading(tmpId, false);
  
  // 3. 刷新数据确保一致性
  get().internal_updateTopicLoading(topicId, true);
  await get().refreshTopic();
  get().internal_updateTopicLoading(topicId, false);
  
  return topicId;
},
```

### 3. Dispatch Methods (`internal_dispatch*`)
专门处理状态更新的方法：
- 命名：`internal_dispatch` + 实体名（`internal_dispatchTopic`, `internal_dispatchMessage`）
- 职责：调用 reducer、更新 Zustand store、处理状态对比

```typescript
// Dispatch Method 示例
internal_dispatchTopic: (payload, action) => {
  const nextTopics = topicReducer(topicSelectors.currentTopics(get()), payload);
  const nextMap = { ...get().topicMaps, [get().activeId]: nextTopics };

  if (isEqual(nextMap, get().topicMaps)) return;
  
  set({ topicMaps: nextMap }, false, action ?? n(`dispatchTopic/${payload.type}`));
},
```

## 何时使用 Reducer 模式 vs. 简单 `set`

### 使用 Reducer 模式的场景

适用于复杂的数据结构管理，特别是：
- 管理对象列表或映射（如 `messagesMap`, `topicMaps`）
- 需要乐观更新的场景
- 状态转换逻辑复杂
- 需要类型安全的 action payload

```typescript
// Reducer 模式示例 - 复杂消息状态管理
export const messagesReducer = (state: ChatMessage[], payload: MessageDispatch): ChatMessage[] => {
  switch (payload.type) {
    case 'updateMessage': {
      return produce(state, (draftState) => {
        const index = draftState.findIndex((i) => i.id === payload.id);
        if (index < 0) return;
        draftState[index] = merge(draftState[index], { 
          ...payload.value, 
          updatedAt: Date.now() 
        });
      });
    }
    case 'createMessage': {
      return produce(state, (draftState) => {
        draftState.push({ 
          ...payload.value, 
          id: payload.id,
          createdAt: Date.now(),
          updatedAt: Date.now(),
          meta: {}
        });
      });
    }
    // ...其他复杂状态转换
  }
};
```

### 使用简单 `set` 的场景

适用于简单状态更新：
- 切换布尔值
- 更新简单字符串/数字
- 设置单一状态字段

```typescript
// 简单 set 示例
updateInputMessage: (message) => {
  if (isEqual(message, get().inputMessage)) return;
  set({ inputMessage: message }, false, n('updateInputMessage'));
},

togglePortal: (open?: boolean) => {
  set({ showPortal: open ?? !get().showPortal }, false, 'togglePortal');
},
```

## 乐观更新实现模式

乐观更新是 LobeChat 中的核心模式，用于提供流畅的用户体验：

### 标准乐观更新流程

```typescript
// 完整的乐观更新示例
internal_updateMessageContent: async (id, content, extra) => {
  const { internal_dispatchMessage, refreshMessages } = get();

  // 1. 立即更新前端状态（乐观更新）
  internal_dispatchMessage({
    id,
    type: 'updateMessage',
    value: { content },
  });

  // 2. 调用后端服务
  await messageService.updateMessage(id, {
    content,
    tools: extra?.toolCalls ? internal_transformToolCalls(extra.toolCalls) : undefined,
    // ...其他字段
  });

  // 3. 刷新确保数据一致性
  await refreshMessages();
},
```

### 创建操作的乐观更新

```typescript
internal_createMessage: async (message, context) => {
  const { internal_createTmpMessage, refreshMessages, internal_toggleMessageLoading } = get();
  
  let tempId = context?.tempMessageId;
  if (!tempId) {
    // 创建临时消息用于乐观更新
    tempId = internal_createTmpMessage(message);
    internal_toggleMessageLoading(true, tempId);
  }

  try {
    const id = await messageService.createMessage(message);
    if (!context?.skipRefresh) {
      await refreshMessages();
    }
    internal_toggleMessageLoading(false, tempId);
    return id;
  } catch (e) {
    internal_toggleMessageLoading(false, tempId);
    // 错误处理：更新消息错误状态
    internal_dispatchMessage({
      id: tempId,
      type: 'updateMessage',
      value: { error: { type: ChatErrorType.CreateMessageError, message: e.message } },
    });
  }
},
```

### 删除操作模式（不使用乐观更新）

删除操作通常不适合乐观更新，因为：
- 删除是破坏性操作，错误恢复复杂
- 用户对删除操作的即时反馈期望较低
- 删除失败时恢复原状态会造成困惑

```typescript
// 删除操作的标准模式 - 无乐观更新
removeGenerationTopic: async (id: string) => {
  const { internal_removeGenerationTopic } = get();
  await internal_removeGenerationTopic(id);
},

internal_removeGenerationTopic: async (id: string) => {
  // 1. 显示加载状态

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xiedexiao/lobe-chat_rust](https://github.com/Xiedexiao/lobe-chat_rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
