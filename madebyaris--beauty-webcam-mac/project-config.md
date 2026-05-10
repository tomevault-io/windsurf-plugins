---
trigger: always_on
description: CoreMediaIO virtual camera implementation guidelines and best practices
---


# CoreMediaIO Virtual Camera Guidelines

## Overview
CoreMediaIO is the framework for creating virtual camera devices on macOS. This guide covers best practices for implementing a virtual camera that appears as a standard webcam to all applications.

## Plugin Architecture

### Basic Plugin Structure
```objc
// BWVirtualCameraPlugin.h
@interface BWVirtualCameraPlugin : NSObject

@property (nonatomic, strong, readonly) NSUUID *pluginUUID;
@property (nonatomic, strong, readonly) NSString *pluginName;

+ (instancetype)sharedInstance;
- (BOOL)initializePluginWithError:(NSError **)error;
- (void)teardownPlugin;

@end

// Plugin registration
- (BOOL)initializePluginWithError:(NSError **)error {
    // Register with CoreMediaIO
    CMIOObjectPropertyAddress propertyAddress = {
        kCMIOHardwarePropertyAllowScreenCaptureDevices,
        kCMIOObjectPropertyScopeGlobal,
        kCMIOObjectPropertyElementMaster
    };
    
    UInt32 allow = 1;
    OSStatus status = CMIOObjectSetPropertyData(kCMIOObjectSystemObject,
                                               &propertyAddress,
                                               0, NULL,
                                               sizeof(allow), &allow);
    
    return status == noErr;
}
```

### Device Implementation
```objc
// BWVirtualCameraDevice.h
@interface BWVirtualCameraDevice : NSObject

@property (nonatomic, strong, readonly) NSString *deviceName;
@property (nonatomic, strong, readonly) NSString *deviceID;
@property (nonatomic, assign, readonly) CMIODeviceID deviceID;
@property (nonatomic, assign, getter=isRunning) BOOL running;

- (BOOL)createDeviceWithError:(NSError **)error;
- (BOOL)startStreamingWithError:(NSError **)error;
- (void)stopStreaming;
- (void)destroyDevice;

- (void)sendVideoFrame:(CVPixelBufferRef)pixelBuffer
             timestamp:(CMTime)timestamp;

@end
```

### Stream Management
```objc
// BWVirtualCameraStream.m
@implementation BWVirtualCameraStream

- (BOOL)startStreamingWithFormat:(CMVideoFormatDescriptionRef)formatDescription
                           error:(NSError **)error {
    
    // Validate format
    if (!formatDescription) {
        if (error) {
            *error = [NSError errorWithDomain:BWErrorDomain
                                         code:BWErrorInvalidFormat
                                     userInfo:@{NSLocalizedDescriptionKey: @"Invalid format description"}];
        }
        return NO;
    }
    
    // Set up timing information
    self.frameRate = 30; // Default to 30fps
    self.frameDuration = CMTimeMake(1, self.frameRate);
    
    // Initialize stream state
    self.isStreaming = YES;
    self.frameCount = 0;
    
    return YES;
}

- (void)sendFrame:(CVPixelBufferRef)pixelBuffer {
    if (!self.isStreaming) return;
    
    // Calculate timestamp
    CMTime timestamp = CMTimeMultiply(self.frameDuration, self.frameCount);
    
    // Create sample buffer
    CMSampleBufferRef sampleBuffer = NULL;
    OSStatus status = CMSampleBufferCreateForImageBuffer(
        kCFAllocatorDefault,
        pixelBuffer,
        true, NULL, NULL,
        self.formatDescription,
        &self.sampleTiming,
        &sampleBuffer
    );
    
    if (status == noErr && sampleBuffer) {
        // Send to CoreMediaIO
        [self enqueueSampleBuffer:sampleBuffer];
        CFRelease(sampleBuffer);
    }
    
    self.frameCount++;
}

@end
```

## Property Management

### Device Properties
```objc
// Essential device properties
static const CMIOObjectPropertyAddress kDevicePropertyName = {
    kCMIOObjectPropertyName,
    kCMIOObjectPropertyScopeGlobal,
    kCMIOObjectPropertyElementMaster
};

static const CMIOObjectPropertyAddress kDevicePropertyUID = {
    kCMIODevicePropertyDeviceUID,
    kCMIOObjectPropertyScopeGlobal,
    kCMIOObjectPropertyElementMaster
};

// Property getter implementation
- (OSStatus)getPropertyWithAddress:(const CMIOObjectPropertyAddress *)address
                         dataSize:(UInt32)dataSize
                         dataUsed:(UInt32 *)dataUsed
                             data:(void *)data {
    
    OSStatus result = kCMIOHardwareNoError;
    
    switch (address->mSelector) {
        case kCMIOObjectPropertyName:
            *dataUsed = [self copyStringProperty:@"BeautyWebcam Virtual Camera"
                                          toData:data
                                        dataSize:dataSize];
            break;
            
        case kCMIODevicePropertyDeviceUID:
            *dataUsed = [self copyStringProperty:self.deviceUID
                                          toData:data
                                        dataSize:dataSize];
            break;
            
        case kCMIODevicePropertyStreams:
            *dataUsed = [self copyStreamsProperty:data dataSize:dataSize];
            break;
            
        default:
            result = kCMIOHardwareUnknownPropertyError;
            break;
    }
    
    return result;
}
```

### Stream Properties
```objc
// Stream format management
- (OSStatus)setFormatDescription:(CMVideoFormatDescriptionRef)formatDescription {
    // Validate format
    CMVideoDimensions dimensions = CMVideoFormatDescriptionGetDimensions(formatDescription);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/beauty-webcam-mac](https://github.com/madebyaris/beauty-webcam-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
