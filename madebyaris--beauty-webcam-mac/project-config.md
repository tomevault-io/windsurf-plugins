---
trigger: always_on
description: Metal shader coding standards and optimization guidelines for BeautyWebcam
---


# Metal Shader Guidelines

## Shader Architecture

### File Organization
```metal
// BeautyWebcam_Shaders.metal
#include <metal_stdlib>
using namespace metal;

// Constants
constant float kSkinSmoothingRadius = 2.0;
constant float kColorEnhancementStrength = 0.3;

// Shared utility functions
float luminance(float3 color);
float3 rgb2yuv(float3 rgb);
float3 yuv2rgb(float3 yuv);

// Main processing kernels
kernel void skinSmoothingKernel(...);
kernel void colorEnhancementKernel(...);
kernel void noiseReductionKernel(...);
```

### Naming Conventions
- **Kernels**: Use descriptive names ending with "Kernel" - e.g., `skinSmoothingKernel`
- **Constants**: Use `k` prefix with camelCase - e.g., `kBilateralSigmaD`
- **Structs**: Use `BW` prefix - e.g., `BWProcessingParams`
- **Functions**: Use camelCase - e.g., `calculateBilateralWeight`

## Performance Optimization

### Thread Group Sizing
```metal
// Optimal thread group sizes for different operations
// 16x16 for 2D image processing (256 threads per group)
kernel void imageProcessingKernel(texture2d<float, access::read> inputTexture [[texture(0)]],
                                 texture2d<float, access::write> outputTexture [[texture(1)]],
                                 uint2 gid [[thread_position_in_grid]]) {
    // Process pixel at gid
}

// Use from Objective-C:
// MTLSize threadgroupSize = MTLSizeMake(16, 16, 1);
// MTLSize threadgroupCount = MTLSizeMake((width + 15) / 16, (height + 15) / 16, 1);
```

### Memory Access Patterns
```metal
// Prefer coalesced memory access
kernel void efficientKernel(texture2d<float, access::read> input [[texture(0)]],
                           texture2d<float, access::write> output [[texture(1)]],
                           uint2 gid [[thread_position_in_grid]]) {
    if (gid.x >= input.get_width() || gid.y >= input.get_height()) return;
    
    // Good: Access neighboring pixels in a pattern that maximizes cache hits
    float4 center = input.read(gid);
    float4 right = input.read(gid + uint2(1, 0));
    float4 down = input.read(gid + uint2(0, 1));
    
    // Process and write result
    output.write(processPixels(center, right, down), gid);
}
```

### Threadgroup Memory Usage
```metal
// Use threadgroup memory for shared computations
kernel void bilateralFilterKernel(texture2d<float, access::read> input [[texture(0)]],
                                 texture2d<float, access::write> output [[texture(1)]],
                                 uint2 gid [[thread_position_in_grid]],
                                 uint2 tid [[thread_position_in_threadgroup]],
                                 threadgroup float4 *sharedMemory [[threadgroup(0)]]) {
    
    // Load data into shared memory with border handling
    uint sharedIndex = tid.y * 18 + tid.x; // 18x18 for 16x16 + 1-pixel border
    if (tid.x < 18 && tid.y < 18) {
        uint2 loadPos = gid + tid - uint2(1, 1); // Offset for border
        sharedMemory[sharedIndex] = input.read(loadPos);
    }
    
    threadgroup_barrier(mem_flags::mem_threadgroup);
    
    // Use shared data for filtering
    if (tid.x < 16 && tid.y < 16) {
        float4 result = bilateralFilter(sharedMemory, tid);
        output.write(result, gid);
    }
}
```

## Image Processing Algorithms

### Bilateral Filter Implementation
```metal
// Optimized bilateral filter for skin smoothing
float bilateralWeight(float2 spatialDistance, float colorDistance, 
                     float sigmaD, float sigmaR) {
    float spatialWeight = exp(-dot(spatialDistance, spatialDistance) / (2.0 * sigmaD * sigmaD));
    float colorWeight = exp(-(colorDistance * colorDistance) / (2.0 * sigmaR * sigmaR));
    return spatialWeight * colorWeight;
}

kernel void skinSmoothingKernel(texture2d<float, access::read> input [[texture(0)]],
                               texture2d<float, access::write> output [[texture(1)]],
                               constant BWProcessingParams &params [[buffer(0)]],
                               uint2 gid [[thread_position_in_grid]]) {
    
    if (gid.x >= input.get_width() || gid.y >= input.get_height()) return;
    
    float4 centerPixel = input.read(gid);
    float3 centerColor = centerPixel.rgb;
    
    float3 filteredColor = float3(0.0);
    float totalWeight = 0.0;
    
    // Sample in kernel radius
    int radius = int(params.smoothingRadius);
    for (int dy = -radius; dy <= radius; dy++) {
        for (int dx = -radius; dx <= radius; dx++) {
            uint2 samplePos = uint2(max(0, min(int(input.get_width()) - 1, int(gid.x) + dx)),
                                   max(0, min(int(input.get_height()) - 1, int(gid.y) + dy)));
            
            float4 samplePixel = input.read(samplePos);
            float3 sampleColor = samplePixel.rgb;
            
            float colorDist = distance(centerColor, sampleColor);
            float2 spatialDist = float2(dx, dy);
            
            float weight = bilateralWeight(spatialDist, colorDist, 
                                         params.sigmaD, params.sigmaR);
            
            filteredColor += weight * sampleColor;
            totalWeight += weight;
        }
    }
    
    filteredColor /= totalWeight;
    
    // Blend with original based on intensity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/beauty-webcam-mac](https://github.com/madebyaris/beauty-webcam-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
