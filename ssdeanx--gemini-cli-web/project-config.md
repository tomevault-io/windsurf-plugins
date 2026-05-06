---
trigger: always_on
description: The Gemini CLI Web UI is a full-stack application designed to provide a rich, interactive web interface for the Gemini Command Line Interface (CLI) tool. It acts as a bridge, enabling users to interact with the Gemini CLI, manage projects, edit files, and track Git changes directly from their browser. The architecture emphasizes real-time communication, robust authentication, and a modular component structure for maintainability and scalability.
---

# Gemini CLI Web UI Architecture

**Overview:**
The Gemini CLI Web UI is a full-stack application designed to provide a rich, interactive web interface for the Gemini Command Line Interface (CLI) tool. It acts as a bridge, enabling users to interact with the Gemini CLI, manage projects, edit files, and track Git changes directly from their browser. The architecture emphasizes real-time communication, robust authentication, and a modular component structure for maintainability and scalability.

**Quick Links:**

*   [Backend Architecture](./server/GEMINI.md)
*   [Frontend Architecture](./src/GEMINI.md)
*   [Configuration](./documentation/configuration.md)
*   [Checkpointing](./documentation/checkpointing.md)
*   [Deployment](./documentation/deployment.md)
*   [Extensions](./documentation/extension.md)
*   [Keyboard Shortcuts](./documentation/keyboard-shortcuts.md)
*   [Memory Import Processor](./documentation/memport.md)
*   [Proxy Script](./documentation/proxy-script.md)
*   [Sandboxing](./documentation/sandbox.md)
*   [Telemetry](./documentation/telemetry.md)
*   [Tools API](./documentation/tools-api.md)
*   [Troubleshooting](./documentation/troubleshooting.md)

## Architecture Diagram

```mermaid
graph TD
    subgraph User
        A[User]
    end

    subgraph Browser (Frontend)
        subgraph "React UI Components"
            App(App.jsx)
            MainContent(MainContent.jsx)
            Sidebar(Sidebar.jsx)
            ChatInterface(ChatInterface.jsx)
            EditorTab(EditorTab.jsx)
            GitPanel(GitPanel.jsx)
            Shell(Shell.jsx)
            ChatSidebar(ChatSidebar.jsx)
            CodeEditor(CodeEditor.jsx)
            CodeTabs(CodeTabs.jsx)
            DarkModeToggle(DarkModeToggle.jsx)
            EditorFileTree(EditorFileTree.jsx)
            ErrorBoundary(ErrorBoundary.jsx)
            FileTree(FileTree.jsx)
            GeminiLogo(GeminiLogo.jsx)
            GeminiStatus(GeminiStatus.jsx)
            ImageViewer(ImageViewer.jsx)
            LoginForm(LoginForm.jsx)
            MicButton(MicButton.jsx)
            MobileNav(MobileNav.jsx)
            NewCodeEditor(NewCodeEditor.jsx)
            ProtectedRoute(ProtectedRoute.jsx)
            QuickSettingsPanel(QuickSettingsPanel.jsx)
            SetupForm(SetupForm.jsx)
            TodoList(TodoList.jsx)
            ToolsSettings(ToolsSettings.jsx)
            SpecDesign(SpecDesign.jsx)
            ProgressIndicator(ProgressIndicator.jsx)
            ThinkingIndicator(ThinkingIndicator.jsx)
            badge(badge.jsx)
            button(button.jsx)
            input(input.jsx)
            scroll_area(scroll-area.jsx)
        end
        subgraph "Contexts"
            AuthContext(AuthContext.jsx)
            ThemeContext(ThemeContext.jsx)
            SettingsContext(SettingsContext.jsx)
        end
        subgraph "Hooks"
            useWebSocket(useWebSocket.js)
            useAudioRecorder(useAudioRecorder.js)
            useVersionCheck(useVersionCheck.js)
        end
        subgraph "Utils"
            api(api.js)
            notificationSound(notificationSound.js)
            whisper(whisper.js)
            lib_utils(utils.js)
        end
    end

    subgraph "Node.js Server (Backend)"
        ExpressServer(Express Server)
        WebSocketServer(WebSocket Server)
        GeminiCLIBridg(Gemini CLI Bridge)
        SessionManager(Session Manager)
        AuthLayer(Auth Layer)
        GitAPI(Git API)
    end

    subgraph "Local System"
        GeminiCLIProcess(Gemini CLI Process)
        ProjectFiles(Project Files)
        GitRepository(Git Repository)
        SQLiteDB(SQLite DB)
        GeminiConfigFiles(gemini config files)
    end

    subgraph "External Services"
        OpenAIWhisperAPI(OpenAI Whisper API)
    end

    A -- Interacts --> App

    App --> MainContent
    App --> Sidebar
    App --> DarkModeToggle
    App --> ErrorBoundary
    App --> MobileNav
    App --> ProtectedRoute
    App --> useVersionCheck

    MainContent --> ChatInterface
    MainContent --> EditorTab
    MainContent --> GitPanel
    MainContent --> Shell
    MainContent --> ChatSidebar
    MainContent --> ImageViewer
    MainContent --> TodoList
    MainContent --> SpecDesign

    Sidebar --> FileTree
    Sidebar --> GeminiLogo
    Sidebar --> QuickSettingsPanel
    Sidebar --> ToolsSettings
    Sidebar --> api

    ChatInterface --> CodeEditor
    ChatInterface --> GeminiStatus
    ChatInterface --> MicButton
    ChatInterface --> useWebSocket
    ChatInterface --> api
    ChatInterface --> notificationSound

    EditorTab --> CodeTabs
    EditorTab --> EditorFileTree
    EditorTab --> NewCodeEditor

    SpecDesign --> ProgressIndicator
    SpecDesign --> ThinkingIndicator

    AuthContext --> LoginForm
    AuthContext --> SetupForm

    MicButton --> useAudioRecorder
    useAudioRecorder --> whisper

    App --> AuthContext

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ssdeanx/Gemini-CLI-Web](https://github.com/ssdeanx/Gemini-CLI-Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
