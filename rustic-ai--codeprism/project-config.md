---
trigger: always_on
description: This rule file provides comprehensive guidance on Docker best practices, covering Dockerfile construction, image optimization, and security considerations. It aims to improve the efficiency, maintainability, and security of Docker-based projects.
---

# Docker Best Practices

This document provides comprehensive guidance on Docker best practices, covering Dockerfile construction, image optimization, security considerations, and more. It aims to improve the efficiency, maintainability, and security of Docker-based projects.

## 1. Code Organization and Structure

- **Directory Structure Best Practices:**
    - Organize your project with a clear separation of concerns.  For example:
        
        project-root/
        ├── Dockerfile            # Dockerfile for building the image
        ├── docker-compose.yml    # Docker Compose file for multi-container setup
        ├── .dockerignore         # Specifies intentionally untracked files that Docker should ignore
        ├── app/                  # Application source code
        │   ├── ...
        ├── config/               # Configuration files
        │   ├── ...
        ├── data/                 # Data files (if any, though consider volumes)
        │   ├── ...
        ├── scripts/              # Scripts for building, deploying, or managing the container
        │   ├── ...
        
    - Keep the `Dockerfile` and `docker-compose.yml` at the root of your project for easy access.

- **File Naming Conventions:**
    - Use descriptive names for your Dockerfiles (e.g., `Dockerfile.web`, `Dockerfile.api`).
    - Follow a consistent naming convention for all files and directories.

- **Module Organization:**
    - Structure your application into modular components to improve reusability and maintainability. This directly affects what goes into a docker image.
    - Use appropriate build tools (e.g., Maven, Gradle, npm) to manage dependencies and package your application.

- **Component Architecture:**
    - Design your application as a set of microservices or components, each running in its own container, when appropriate.
    - Use Docker Compose to orchestrate multi-container applications.

- **Code Splitting Strategies:**
    - Break down large applications into smaller, more manageable parts to reduce image size and improve build times.
    - Consider multi-stage builds to include build-time dependencies in one stage and only the runtime dependencies in the final image.

## 2. Common Patterns and Anti-patterns

- **Design Patterns Specific to Docker:**
    - **Sidecar Pattern:** Run a utility container alongside your main application container (e.g., for logging, monitoring).
    - **Ambassador Pattern:** Proxy requests to a service running outside the container.
    - **Adapter Pattern:** Adapt the interface of a service to match the expected interface of a client.
    - **Init Container Pattern:** Run initialization tasks before the main application container starts.  Often used to set up configuration, prepare databases, etc.

- **Recommended Approaches for Common Tasks:**
    - **Configuration Management:** Use environment variables to configure your application.
    - **Logging:** Centralize logging using a logging driver or a dedicated logging container (e.g., Fluentd, Logstash).
    - **Health Checks:** Implement health checks to ensure that your services are running correctly.
    - **Process Management:** Use a process manager (e.g., `tini`, `dumb-init`) to handle signal forwarding and zombie process reaping.

- **Anti-patterns and Code Smells to Avoid:**
    - **Storing secrets in Dockerfile or images:** Never hardcode passwords or API keys in your Dockerfile.
    - **Running services as root:** Avoid running your application as the root user.
    - **Installing unnecessary packages:** Keep your images lean by only installing the required dependencies.
    - **Ignoring `.dockerignore`:** Make sure to use `.dockerignore` to exclude unnecessary files from the build context, reducing image size and build time.
    - **Using `ADD` instead of `COPY` unnecessarily:** `COPY` is usually more transparent and predictable.

- **State Management Best Practices:**
    - **Stateless Applications:** Design your application to be stateless whenever possible.
    - **Volumes:** Use volumes for persistent storage (e.g., databases, logs).
    - **Bind Mounts:** Use bind mounts for development to allow code changes to be reflected immediately in the container.

- **Error Handling Patterns:**
    - Implement robust error handling in your application.
    - Use appropriate logging levels to capture errors and warnings.
    - Implement retry mechanisms for transient errors.
    - Monitor your application for errors and take corrective actions.

## 3. Performance Considerations

- **Optimization Techniques:**
    - **Multi-stage builds:** Use multi-stage builds to create smaller, more efficient images.
    - **Minimize layers:** Combine multiple commands into a single layer using `&&`.
    - **Use a lightweight base image:** Choose a minimal base image like Alpine Linux.
    - **Optimize caching:** Order your Dockerfile commands to maximize cache reuse.

- **Memory Management:**
    - Set memory limits for your containers to prevent them from consuming excessive resources.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
