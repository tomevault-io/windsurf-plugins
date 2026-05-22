---
trigger: always_on
description: This document outlines the testing standards and best practices for the OpenFrame project.
---

# Testing Standards

This document outlines the testing standards and best practices for the OpenFrame project.

## Testing Pyramid

OpenFrame follows the testing pyramid approach:

```
    /\
   /  \
  /    \
 / E2E  \
/--------\
/ Integration \
/----------------\
/     Unit Tests    \
/----------------------\
```

- **Unit Tests**: Test individual components in isolation
- **Integration Tests**: Test interactions between components
- **End-to-End Tests**: Test complete user flows

## Unit Testing

### Java Unit Tests

Java unit tests use JUnit 5 and Mockito:

```java
@ExtendWith(MockitoExtension.class)
public class DeviceServiceTest {
    @Mock
    private DeviceRepository deviceRepository;
    
    @InjectMocks
    private DeviceService deviceService;
    
    @Test
    public void testGetDeviceById() {
        // Arrange
        String deviceId = "device-123";
        Device expectedDevice = new Device();
        expectedDevice.setId(deviceId);
        expectedDevice.setHostname("test-device");
        
        when(deviceRepository.findById(deviceId)).thenReturn(Mono.just(expectedDevice));
        
        // Act
        Mono<Device> result = deviceService.getDeviceById(deviceId);
        
        // Assert
        StepVerifier.create(result)
            .expectNext(expectedDevice)
            .verifyComplete();
        
        verify(deviceRepository).findById(deviceId);
    }
}
```

### Vue.js Unit Tests

Vue.js unit tests use Vitest and Vue Test Utils:

```typescript
import { describe, it, expect, vi } from 'vitest';
import { mount } from '@vue/test-utils';
import DeviceList from '@/components/DeviceList.vue';

describe('DeviceList', () => {
  it('renders devices correctly', async () => {
    // Arrange
    const devices = [
      { id: '1', hostname: 'device-1', status: 'online' },
      { id: '2', hostname: 'device-2', status: 'offline' }
    ];
    
    // Act
    const wrapper = mount(DeviceList, {
      props: {
        devices
      }
    });
    
    // Assert
    expect(wrapper.findAll('tr').length).toBe(devices.length + 1); // +1 for header row
    expect(wrapper.text()).toContain('device-1');
    expect(wrapper.text()).toContain('device-2');
  });
  
  it('emits select event when device is clicked', async () => {
    // Arrange
    const devices = [
      { id: '1', hostname: 'device-1', status: 'online' }
    ];
    
    const wrapper = mount(DeviceList, {
      props: {
        devices
      }
    });
    
    // Act
    await wrapper.find('tr:nth-child(2)').trigger('click');
    
    // Assert
    expect(wrapper.emitted().select).toBeTruthy();
    expect(wrapper.emitted().select[0]).toEqual([devices[0]]);
  });
});
```

## Integration Testing

### Spring Boot Integration Tests

Spring Boot integration tests use `@SpringBootTest`:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class DeviceControllerIntegrationTest {
    @Autowired
    private WebTestClient webTestClient;
    
    @Autowired
    private DeviceRepository deviceRepository;
    
    @BeforeEach
    public void setup() {
        deviceRepository.deleteAll().block();
    }
    
    @Test
    public void testGetDevices() {
        // Arrange
        Device device1 = new Device();
        device1.setHostname("device-1");
        device1.setStatus("online");
        
        Device device2 = new Device();
        device2.setHostname("device-2");
        device2.setStatus("offline");
        
        deviceRepository.saveAll(Arrays.asList(device1, device2)).blockLast();
        
        // Act & Assert
        webTestClient.get()
            .uri("/api/devices")
            .exchange()
            .expectStatus().isOk()
            .expectBodyList(Device.class)
            .hasSize(2)
            .contains(device1, device2);
    }
}
```

### API Integration Tests

API integration tests use WebTestClient:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class ApiIntegrationTest {
    @Autowired
    private WebTestClient webTestClient;
    
    @Test
    public void testCreateDevice() {
        // Arrange
        DeviceRequest request = new DeviceRequest();
        request.setHostname("test-device");
        request.setOperatingSystem("Linux");
        
        // Act & Assert
        webTestClient.post()
            .uri("/api/devices")
            .contentType(MediaType.APPLICATION_JSON)
            .bodyValue(request)
            .exchange()
            .expectStatus().isCreated()
            .expectBody()
            .jsonPath("$.id").isNotEmpty()
            .jsonPath("$.hostname").isEqualTo("test-device")
            .jsonPath("$.operatingSystem").isEqualTo("Linux");
    }
}
```

## End-to-End Testing

### Cypress Tests

End-to-end tests use Cypress:

```javascript
describe('Device Management', () => {
  beforeEach(() => {
    cy.login('admin', 'password');
    cy.visit('/devices');
  });
  
  it('should display device list', () => {
    cy.get('table').should('be.visible');
    cy.get('tr').should('have.length.greaterThan', 1);
  });
  
  it('should navigate to device details', () => {
    cy.get('tr').eq(1).click();
    cy.url().should('include', '/devices/');

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-tenant](https://github.com/flamingo-stack/openframe-oss-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
