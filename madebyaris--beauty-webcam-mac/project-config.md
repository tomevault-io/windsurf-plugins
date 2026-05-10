---
trigger: always_on
description: Performance optimization guidelines for BeautyWebcam video processing
---


# Performance Optimization Guidelines

## Target Performance Metrics
- **CPU Usage**: <15% on M1 MacBook Air during active processing
- **Memory Usage**: <150MB total application footprint
- **GPU Usage**: <30% Metal compute utilization
- **Latency**: <50ms from capture to virtual camera output
- **Frame Rate**: Consistent 30fps minimum, 60fps target

## Memory Management for Performance

### Buffer Management
```objc
// Use CVPixelBufferPool for efficient buffer reuse
@property (nonatomic, strong) CVPixelBufferPoolRef pixelBufferPool;

- (void)setupPixelBufferPool {
    NSDictionary *attributes = @{
        (NSString *)kCVPixelBufferPixelFormatTypeKey: @(kCVPixelFormatType_32BGRA),
        (NSString *)kCVPixelBufferWidthKey: @(1920),
        (NSString *)kCVPixelBufferHeightKey: @(1080),
        (NSString *)kCVPixelBufferMetalCompatibilityKey: @YES
    };
    
    CVPixelBufferPoolCreate(kCFAllocatorDefault, NULL, 
                           (__bridge CFDictionaryRef)attributes, 
                           &_pixelBufferPool);
}
```

### Autorelease Pool Usage
```objc
// Critical: Use autorelease pools in processing loops
- (void)processVideoFrames {
    while (self.isProcessing) {
        @autoreleasepool {
            CVPixelBufferRef frame = [self captureNextFrame];
            [self processFrame:frame];
            CVPixelBufferRelease(frame);
        }
    }
}
```

### Memory Pool Patterns
```objc
// Implement object pooling for frequently created objects
@interface BWObjectPool : NSObject
- (id)borrowObject;
- (void)returnObject:(id)object;
@end

// Use for expensive-to-create processing objects
@property (nonatomic, strong) BWObjectPool *processorPool;
```

## GPU Acceleration with Metal

### Compute Shader Best Practices
```objc
// Optimal threadgroup sizes for different operations
static const MTLSize kSkinSmoothingThreadgroupSize = {16, 16, 1};
static const MTLSize kColorCorrectionThreadgroupSize = {32, 8, 1};

// Use appropriate texture formats
MTLTextureDescriptor *descriptor = [MTLTextureDescriptor 
    texture2DDescriptorWithPixelFormat:MTLPixelFormatBGRA8Unorm
                                 width:width
                                height:height
                             mipmapped:NO];
descriptor.usage = MTLTextureUsageShaderRead | MTLTextureUsageShaderWrite;
```

### Command Buffer Optimization
```objc
// Batch operations into single command buffer
- (void)processFrameWithMetal:(CVPixelBufferRef)pixelBuffer {
    id<MTLCommandBuffer> commandBuffer = [self.commandQueue commandBuffer];
    
    // Batch multiple compute operations
    [self addSkinSmoothingToCommandBuffer:commandBuffer];
    [self addColorCorrectionToCommandBuffer:commandBuffer];
    [self addNoiseReductionToCommandBuffer:commandBuffer];
    
    [commandBuffer commit];
    [commandBuffer waitUntilCompleted];
}
```

## Core Image Optimization

### Filter Chain Optimization
```objc
// Chain filters efficiently to minimize intermediate textures
CIFilter *filter1 = [CIFilter filterWithName:@"CIBilateralFilter"];
CIFilter *filter2 = [CIFilter filterWithName:@"CIColorControls"];

// Connect filters directly instead of using intermediate images
filter2.inputImage = filter1.outputImage;
```

### Context Reuse
```objc
// Reuse CIContext instances - they're expensive to create
@property (nonatomic, strong) CIContext *metalContext;

- (void)setupCoreImageContext {
    self.metalContext = [CIContext contextWithMTLDevice:self.metalDevice
                                                options:@{
        kCIContextWorkingColorSpace: [NSNull null],
        kCIContextUseSoftwareRenderer: @NO
    }];
}
```

## Threading and Concurrency

### Processing Pipeline Threading
```objc
// Separate queues for different pipeline stages
@property (nonatomic, strong) dispatch_queue_t captureQueue;
@property (nonatomic, strong) dispatch_queue_t processingQueue;
@property (nonatomic, strong) dispatch_queue_t outputQueue;

- (void)setupProcessingPipeline {
    // High priority for capture to prevent frame drops
    dispatch_queue_attr_t captureAttr = dispatch_queue_attr_make_with_qos_class(
        DISPATCH_QUEUE_SERIAL, QOS_CLASS_USER_INTERACTIVE, 0);
    self.captureQueue = dispatch_queue_create("capture", captureAttr);
    
    // Background processing queue
    dispatch_queue_attr_t processAttr = dispatch_queue_attr_make_with_qos_class(
        DISPATCH_QUEUE_CONCURRENT, QOS_CLASS_DEFAULT, 0);
    self.processingQueue = dispatch_queue_create("processing", processAttr);
}
```

### Lock-Free Programming
```objc
// Use atomic operations for simple state management
@property (atomic, assign) BOOL isProcessing;
@property (atomic, assign) NSInteger frameCount;

// Use dispatch_semaphore for resource limiting
@property (nonatomic, strong) dispatch_semaphore_t frameSemaphore;

- (void)processFrame:(CVPixelBufferRef)frame {
    // Limit concurrent processing
    dispatch_semaphore_wait(self.frameSemaphore, DISPATCH_TIME_FOREVER);
    
    dispatch_async(self.processingQueue, ^{
        // Process frame
        [self enhanceFrame:frame];
        dispatch_semaphore_signal(self.frameSemaphore);
    });
}
```

## Algorithm Optimization

### Bilateral Filter Implementation
```objc
// Use separable filters when possible

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/beauty-webcam-mac](https://github.com/madebyaris/beauty-webcam-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
