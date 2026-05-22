---
trigger: always_on
description: This document outlines the CI/CD pipeline configuration and best practices for the OpenFrame project.
---

# CI/CD Pipeline

This document outlines the CI/CD pipeline configuration and best practices for the OpenFrame project.

## Pipeline Overview

OpenFrame uses GitHub Actions for continuous integration and deployment, following these stages:

1. **Build**: Compile code and build artifacts
2. **Test**: Run unit and integration tests
3. **Analyze**: Perform static code analysis and security scanning
4. **Package**: Create Docker images
5. **Deploy**: Deploy to staging and production environments

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│         │     │         │     │         │     │         │     │         │
│  Build  │────▶│  Test   │────▶│ Analyze │────▶│ Package │────▶│ Deploy  │
│         │     │         │     │         │     │         │     │         │
└─────────┘     └─────────┘     └─────────┘     └─────────┘     └─────────┘
```

## GitHub Actions Workflow

The main workflow is defined in `.github/workflows/deploy.yml`:

```yaml
name: Build and Deploy

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK
      uses: actions/setup-java@v3
      with:
        java-version: '21'
        distribution: 'temurin'
        cache: 'maven'
        
    - name: Build with Maven
      run: mvn -B clean package -DskipTests
      
    - name: Cache Maven packages
      uses: actions/cache@v3
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
        restore-keys: ${{ runner.os }}-m2
        
    - name: Upload build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: build-artifacts
        path: |
          **/target/*.jar
          !**/target/classes
          !**/target/test-classes

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK
      uses: actions/setup-java@v3
      with:
        java-version: '21'
        distribution: 'temurin'
        cache: 'maven'
        
    - name: Download build artifacts
      uses: actions/download-artifact@v3
      with:
        name: build-artifacts
        
    - name: Run Tests
      run: mvn -B test
      
    - name: Upload test results
      uses: actions/upload-artifact@v3
      with:
        name: test-results
        path: |
          **/target/surefire-reports
          **/target/site/jacoco

  analyze:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK
      uses: actions/setup-java@v3
      with:
        java-version: '21'
        distribution: 'temurin'
        cache: 'maven'
        
    - name: SonarQube Scan
      run: mvn -B sonar:sonar
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
        
    - name: OWASP Dependency Check
      run: mvn -B org.owasp:dependency-check-maven:check
      
    - name: Upload analysis results
      uses: actions/upload-artifact@v3
      with:
        name: analysis-results
        path: |
          **/target/dependency-check-report.html
          **/target/sonar

  package:
    needs: [test, analyze]
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
      
    - name: Login to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
        
    - name: Download build artifacts
      uses: actions/download-artifact@v3
      with:
        name: build-artifacts
        
    - name: Build and push API image
      uses: docker/build-push-action@v4
      with:
        context: ./services/openframe-api
        push: ${{ github.event_name != 'pull_request' }}
        tags: openframe/api:latest,openframe/api:${{ github.sha }}
        
    - name: Build and push Gateway image
      uses: docker/build-push-action@v4
      with:
        context: ./services/openframe-gateway
        push: ${{ github.event_name != 'pull_request' }}
        tags: openframe/gateway:latest,openframe/gateway:${{ github.sha }}
        
    # Additional services...

  deploy-staging:
    if: github.event_name != 'pull_request'
    needs: package
    runs-on: ubuntu-latest
    environment: staging
    steps:
    - uses: actions/checkout@v3
    
    - name: Set Kubernetes context
      uses: azure/k8s-set-context@v3
      with:
        kubeconfig: ${{ secrets.KUBE_CONFIG_STAGING }}
        
    - name: Deploy to Kubernetes
      run: |
        # Update image tags
        sed -i "s|image: openframe/api:.*|image: openframe/api:${{ github.sha }}|g" kubernetes/staging/*.yaml
        sed -i "s|image: openframe/gateway:.*|image: openframe/gateway:${{ github.sha }}|g" kubernetes/staging/*.yaml
        
        # Apply Kubernetes manifests
        kubectl apply -f kubernetes/staging/
        
    - name: Verify deployment
      run: |
        kubectl rollout status deployment/openframe-api -n openframe
        kubectl rollout status deployment/openframe-gateway -n openframe


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-tenant](https://github.com/flamingo-stack/openframe-oss-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
