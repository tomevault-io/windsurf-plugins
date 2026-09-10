---
trigger: always_on
description: How the WinUI app hosts a **React app in a WebView2** control to render chat messages (Markdown, syntax highlighting, selection, copy), and the C# ↔ JS message contract. Read this before working on anything that renders messages or touches `Conversation.xaml.cs`, `WebViewExtensions.cs`, the `WebView*Dto` files, or the `React Chat Renderer` project.
---

# Skill: WebView2 React chat renderer

How the WinUI app hosts a **React app in a WebView2** control to render chat messages (Markdown, syntax highlighting, selection, copy), and the C# ↔ JS message contract. Read this before working on anything that renders messages or touches `Conversation.xaml.cs`, `WebViewExtensions.cs`, the `WebView*Dto` files, or the `React Chat Renderer` project.

---

## What it does / why

`Conversation.xaml` contains a single **`<WebView2 x:Name="ChatWebView">`** that fills the chat area. It is deliberately **not** a XAML ListView — instead it loads a **prebuilt React bundle** and renders the conversation there. This gives full Markdown rendering, syntax highlighting, text selection, copy and scroll behavior that XAML ListViews are poor at.

The React source lives in `Source/React Chat Renderer/RendererApp/` (a **Create React App**: React 18, `@wooorm/starry-night` for syntax highlight, `Markdig` does markdown→HTML **on the C# side**, not in the browser). Its production build is zipped into **`ChatListUI.zip`** and shipped as a content asset of the `MdcAi.ChatUI` project.

## How the React app is served (not NavigateToString)

The C# host uses a **virtual HTTP server inside WebView2 via `WebResourceRequested`**:

1. `ChatWebView.CoreWebView2.Settings.IsWebMessageEnabled = true`.
2. `core.AddWebResourceRequestedFilter("http://localhost:3431/*", CoreWebView2WebResourceContext.All)`.
3. It sets `ChatWebView.Source = new Uri(@"http://localhost:3431/index.html")` (unless `Debugging.Enabled && Debugging.NpmRenderer`, in which case it points at a live dev server, e.g. `http://localhost:3000/`).
4. A `WebResourceRequested` event handler (`ProcessWebResource`) opens `ChatListUI.zip` (via `AppServices.GetAppFile("ChatListUI.zip")`), finds the entry matching the request path, maps its MIME via `WebViewExtensions.MimeTypes`, extracts it to an `InMemoryRandomAccessStream` (zip files aren't random-access) and returns `core.Environment.CreateWebResourceResponse(...)`.

So the WebView loads a locally-real URL (`http://localhost:3431/`) that is intercepted and served from the shipped zip. WebView2 caches responses, so the zip read happens only on first load.

## The message contract (`WebViewRequestDto { Name, Data }`)

**Both directions** use the same envelope: `WebViewRequestDto { string Name; object Data }` serialized with Newtonsoft.Json. There is **no schema/version field** — the `Name` discriminator is everything. (The React side defines a parallel structure.)

### C# → JS (the host sends via `CoreWebView2.PostWebMessageAsJson(...)`)
- **`SetMessages`** — `Data = WebViewSetMessagesRequestDto { Messages: WebViewChatMessageDto[] }`. Each message payload:
  ```jsonc
  {
    "Id": "...", "Role": "user|assistant",
    "Content": "<html>",      // pre-rendered HTML (from Markdig)
    "Version": 1, "VersionCount": 1, "CreatedTs": "...",
    "Model": "gpt-4o",        // model that produced the message (assistant msgs; null on user/legacy)
    "Provider": "OpenAI",     // provider display name serving that model (null when unknown)
    "Effort": "medium"        // reasoning effort that produced the message (assistant msgs; null on user/legacy/effort-less models)
  }
  ```

`Content` is set from `m.HTMLContent ?? $"<p>{m.Content}</p>"` (see `ChatMessageVmExt.GetWebViewDto`).

- **`SetSelection`** — `Data` is an **int index** into the messages array (used when the app wants to move selection in JS).
- **`HideCaret`** — signals the JS to remove the streaming typing-caret marker from the last message (the JS hides `#caret` elements after a delay).

### JS → C# (`window.chrome.webview.postMessage(obj)`, received via `CoreWebView2.WebMessageReceived`)
Consumed in `Conversation.xaml.cs` and deserialized to `WebViewRequestDto`:

- **`Ready`** — React app signals it has mounted and registered its handler. C# uses this as a "now safe to push messages" gate (`webReady`); it replays the latest `SetMessages` request.
- **`SetSelection`** — `Data` = index of clicked message → C# sets `SelectedMessage`.
- **`IsScrollToBottom`** — `Data` = bool; C# records whether the user is scrolled to bottom, to decide auto-scroll later.
- **`LogDebug` / `LogInfo` / `LogError`** — logging from the React side (message data carries `{Message, Stack, Name}`).

So if you add a new feature to the renderer, add a new `Name` value + its `Data` shape consistently to BOTH the C# (plus the `WebView*Dto` if you want a typed shape) and the React `handleMessage` dispatcher.

## Scroll behavior

- `WebViewExtensions.IsScrolledToBottom()` / `ScrollToBottom()` use `ExecuteScriptAsync`.
- On the C# side a `Subject<Unit> scrollToBottom` is throttled (500 ms) and only triggers a scroll if the user is currently scrolled down (`isScrolledDown`), and is fed from `PromptField.Events().BeforeTextChanging` when newlines are typed (because an auto-growing prompt reflows the WebView and up-scrolls).

## The React side (how it renders)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bawkee/MdcAi](https://github.com/bawkee/MdcAi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
