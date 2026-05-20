---
trigger: always_on
description: UI/UX patterns and window management guide
---


# UI/UX Patterns and Window Management Guide

## Window Architecture

### Window Types
UGTLive uses several specialized windows, each with specific responsibilities:

1. **Main Window** ([src/MainWindow.xaml](mdc:src/MainWindow.xaml))
   - Central control panel
   - Start/Stop translation
   - Access to all other windows
   - System tray integration

2. **Monitor Window** ([src/MonitorWindow.xaml](mdc:src/MonitorWindow.xaml))
   - Live preview of capture area
   - Visual OCR feedback
   - Resizable and movable

3. **ChatBox Window** ([src/ChatBoxWindow.xaml](mdc:src/ChatBoxWindow.xaml))
   - Translation display overlay
   - Always-on-top option
   - Customizable appearance

4. **Settings Window** ([src/SettingsWindow.xaml](mdc:src/SettingsWindow.xaml))
   - Tabbed interface for all settings
   - API key management
   - Service configuration
   - OCR and translation settings

5. **Log Window** ([src/LogWindow.xaml](mdc:src/LogWindow.xaml))
   - Application log viewer
   - Real-time log updates
   - Filtering and search

## Keyboard Shortcuts ([src/HotkeyManager.cs](mdc:src/HotkeyManager.cs))

### Global Shortcuts
All shortcuts are managed through HotkeyManager and configurable via hotkeys.txt:

| Action | Default | Event |
|--------|---------|-------|
| Start/Stop OCR | Shift+S | `StartStopRequested` |
| Show/Hide Monitor | Shift+M | `MonitorToggleRequested` |
| Show/Hide ChatBox | Shift+C | `ChatBoxToggleRequested` |
| Show/Hide Settings | Shift+P | `SettingsToggleRequested` |
| Show/Hide Log | Shift+L | `LogToggleRequested` |
| Show/Hide Main Window | Shift+H | `MainWindowVisibilityToggleRequested` |
| Clear Overlays | Shift+X | `ClearOverlaysRequested` |
| Passthrough Toggle | Shift+T | `PassthroughToggleRequested` |
| Overlay Mode Toggle | Shift+O | `OverlayModeToggleRequested` |
| Listen Toggle | Shift+A | `ListenToggleRequested` |
| View in Browser | Shift+B | `ViewInBrowserRequested` |
| Play All Audio | Shift+W | `PlayAllAudioRequested` |

### Implementation Pattern
```csharp
// Register hotkey events
HotkeyManager.Instance.StartStopRequested += OnStartStopRequested;
HotkeyManager.Instance.MonitorToggleRequested += OnMonitorToggleRequested;

// Hotkeys are loaded from hotkeys.txt and can be customized
```

### Hotkey Configuration
Hotkeys are stored in `hotkeys.txt` with format:
```
action=key1+key2+key3
```

## UI Design Patterns

### Window Lifecycle
1. **Creation**: Windows created once at startup
2. **Visibility**: Show/Hide instead of Create/Destroy
3. **State Persistence**: Position and size saved via ConfigManager
4. **Cleanup**: Proper disposal on application exit

### Common Window Properties
```xml
<!-- Standard window attributes -->
WindowStyle="None"
AllowsTransparency="True"
ResizeMode="CanResize"
Topmost="{Binding IsAlwaysOnTop}"
```

### Styling Conventions

#### Colors and Themes
- Dark theme by default
- Configurable accent colors
- Semi-transparent backgrounds
- High contrast text

#### Standard Controls
```xml
<!-- Button Style -->
<Button Style="{StaticResource ModernButton}"
        Background="#FF2D2D30"
        Foreground="White"
        BorderThickness="0"/>

<!-- TextBox Style -->
<TextBox Style="{StaticResource ModernTextBox}"
         Background="#FF3F3F46"
         Foreground="White"
         BorderBrush="#FF007ACC"/>
```

## ChatBox Customization

### Appearance Settings ([src/ChatBoxOptionsWindow.xaml](mdc:src/ChatBoxOptionsWindow.xaml))
- Background color picker
- Text color picker
- Transparency slider (0-100%)
- Font family and size
- Border options

### Layout Options
- Auto-size to content
- Maximum width/height
- Text alignment
- Padding configuration

## Settings Organization

### Tab Structure in Settings Window
1. **General**: Basic app settings, hotkeys
2. **OCR**: OCR service selection and config
3. **Translation**: Service selection and API keys
4. **ChatBox**: Display preferences
5. **Audio**: TTS settings and voice selection
6. **Advanced**: Debug and experimental features

### Settings Pattern
```csharp
// Property in ConfigManager
public string GetSetting() => GetValue(SETTING_KEY, defaultValue);
public void SetSetting(string value)
{
    SetValue(SETTING_KEY, value);
    SaveConfig();
}

// UI Binding in XAML
<TextBox Text="{Binding SettingName, Mode=TwoWay}"/>
```

## Mouse Interaction ([src/MouseManager.cs](mdc:src/MouseManager.cs))

### Region Selection
- Click and drag to select area
- Visual feedback during selection
- Escape key to cancel
- Double-click to confirm

### Window Dragging
- Custom title bar implementation
- Drag from any empty area
- Snap to screen edges

## Notification Patterns

### Status Messages
- Displayed in main window status bar
- Auto-fade after 3 seconds
- Color-coded by type (info, warning, error)

### Error Handling UI
```csharp
// Show error via ErrorPopupManager
ErrorPopupManager.ShowError("Error Title", "Error message");

// Or show in UI
Application.Current.Dispatcher.Invoke(() =>
{
    StatusText.Text = $"Error: {message}";
    StatusText.Foreground = Brushes.Red;
});
```

## Accessibility Considerations

### Keyboard Navigation
- Tab order properly set
- All functions keyboard accessible
- Tooltips for all controls
- Keyboard shortcuts documented

### Visual Accessibility
- High contrast mode support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SethRobinson/UGTLive](https://github.com/SethRobinson/UGTLive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
