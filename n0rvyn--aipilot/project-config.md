---
trigger: always_on
description: Markdown 渲染和 CodeMirror 检查清单
---


# Markdown 渲染检查清单

> **重要性**: ⭐⭐⭐⭐ 高优先级  
> **加载方式**: Auto-Attach（编辑 Markdown 相关文件时自动加载）  
> **用途**: Markdown 渲染和 CodeMirror 集成的最佳实践  

---

## 🔒 安全渲染

### 1. 使用 Obsidian 的 MarkdownRenderer

```typescript
import { MarkdownRenderer } from 'obsidian';

// ✅ 推荐：使用 Obsidian 的安全渲染器
async function renderMarkdown(markdown: string, container: HTMLElement) {
    container.empty();
    
    await MarkdownRenderer.render(
        this.app,
        markdown,
        container,
        '', // sourcePath（可选）
        this // component
    );
}

// ❌ 危险：直接插入 HTML
container.innerHTML = marked(markdown); // XSS 风险！
```

### 2. 清理用户输入

```typescript
// ✅ Obsidian 的渲染器会自动处理
await MarkdownRenderer.render(
    this.app,
    userInput, // 安全，会自动清理
    container,
    '',
    this
);

// ✅ 如果必须使用 marked，配置安全选项
import { marked } from 'marked';

marked.setOptions({
    sanitize: false, // marked 不再支持 sanitize
    breaks: true,
    gfm: true
});

// 使用 DOMPurify 清理（如果需要）
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(html);
```

---

## 📝 Markdown 渲染模式

### Obsidian MarkdownRenderer API

```typescript
// 基本用法
await MarkdownRenderer.render(
    this.app,              // App 实例
    markdown,              // Markdown 字符串
    container,             // 渲染目标容器
    sourcePath,            // 源文件路径（用于链接解析）
    component              // 组件实例（用于生命周期管理）
);

// 渲染后处理
await MarkdownRenderer.render(this.app, markdown, container, '', this);

// 添加自定义类
container.addClass('my-markdown-content');

// 处理内部链接
container.querySelectorAll('a.internal-link').forEach(link => {
    link.addEventListener('click', (e) => {
        e.preventDefault();
        const target = link.getAttribute('data-href');
        this.app.workspace.openLinkText(target, '');
    });
});
```

---

## 🎨 代码高亮

### 使用 CodeMirror 渲染代码块

```typescript
import { EditorView } from '@codemirror/view';
import { EditorState } from '@codemirror/state';
import { javascript } from '@codemirror/lang-javascript';
import { python } from '@codemirror/lang-python';
import { oneDark } from '@codemirror/theme-one-dark';

// ✅ 创建 CodeMirror 编辑器
function renderCodeBlock(code: string, language: string, container: HTMLElement) {
    const extensions = [
        EditorView.editable.of(false), // 只读
        EditorState.readOnly.of(true),
        getLanguageExtension(language),
        oneDark // 主题（可选）
    ];
    
    const state = EditorState.create({
        doc: code,
        extensions
    });
    
    const view = new EditorView({
        state,
        parent: container
    });
    
    return view;
}

function getLanguageExtension(language: string) {
    switch (language.toLowerCase()) {
        case 'javascript':
        case 'js':
            return javascript();
        case 'python':
        case 'py':
            return python();
        // 添加更多语言支持
        default:
            return [];
    }
}
```

### 代码块复制功能

```typescript
// ✅ 添加复制按钮
function addCopyButton(codeBlock: HTMLElement, code: string) {
    const copyBtn = codeBlock.createEl('button', {
        text: 'Copy',
        cls: 'copy-code-button'
    });
    
    copyBtn.addEventListener('click', async () => {
        try {
            await navigator.clipboard.writeText(code);
            copyBtn.textContent = 'Copied!';
            setTimeout(() => {
                copyBtn.textContent = 'Copy';
            }, 2000);
        } catch (error) {
            console.error('Failed to copy:', error);
        }
    });
}
```

---

## 🎯 自定义 Markdown 处理

### 扩展 Markdown 语法

```typescript
import { marked } from 'marked';

// ✅ 自定义渲染器
const renderer = new marked.Renderer();

// 自定义链接渲染
renderer.link = (href, title, text) => {
    const isInternal = href?.startsWith('[[') && href?.endsWith(']]');
    
    if (isInternal) {
        const target = href.slice(2, -2);
        return `<a class="internal-link" data-href="${target}">${text}</a>`;
    }
    
    return `<a href="${href}" title="${title || ''}" target="_blank">${text}</a>`;
};

// 自定义代码块渲染
renderer.code = (code, language) => {
    const lang = language || 'text';
    return `<div class="code-block" data-language="${lang}">
        <pre><code class="language-${lang}">${escapeHtml(code)}</code></pre>
    </div>`;
};

// 使用自定义渲染器
marked.use({ renderer });
```

### 处理数学公式（LaTeX）

```typescript
// ✅ 使用 Obsidian 的数学渲染
async function renderMath(latex: string, container: HTMLElement, displayMode: boolean) {
    // Obsidian 会自动处理 $...$ 和 $$...$$ 语法
    // 通过 MarkdownRenderer.render() 即可
    await MarkdownRenderer.render(
        this.app,
        displayMode ? `$$${latex}$$` : `$${latex}$`,
        container,
        '',
        this
    );
}
```

---

## 🎭 流式渲染（Streaming）

### 实时更新 Markdown 内容

```typescript
class StreamingRenderer {
    private container: HTMLElement;
    private currentContent: string = '';
    
    constructor(container: HTMLElement) {
        this.container = container;
    }
    
    // ✅ 追加内容并重新渲染
    async append(chunk: string) {
        this.currentContent += chunk;
        await this.render();
    }
    
    // ✅ 渲染当前内容
    async render() {
        this.container.empty();
        await MarkdownRenderer.render(
            this.app,
            this.currentContent,
            this.container,
            '',
            this
        );
    }
    
    // ✅ 清空内容
    clear() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/n0rvyn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
