---
trigger: always_on
description: Universal Game Translator Live project overview and structure
---


# Universal Game Translator Live - Project Guide

## Overview
Universal Game Translator Live (UGTLive) is a Windows WPF application that provides real-time screen translation using OCR and Large Language Models (LLMs). The application captures screen regions, performs OCR, and translates text using services like Gemini, ChatGPT, Ollama, Google Translate, or llama.cpp.

## Project Structure

### Core Application Files
- [src/App.xaml](mdc:src/App.xaml) - WPF application entry point
- [src/MainWindow.xaml.cs](mdc:src/MainWindow.xaml.cs) - Main application window and control logic
- [src/Logic.cs](mdc:src/Logic.cs) - Core translation and OCR processing logic
- [src/ConfigManager.cs](mdc:src/ConfigManager.cs) - Settings and configuration management

### UI Components
- [src/MonitorWindow.xaml.cs](mdc:src/MonitorWindow.xaml.cs) - Screen capture preview window
- [src/ChatBoxWindow.xaml.cs](mdc:src/ChatBoxWindow.xaml.cs) - Translation overlay/chat display
- [src/SettingsWindow.xaml.cs](mdc:src/SettingsWindow.xaml.cs) - Application settings interface
- [src/ChatBoxOptionsWindow.xaml.cs](mdc:src/ChatBoxOptionsWindow.xaml.cs) - ChatBox customization options
- [src/ChatBoxSelectorWindow.xaml.cs](mdc:src/ChatBoxSelectorWindow.xaml.cs) - ChatBox region selector
- [src/LogWindow.xaml.cs](mdc:src/LogWindow.xaml.cs) - Log viewer window

### Dialog Windows
- [src/UpdateAvailableDialog.xaml.cs](mdc:src/UpdateAvailableDialog.xaml.cs) - Version update notification
- [src/TtsVoiceSelectorDialog.xaml.cs](mdc:src/TtsVoiceSelectorDialog.xaml.cs) - TTS voice selection
- [src/ServiceInstallDialog.xaml.cs](mdc:src/ServiceInstallDialog.xaml.cs) - Python service installation
- [src/ServiceDiagnosticDialog.xaml.cs](mdc:src/ServiceDiagnosticDialog.xaml.cs) - Service diagnostics
- [src/ServerSetupDialog.xaml.cs](mdc:src/ServerSetupDialog.xaml.cs) - Server setup wizard
- [src/OllamaModelSelectorWindow.xaml.cs](mdc:src/OllamaModelSelectorWindow.xaml.cs) - Ollama model selection
- [src/GoogleVisionSetupDialog.xaml.cs](mdc:src/GoogleVisionSetupDialog.xaml.cs) - Google Vision setup
- [src/ShutdownDialog.xaml.cs](mdc:src/ShutdownDialog.xaml.cs) - Shutdown confirmation
- [src/NoTextInfoDialog.xaml.cs](mdc:src/NoTextInfoDialog.xaml.cs) - No text detected notification

### OCR Services
- [src/WindowsOCRManager.cs](mdc:src/WindowsOCRManager.cs) - Windows built-in OCR integration
- [src/GoogleVisionOCRService.cs](mdc:src/GoogleVisionOCRService.cs) - Google Cloud Vision OCR
- [src/UniversalBlockDetector.cs](mdc:src/UniversalBlockDetector.cs) - Advanced text block detection and grouping
- [src/PythonServicesManager.cs](mdc:src/PythonServicesManager.cs) - Manages Python OCR services (EasyOCR, MangaOCR, PaddleOCR, DocTR)
- [src/PythonService.cs](mdc:src/PythonService.cs) - Individual Python service wrapper

### Translation Services
- [src/ITranslationService.cs](mdc:src/ITranslationService.cs) - Translation service interface
- [src/GeminiTranslationService.cs](mdc:src/GeminiTranslationService.cs) - Google Gemini API integration
- [src/ChatGptTranslationService.cs](mdc:src/ChatGptTranslationService.cs) - OpenAI ChatGPT integration
- [src/OllamaTranslationService.cs](mdc:src/OllamaTranslationService.cs) - Local Ollama LLM integration
- [src/GoogleTranslateService.cs](mdc:src/GoogleTranslateService.cs) - Google Translate integration
- [src/LlamaCppTranslationService.cs](mdc:src/LlamaCppTranslationService.cs) - llama.cpp local translation
- [src/TranslationServiceFactory.cs](mdc:src/TranslationServiceFactory.cs) - Factory for creating translation services

### Audio Services
- [src/GoogleTTSService.cs](mdc:src/GoogleTTSService.cs) - Google Text-to-Speech
- [src/ElevenLabsService.cs](mdc:src/ElevenLabsService.cs) - ElevenLabs TTS integration
- [src/OpenAIRealtimeAudioService.cs](mdc:src/OpenAIRealtimeAudioService.cs) - OpenAI real-time audio transcription
- [src/AudioPlaybackManager.cs](mdc:src/AudioPlaybackManager.cs) - Audio playback management
- [src/AudioPreloadService.cs](mdc:src/AudioPreloadService.cs) - TTS audio preloading

### Utilities and Managers
- [src/HotkeyManager.cs](mdc:src/HotkeyManager.cs) - Global keyboard shortcuts management
- [src/MouseManager.cs](mdc:src/MouseManager.cs) - Mouse input handling
- [src/LogManager.cs](mdc:src/LogManager.cs) - Application logging
- [src/SplashManager.cs](mdc:src/SplashManager.cs) - Splash screen management
- [src/TextObject.cs](mdc:src/TextObject.cs) - Text region data structure
- [src/ErrorPopupManager.cs](mdc:src/ErrorPopupManager.cs) - Error popup management
- [src/GamepadManager.cs](mdc:src/GamepadManager.cs) - Gamepad input management
- [src/WebViewEnvironmentManager.cs](mdc:src/WebViewEnvironmentManager.cs) - WebView2 environment management
- [src/WebViewPool.cs](mdc:src/WebViewPool.cs) - WebView2 instance pooling
- [src/OverlayProfiler.cs](mdc:src/OverlayProfiler.cs) - Overlay performance profiling
- [src/CondaHelper.cs](mdc:src/CondaHelper.cs) - Conda environment management
- [src/ServiceConfigParser.cs](mdc:src/ServiceConfigParser.cs) - Python service config parsing
- [src/ServiceItemViewModel.cs](mdc:src/ServiceItemViewModel.cs) - Service UI view model
- [src/OllamaModelDownloader.cs](mdc:src/OllamaModelDownloader.cs) - Ollama model download management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SethRobinson/UGTLive](https://github.com/SethRobinson/UGTLive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
