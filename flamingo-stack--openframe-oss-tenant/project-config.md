---
trigger: always_on
description: This document outlines the monitoring and observability best practices for the OpenFrame project.
---

# Monitoring and Observability

This document outlines the monitoring and observability best practices for the OpenFrame project.

## Monitoring Stack

OpenFrame uses a comprehensive monitoring stack:

- **Prometheus**: Metrics collection and storage
- **Grafana**: Visualization and dashboards
- **Loki**: Log aggregation
- **Tempo**: Distributed tracing
- **Alertmanager**: Alert management and notifications

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│             │     │             │     │             │
│  Prometheus │     │    Loki     │     │   Tempo     │
│             │     │             │     │             │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │
       │                   │                   │
       ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────┐
│                                                     │
│                      Grafana                        │
│                                                     │
└─────────────────────────────────────────────────────┘
                          │
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│                                                     │
│                   Alertmanager                      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## Metrics Collection

### Application Metrics

Spring Boot applications expose metrics through Micrometer and Actuator:

```java
@Configuration
public class MetricsConfig {
    @Bean
    public MeterRegistryCustomizer<MeterRegistry> metricsCommonTags(
            @Value("${spring.application.name}") String applicationName) {
        return registry -> registry.config()
            .commonTags("application", applicationName);
    }
    
    @Bean
    public TimedAspect timedAspect(MeterRegistry registry) {
        return new TimedAspect(registry);
    }
}
```

Example usage in code:
```java
@Service
public class DeviceService {
    private final MeterRegistry meterRegistry;
    private final DeviceRepository deviceRepository;
    
    @Timed(value = "device.service.get.time", description = "Time taken to get device")
    public Mono<Device> getDeviceById(String id) {
        return deviceRepository.findById(id)
            .doOnSuccess(device -> {
                if (device != null) {
                    meterRegistry.counter("device.service.get.success").increment();
                } else {
                    meterRegistry.counter("device.service.get.notfound").increment();
                }
            })
            .doOnError(e -> meterRegistry.counter("device.service.get.error").increment());
    }
    
    @Timed(value = "device.service.create.time", description = "Time taken to create device")
    public Mono<Device> createDevice(Device device) {
        return deviceRepository.save(device)
            .doOnSuccess(d -> meterRegistry.counter("device.service.create.success").increment())
            .doOnError(e -> meterRegistry.counter("device.service.create.error").increment());
    }
}
```

### Custom Metrics

Define custom metrics for business-specific monitoring:

```java
@Component
public class DeviceMetrics {
    private final MeterRegistry meterRegistry;
    
    @Scheduled(fixedRate = 60000)
    public void recordDeviceMetrics() {
        // Record device count by status
        Map<String, Long> deviceCountByStatus = deviceRepository.countByStatus().block();
        deviceCountByStatus.forEach((status, count) -> {
            meterRegistry.gauge("device.count.by.status", 
                Tags.of("status", status), count);
        });
        
        // Record device count by OS
        Map<String, Long> deviceCountByOs = deviceRepository.countByOperatingSystem().block();
        deviceCountByOs.forEach((os, count) -> {
            meterRegistry.gauge("device.count.by.os", 
                Tags.of("os", os), count);
        });
    }
}
```

### Prometheus Configuration

Configure Prometheus to scrape metrics from OpenFrame services:

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'openframe-api'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['openframe-api:8080']
    
  - job_name: 'openframe-gateway'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['openframe-gateway:8100']
    
  - job_name: 'openframe-management'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['openframe-management:8081']
    
  # Additional services...
```

## Logging

### Structured Logging

Use structured logging with JSON format:

```xml
<!-- logback-spring.xml -->
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeMdc>true</includeMdc>
            <includeContext>true</includeContext>
            <customFields>{"application":"${spring.application.name}"}</customFields>
        </encoder>
    </appender>
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-tenant](https://github.com/flamingo-stack/openframe-oss-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
