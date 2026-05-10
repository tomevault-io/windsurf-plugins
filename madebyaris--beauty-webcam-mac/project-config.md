---
trigger: always_on
description: AppKit UI guidelines and menu bar implementation best practices for BeautyWebcam
---


# AppKit UI Guidelines

## Menu Bar Application Design

### NSStatusItem Implementation
```objc
// BWMenuBarManager.h
@interface BWMenuBarManager : NSObject

@property (nonatomic, strong) NSStatusItem *statusItem;
@property (nonatomic, strong) NSMenu *statusMenu;
@property (nonatomic, weak) id<BWMenuBarDelegate> delegate;

+ (instancetype)sharedManager;
- (void)setupMenuBar;
- (void)updateStatusWithState:(BWApplicationState)state;

@end

// BWMenuBarManager.m
@implementation BWMenuBarManager

+ (instancetype)sharedManager {
    static BWMenuBarManager *sharedManager = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedManager = [[BWMenuBarManager alloc] init];
    });
    return sharedManager;
}

- (void)setupMenuBar {
    // Create status item with variable width
    self.statusItem = [[NSStatusBar systemStatusBar] 
        statusItemWithLength:NSVariableStatusItemLength];
    
    // Configure status item
    self.statusItem.button.image = [self statusImageForState:BWApplicationStateInactive];
    self.statusItem.button.imagePosition = NSImageOnly;
    self.statusItem.button.target = self;
    self.statusItem.button.action = @selector(statusItemClicked:);
    
    // Create menu
    [self setupStatusMenu];
    
    // Set menu (but don't assign it yet - we'll show it manually)
    // This allows us to handle both left and right clicks
}

- (void)setupStatusMenu {
    self.statusMenu = [[NSMenu alloc] init];
    self.statusMenu.delegate = self;
    
    // Enhancement toggle
    NSMenuItem *toggleItem = [[NSMenuItem alloc] 
        initWithTitle:@"Enhancement Off"
        action:@selector(toggleEnhancement:)
        keyEquivalent:@""];
    toggleItem.target = self;
    toggleItem.tag = BWMenuItemTagToggle;
    [self.statusMenu addItem:toggleItem];
    
    [self.statusMenu addItem:[NSMenuItem separatorItem]];
    
    // Preset menu items
    [self addPresetMenuItems];
    
    [self.statusMenu addItem:[NSMenuItem separatorItem]];
    
    // Settings
    NSMenuItem *settingsItem = [[NSMenuItem alloc] 
        initWithTitle:@"Settings..."
        action:@selector(showSettings:)
        keyEquivalent:@","];
    settingsItem.target = self;
    [self.statusMenu addItem:settingsItem];
    
    // Performance monitor (optional)
    NSMenuItem *performanceItem = [[NSMenuItem alloc] 
        initWithTitle:@"Performance Monitor"
        action:@selector(showPerformanceMonitor:)
        keyEquivalent:@""];
    performanceItem.target = self;
    [self.statusMenu addItem:performanceItem];
    
    [self.statusMenu addItem:[NSMenuItem separatorItem]];
    
    // Help and quit
    NSMenuItem *helpItem = [[NSMenuItem alloc] 
        initWithTitle:@"Help & Support"
        action:@selector(showHelp:)
        keyEquivalent:@""];
    helpItem.target = self;
    [self.statusMenu addItem:helpItem];
    
    NSMenuItem *quitItem = [[NSMenuItem alloc] 
        initWithTitle:@"Quit BeautyWebcam"
        action:@selector(quitApplication:)
        keyEquivalent:@"q"];
    quitItem.target = self;
    [self.statusMenu addItem:quitItem];
}

@end
```

### Status Icon Management
```objc
// Dynamic status icon updates
- (NSImage *)statusImageForState:(BWApplicationState)state {
    NSString *imageName;
    
    switch (state) {
        case BWApplicationStateInactive:
            imageName = @"StatusBarIcon_Inactive";
            break;
        case BWApplicationStateActive:
            imageName = @"StatusBarIcon_Active";
            break;
        case BWApplicationStateProcessing:
            imageName = @"StatusBarIcon_Processing";
            break;
        case BWApplicationStateError:
            imageName = @"StatusBarIcon_Error";
            break;
    }
    
    NSImage *image = [NSImage imageNamed:imageName];
    
    // Configure for menu bar appearance
    image.template = YES; // Adapts to menu bar color scheme
    return image;
}

- (void)updateStatusWithState:(BWApplicationState)state {
    NSImage *image = [self statusImageForState:state];
    
    // Update on main thread
    dispatch_async(dispatch_get_main_queue(), ^{
        self.statusItem.button.image = image;
        
        // Update tooltip
        NSString *tooltip = [self tooltipForState:state];
        self.statusItem.button.toolTip = tooltip;
        
        // Update menu items
        [self updateMenuItemsForState:state];
    });
}

- (void)animateProcessingState {
    // Subtle animation for processing state
    if (self.currentState != BWApplicationStateProcessing) return;
    
    NSArray *frames = @[
        [NSImage imageNamed:@"StatusBarIcon_Processing_1"],
        [NSImage imageNamed:@"StatusBarIcon_Processing_2"],
        [NSImage imageNamed:@"StatusBarIcon_Processing_3"]
    ];
    
    static NSInteger frameIndex = 0;
    self.statusItem.button.image = frames[frameIndex % frames.count];
    frameIndex++;
    
    // Schedule next frame
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 0.5 * NSEC_PER_SEC),
                  dispatch_get_main_queue(), ^{
        [self animateProcessingState];
    });
}
```

## Settings Window Design

### Modern Settings Panel
```objc
// BWSettingsWindowController.h
@interface BWSettingsWindowController : NSWindowController


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/beauty-webcam-mac](https://github.com/madebyaris/beauty-webcam-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
