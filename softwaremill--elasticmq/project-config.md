---
trigger: always_on
description: ElasticMQ is an in-memory, SQS-compatible message queue system. It can run stand-alone, via Docker, or embedded in Scala/Java applications. This repository contains:
---

# ElasticMQ - Project Documentation

## Project Overview

ElasticMQ is an in-memory, SQS-compatible message queue system. It can run stand-alone, via Docker, or embedded in Scala/Java applications. This repository contains:

- **Scala backend** — actor-based core + Pekko HTTP SQS REST interface
- **Next.js UI** (`ui/`) — web interface for monitoring and managing queues

## Repository Structure

```
elasticmq/
├── core/                          # Core actor-based queue implementation
├── common-test/                   # Shared test utilities
├── persistence/
│   ├── persistence-core/          # Base persistence interfaces
│   ├── persistence-file/          # File-based persistence (PureConfig)
│   └── persistence-sql/           # SQL persistence (ScalikeJDBC + H2)
├── rest/
│   ├── rest-sqs/                  # SQS-compatible REST API (Pekko HTTP)
│   └── rest-sqs-testing-amazon-java-sdk/  # Integration tests with AWS SDKs
├── server/                        # Stand-alone server entry point
│   └── docker/                    # Docker config files
├── native-server/                 # GraalVM native image build (Java 11+)
├── integration-tests/
├── performance-tests/
├── examples/
├── ui/                            # Next.js web UI
└── build.sbt
```

## Backend (Scala)

### Tech Stack

- **Language:** Scala (cross-compiled: 2.12, 2.13, 3.x; default 2.13)
- **Actor system:** Apache Pekko
- **HTTP:** Pekko HTTP + spray-json
- **Persistence:** PureConfig (file), ScalikeJDBC + H2 (SQL)
- **Config:** Typesafe Config
- **Build:** sbt

### Key Modules

| Module | Artifact | Purpose |
|--------|----------|---------|
| `core` | `elasticmq-core` | Actor-based queue engine |
| `rest/rest-sqs` | `elasticmq-rest-sqs` | SQS REST API |
| `persistence/persistence-file` | `elasticmq-persistence-file` | File-based queue persistence |
| `persistence/persistence-sql` | `elasticmq-persistence-sql` | SQL-based queue persistence |
| `server` | `elasticmq-server` | Stand-alone runnable JAR |

### Development Commands

```bash
# Run all tests
sbt test

# Run specific module tests
sbt "restSqs/test"

# Build stand-alone JAR
sbt "server/assembly"

# Build Docker image
sbt "server/docker:publishLocal"

# Run server locally
java -jar server/target/scala-2.13/elasticmq-server-*.jar
```

### Running ElasticMQ

**Stand-alone:**
```bash
java -jar elasticmq-server-$VERSION.jar
# Binds to localhost:9324 (SQS)
```

**Docker:**
```bash
docker run -p 9324:9324 softwaremill/elasticmq
```

**Exposed ports:**
- `9324` — SQS-compatible REST API

### Configuration

ElasticMQ uses Typesafe Config (`elasticmq.conf`). Pass with:
```bash
java -Dconfig.file=/path/to/elasticmq.conf -jar elasticmq-server.jar
```

### Scala Cross-Compilation

Set Scala version via environment variable before running sbt:
```bash
SCALA_MAJOR_VERSION=2.12 sbt test
SCALA_MAJOR_VERSION=3    sbt test
```

## UI (Next.js)

### Tech Stack

- **Framework:** Next.js 16 (App Router)
- **Language:** TypeScript (strict mode)
- **Styling:** Tailwind CSS v4
- **AWS Integration:** @aws-sdk/client-sqs
- **Package Manager:** npm
- **Runtime:** Node.js LTS (via nvm)

### Structure

```
ui/
├── app/
│   ├── layout.tsx              # Root layout with ThemeProvider
│   ├── page.tsx                # Queue list page
│   ├── globals.css             # Global styles + theme variables
│   └── queues/[name]/page.tsx  # Dynamic queue details page
├── components/
│   ├── queue-list.tsx          # Queue list with polling (client)
│   ├── queue-card.tsx          # Individual queue card display
│   ├── queue-details.tsx       # Queue details with polling (client)
│   ├── page-header.tsx         # Page header with theme switcher (client)
│   ├── theme-switcher.tsx      # Theme toggle component (client)
│   ├── loading-skeleton.tsx    # Loading state component
│   └── error-display.tsx       # Error state component
├── lib/
│   ├── types.ts                # TypeScript interfaces
│   ├── sqs-client.ts           # SQS client config (localhost:9324)
│   ├── actions.ts              # Server actions ('use server')
│   ├── utils.ts                # Formatting utilities
│   └── theme-provider.tsx      # Theme context provider (client)
└── .env.local                  # Environment variables (gitignored)
```

### Development Commands

```bash
cd ui

# Use Node LTS
nvm use lts

# Install dependencies
npm install

# Start dev server (http://localhost:3000)
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Lint
npm run lint
```

### Environment Variables

**File:** `ui/.env.local` (gitignored) — copy from `ui/.env.local.example`.

**Local ElasticMQ (default):**
```bash
SQS_ENDPOINT=http://localhost:9324
AWS_REGION=elasticmq
AWS_ACCESS_KEY_ID=x
AWS_SECRET_ACCESS_KEY=x
```

**Real AWS account:**
```bash
# SQS_ENDPOINT is omitted — SDK defaults to the real AWS endpoint
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
# Disable auto-refresh to avoid unnecessary API calls and costs:
NEXT_PUBLIC_AUTO_REFRESH_DISABLED=true
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [softwaremill/elasticmq](https://github.com/softwaremill/elasticmq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
