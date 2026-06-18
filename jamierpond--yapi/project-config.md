---
trigger: always_on
description: CLI-first API testing for HTTP, GraphQL, gRPC, and TCP.
---

# yapi

CLI-first API testing for HTTP, GraphQL, gRPC, and TCP.

## The Workflow

yapi enables test-driven API development. Write the test first, then implement until it passes:

1. **Write the test** - Create a `.yapi.yml` file with the expected behavior
2. **Run it** - `yapi run file.yapi.yml` (it will fail)
3. **Implement/fix** - Build the API endpoint
4. **Iterate** - Refine assertions, add edge cases

This loop is the core of agentic API development with yapi.

---

## Environment Setup (Do This First)

Before writing any tests, set up your environments. Create `yapi.config.yml` in your project root:

```yaml
yapi: v1
default_environment: local

environments:
  local:
    url: http://localhost:3000
    vars:
      API_KEY: dev_key_123

  staging:
    url: https://staging.example.com
    vars:
      API_KEY: ${STAGING_API_KEY}  # from shell env

  prod:
    url: https://api.example.com
    vars:
      API_KEY: ${PROD_API_KEY}
    env_files:
      - .env.prod  # load secrets from file
```

Now your tests use `${url}` and `${API_KEY}` - same test, any environment:

```bash
yapi run get-users.yapi.yml              # uses local (default)
yapi run get-users.yapi.yml --env staging
yapi run get-users.yapi.yml --env prod
```

**Variable resolution order** (highest priority first):
1. Shell environment variables
2. Environment-specific `vars`
3. Environment-specific `env_files`
4. Default `vars`
5. Default `env_files`

---

## A) Smoke Testing

Quick health checks to verify endpoints are alive.

### HTTP

```yaml
yapi: v1
url: ${url}/health
method: GET
expect:
  status: 200
```

### GraphQL

```yaml
yapi: v1
url: ${url}/graphql
graphql: |
  query { __typename }
expect:
  status: 200
  assert:
    - .data.__typename != null
```

### gRPC

```yaml
yapi: v1
url: grpc://${host}:${port}
service: grpc.health.v1.Health
rpc: Check
plaintext: true
body:
  service: ""
expect:
  status: 200
```

### TCP

```yaml
yapi: v1
url: tcp://${host}:${port}
data: "PING\n"
encoding: text
expect:
  status: 200
```

---

## B) Integration Testing

Multi-step workflows with data passing between requests. Use chains when steps depend on each other.

### Authentication Flow

```yaml
yapi: v1
chain:
  - name: login
    url: ${url}/auth/login
    method: POST
    body:
      email: test@example.com
      password: ${TEST_PASSWORD}
    expect:
      status: 200
      assert:
        - .token != null

  - name: get_profile
    url: ${url}/users/me
    method: GET
    headers:
      Authorization: Bearer ${login.token}
    expect:
      status: 200
      assert:
        - .email == "test@example.com"
```

### CRUD Flow

```yaml
yapi: v1
chain:
  - name: create
    url: ${url}/posts
    method: POST
    body:
      title: "Test Post"
      content: "Hello World"
    expect:
      status: 201
      assert:
        - .id != null

  - name: read
    url: ${url}/posts/${create.id}
    method: GET
    expect:
      status: 200
      assert:
        - .title == "Test Post"

  - name: update
    url: ${url}/posts/${create.id}
    method: PATCH
    body:
      title: "Updated Post"
    expect:
      status: 200

  - name: delete
    url: ${url}/posts/${create.id}
    method: DELETE
    expect:
      status: 204
```

### Running Integration Tests

Name test files with `.test.yapi.yml` suffix:
```
tests/
  auth.test.yapi.yml
  posts.test.yapi.yml
  users.test.yapi.yml
```

Run all tests:
```bash
yapi test ./tests                    # sequential
yapi test ./tests --parallel 4       # concurrent
yapi test ./tests --env staging      # against staging
yapi test ./tests --verbose          # detailed output
```

---

## C) Uptime Monitoring

Create test suites for monitoring your services in production.

### Monitor Suite Structure

```
monitors/
  api-health.test.yapi.yml
  auth-service.test.yapi.yml
  database-check.test.yapi.yml
  graphql-schema.test.yapi.yml
```

### Health Check with Timeout

```yaml
yapi: v1
url: ${url}/health
method: GET
timeout: 5s  # fail if response takes longer
expect:
  status: 200
  assert:
    - .status == "healthy"
    - .database == "connected"
```

### Run Monitoring Suite

```bash
# Check all monitors in parallel
yapi test ./monitors --parallel 10 --env prod

# With verbose output for debugging
yapi test ./monitors --parallel 10 --env prod --verbose
```

### CI/CD Integration (GitHub Actions)

```yaml
name: API Health Check
on:
  schedule:
    - cron: '*/5 * * * *'  # every 5 minutes
  workflow_dispatch:

jobs:
  monitor:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install yapi
        run: curl -fsSL https://yapi.run/install/linux.sh | bash

      - name: Run health checks
        env:
          PROD_API_KEY: ${{ secrets.PROD_API_KEY }}
        run: yapi test ./monitors --env prod --parallel 5
```

### Load Testing

Stress test endpoints or entire workflows:

```bash
# 1000 requests, 50 concurrent
yapi stress api-flow.yapi.yml -n 1000 -p 50

# Run for 30 seconds
yapi stress api-flow.yapi.yml -d 30s -p 25

# Against production (with confirmation)
yapi stress api-flow.yapi.yml -e prod -n 500 -p 10
```

---

## D) Async Job Polling with `wait_for`

For endpoints that process data asynchronously, use `wait_for` to poll until conditions are met.

### Fixed Period Polling

```yaml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jamierpond/yapi](https://github.com/jamierpond/yapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
