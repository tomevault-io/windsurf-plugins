---
trigger: always_on
description: This document outlines the messaging and event-driven architecture patterns used in the OpenFrame project.
---

# Messaging Patterns

This document outlines the messaging and event-driven architecture patterns used in the OpenFrame project.

## Kafka Usage Patterns

### Topic Naming Conventions

- Use kebab-case for topic names
- Follow the pattern `{domain}-{entity}-{action}`
- Use plural for entity collections
- Use past tense for actions

Examples:
```
devices-created
devices-updated
devices-deleted
alerts-triggered
metrics-collected
```

### Producer Patterns

- Use the KafkaTemplate for producing messages
- Set appropriate key for message partitioning
- Include metadata in message headers
- Handle production errors gracefully
- Use asynchronous sending for non-blocking operations

Example producer:
```java
@Service
public class DeviceEventProducer {
    private final KafkaTemplate<String, DeviceEvent> kafkaTemplate;
    
    public CompletableFuture<SendResult<String, DeviceEvent>> publishDeviceCreated(Device device) {
        DeviceEvent event = new DeviceEvent(
            UUID.randomUUID().toString(),
            "DEVICE_CREATED",
            LocalDateTime.now(),
            device
        );
        
        return kafkaTemplate.send("devices-created", device.getId(), event)
            .handle((result, ex) -> {
                if (ex != null) {
                    log.error("Failed to send device created event: {}", ex.getMessage());
                    throw new EventPublishingException("Failed to publish device created event", ex);
                }
                return result;
            });
    }
}
```

### Consumer Patterns

- Use the @KafkaListener annotation for consuming messages
- Group consumers by functionality
- Handle deserialization errors
- Implement idempotent processing
- Use manual acknowledgment for critical processing
- Implement proper error handling and retries

Example consumer:
```java
@Service
public class DeviceEventConsumer {
    private final DeviceService deviceService;
    
    @KafkaListener(
        topics = "devices-created",
        groupId = "device-processor",
        containerFactory = "kafkaListenerContainerFactory"
    )
    public void consumeDeviceCreatedEvent(
            @Payload DeviceEvent event,
            @Header(KafkaHeaders.RECEIVED_KEY) String key,
            @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
            @Header(KafkaHeaders.OFFSET) long offset,
            Acknowledgment acknowledgment) {
        
        try {
            log.info("Processing device created event: {}, partition: {}, offset: {}", 
                    event.getId(), partition, offset);
            
            deviceService.processDeviceCreated(event.getDevice());
            
            acknowledgment.acknowledge();
        } catch (Exception e) {
            log.error("Error processing device created event: {}", e.getMessage());
            // Implement retry or dead letter queue logic
        }
    }
}
```

### Serialization

- Use JSON for message serialization
- Define clear schema for each message type
- Include version information in the schema
- Handle schema evolution gracefully
- Consider using Avro for complex schemas

Example serialization configuration:
```java
@Configuration
public class KafkaConfig {
    @Bean
    public KafkaTemplate<String, Object> kafkaTemplate(
            ProducerFactory<String, Object> producerFactory) {
        return new KafkaTemplate<>(producerFactory);
    }
    
    @Bean
    public ProducerFactory<String, Object> producerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        return new DefaultKafkaProducerFactory<>(configProps);
    }
    
    @Bean
    public ConsumerFactory<String, Object> consumerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        configProps.put(JsonDeserializer.TRUSTED_PACKAGES, "com.openframe.*");
        return new DefaultKafkaConsumerFactory<>(configProps);
    }
    
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Object> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, Object> factory =
            new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL_IMMEDIATE);
        return factory;
    }
}
```

## Event Schema Design

### Event Structure

- Use a consistent structure for all events
- Include common metadata fields
- Use strong typing for event payloads
- Version your event schemas

Example event structure:
```java
@Data
public class Event<T> {
    private String id;
    private String type;
    private String source;
    private LocalDateTime timestamp;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-tenant](https://github.com/flamingo-stack/openframe-oss-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
