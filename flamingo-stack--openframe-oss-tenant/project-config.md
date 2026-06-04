---
trigger: always_on
description: This document outlines the container and orchestration best practices for the OpenFrame project.
---

# Docker and Kubernetes

This document outlines the container and orchestration best practices for the OpenFrame project.

## Dockerfile Standards

### Base Images

- Use official base images from trusted sources
- Specify exact version tags (not `latest`)
- Use slim or alpine variants where appropriate
- Keep base images consistent across services
- Regularly update base images for security patches

Example Dockerfile:
```dockerfile
# Use specific version of OpenJDK
FROM eclipse-temurin:21-jre-alpine

# Add metadata
LABEL maintainer="OpenFrame Team <team@openframe.com>"
LABEL description="OpenFrame API Service"
LABEL version="1.0.0"

# Set working directory
WORKDIR /app

# Copy application JAR
COPY target/openframe-api.jar /app/

# Set non-root user
RUN addgroup -S openframe && adduser -S openframe -G openframe
USER openframe

# Set entry point
ENTRYPOINT ["java", "-jar", "openframe-api.jar"]
```

### Multi-Stage Builds

- Use multi-stage builds to minimize image size
- Separate build and runtime environments
- Only include necessary artifacts in the final image
- Remove build dependencies from the final image

Example multi-stage build:
```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /build
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /build/target/openframe-api.jar /app/
RUN addgroup -S openframe && adduser -S openframe -G openframe
USER openframe
ENTRYPOINT ["java", "-jar", "openframe-api.jar"]
```

### Layer Optimization

- Order instructions from least to most frequently changing
- Group related commands to minimize layers
- Use .dockerignore to exclude unnecessary files
- Minimize the number of RUN commands
- Clean up in the same layer where files are added

Example layer optimization:
```dockerfile
# Install dependencies in a single layer
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        ca-certificates \
        fontconfig \
        && rm -rf /var/lib/apt/lists/*

# Copy application files
COPY --chown=openframe:openframe . /app/

# Set environment variables
ENV JAVA_OPTS="-Xms512m -Xmx1024m"
ENV SPRING_PROFILES_ACTIVE="docker"
```

### Security Best Practices

- Run containers as non-root users
- Remove unnecessary tools and packages
- Scan images for vulnerabilities
- Use multi-stage builds to minimize attack surface
- Set appropriate file permissions
- Use secrets management for sensitive data

Example security configuration:
```dockerfile
# Create non-root user
RUN addgroup -S openframe && adduser -S openframe -G openframe

# Set appropriate permissions
COPY --chown=openframe:openframe . /app/

# Switch to non-root user
USER openframe

# Use non-privileged port
EXPOSE 8080

# Don't run as root
ENTRYPOINT ["java", "-jar", "openframe-api.jar"]
```

## Docker Compose Configuration

### Service Organization

- Group related services together
- Use consistent naming conventions
- Separate infrastructure and application services
- Use environment variables for configuration
- Define dependencies between services

Example Docker Compose organization:
```yaml
version: '3.8'

# Infrastructure services
services:
  mongodb:
    image: mongo:6.0
    volumes:
      - mongodb-data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_USERNAME}
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_PASSWORD}
    networks:
      - openframe-network

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    networks:
      - openframe-network

  kafka:
    image: confluentinc/cp-kafka:7.3.0
    depends_on:
      - zookeeper
    environment:
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092
    volumes:
      - kafka-data:/var/lib/kafka/data
    networks:
      - openframe-network

  zookeeper:
    image: confluentinc/cp-zookeeper:7.3.0
    volumes:
      - zookeeper-data:/var/lib/zookeeper/data
    environment:
      - ZOOKEEPER_CLIENT_PORT=2181
    networks:
      - openframe-network

# Application services
  openframe-gateway:
    image: openframe/gateway:latest
    build:
      context: ./services/openframe-gateway
    depends_on:
      - openframe-config
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - SPRING_CLOUD_CONFIG_URI=http://openframe-config.microservices.svc.cluster.local:8888
    ports:
      - "8100:8100"
    networks:
      - openframe-network

  openframe-api:
    image: openframe/api:latest
    build:
      context: ./services/openframe-api
    depends_on:
      - openframe-config
      - mongodb
      - kafka
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - SPRING_CLOUD_CONFIG_URI=http://openframe-config.microservices.svc.cluster.local:8888
    networks:
      - openframe-network

volumes:
  mongodb-data:
  redis-data:
  kafka-data:
  zookeeper-data:

networks:
  openframe-network:
    driver: bridge
```

### Environment Configuration

- Use .env files for environment-specific variables
- Don't hardcode sensitive information
- Use consistent variable naming
- Document required environment variables
- Provide default values where appropriate

Example .env file:
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-tenant](https://github.com/flamingo-stack/openframe-oss-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
