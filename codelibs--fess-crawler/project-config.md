---
trigger: always_on
description: Quick reference for AI assistants working on the Fess Crawler project.
---

# CLAUDE.md - Fess Crawler Development Guide

Quick reference for AI assistants working on the Fess Crawler project.

## Project Overview

**Fess Crawler** is a Java-based web crawling framework for enterprise content extraction.

### Essential Info

- **Language**: Java 21+
- **Build**: Maven 3.x
- **License**: Apache 2.0
- **DI**: LastaFlute DI
- **Repo**: https://github.com/codelibs/fess-crawler

### Tech Stack

- **HTTP**: Apache HttpComponents 4.5+ and 5.x (switchable)
- **Extraction**: Apache Tika, POI, PDFBox
- **Testing**: JUnit 4, UTFlute, Mockito, Testcontainers
- **Storage**: In-memory (default), OpenSearch (optional)
- **Cloud**: AWS SDK v2 (S3), Google Cloud Storage

### Protocols

HTTP/HTTPS, File, FTP/FTPS, SMB/CIFS (SMB1/SMB2+), Storage (MinIO via `storage://`), S3 (`s3://`), GCS (`gcs://`)

### Content Formats

Office (Word, Excel, PowerPoint), PDF, Archives (ZIP, TAR, GZ, LHA), HTML, XML, JSON, Markdown, Media metadata, Images (EXIF/IPTC/XMP), Email (EML)

---

## Architecture

### Module Structure

```
fess-crawler-parent/
├── fess-crawler/              # Core framework
├── fess-crawler-lasta/        # LastaFlute DI integration
└── fess-crawler-opensearch/   # OpenSearch backend
```

### Key Design Patterns

- **Factory**: `CrawlerClientFactory`, `ExtractorFactory` - protocol/format-specific component selection
- **Strategy**: `CrawlerClient`, `Extractor`, `Transformer` - pluggable implementations
- **Builder**: `RequestDataBuilder`, `ExtractorBuilder` - fluent construction
- **Template Method**: `AbstractCrawlerClient`, `AbstractExtractor` - common logic with overrides
- **DI**: LastaFlute container with `@Resource` and XML config

### Core Principles

**Thread Safety**: `AtomicLong` for counters, `volatile` for status flags, synchronized blocks, thread-local storage via `CrawlingParameterUtil`

**Resource Management**: `AutoCloseable` throughout, `DeferredFileOutputStream` for large responses, connection pooling, background temp file deletion via `FileUtil.deleteInBackground()`

**Fault Tolerance**: `FaultTolerantClient` wrapper (retry, circuit breaker), `SwitchableHttpClient` for HTTP client fallback

---

## Key Components

### Core Classes

- **Crawler** (`Crawler.java`): Main orchestrator - `execute()`, `addUrl()`, `cleanup()`, `stop()`
- **CrawlerContext** (`CrawlerContext.java`): Execution context - `sessionId`, `status`, `accessCount`, `numOfThread`, `maxDepth`, `maxAccessCount`
- **CrawlerThread** (`CrawlerThread.java`): Worker thread - Poll URL → Validate → Execute → Process → Queue children

### HTTP Client Architecture

```
SwitchableHttpClient (extends FaultTolerantClient)
    ├── Hc5HttpClient (default) - Apache HttpComponents 5.x
    └── Hc4HttpClient (fallback) - Apache HttpComponents 4.x

HcHttpClient (abstract base class)
    ├── Hc4HttpClient
    └── Hc5HttpClient
```

Switch via system property: `-Dfess.crawler.http.client=hc4` or `hc5` (default)

**Key Properties**: `connectionTimeout`, `soTimeout`, `proxyHost`, `proxyPort`, `userAgent`, `robotsTxtEnabled`, `ignoreSslCertificate`, `maxTotalConnection`, `defaultMaxConnectionPerRoute`

### CrawlerClientFactory

Pattern-based client selection (from `crawler/client.xml`):
- `http:.*`, `https:.*` → SwitchableHttpClient
- `file:.*` → FileSystemClient
- `smb:.*` → SmbClient (SMB2+), `smb1:.*` → SmbClient (SMB1)
- `ftp:.*`, `ftps:.*` → FtpClient
- `storage:.*` → StorageClient, `s3:.*` → S3Client, `gcs:.*` → GcsClient

### Cloud Storage Clients

- **S3Client**: AWS SDK v2, `s3://bucket/path`, properties: `endpoint`, `accessKey`, `secretKey`, `region`
- **GcsClient**: Google Cloud SDK, `gcs://bucket/path`, properties: `projectId`, `credentialsFile`, `endpoint`
- **StorageClient**: MinIO SDK, `storage://bucket/path`

### Services

- **UrlQueueService**: URL queue management (FIFO), duplicate detection
- **DataService**: Access result persistence, iteration
- Implementations: `UrlQueueServiceImpl`, `DataServiceImpl` (in-memory), `OpenSearchDataService` (persistent)

### Processing Pipeline

```
CrawlerThread → Client → ResponseProcessor → Transformer → Extractor → ExtractData
                                                                            ↓
                         ← UrlQueueService ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ←
                         ← DataService ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ←
```

- **Rule**: Pattern-based response routing (`RegexRule`, `SitemapsRule`)
- **ResponseProcessor**: `DefaultResponseProcessor`, `SitemapsResponseProcessor`, `NullResponseProcessor`
- **Transformer**: `HtmlTransformer`, `XmlTransformer`, `FileTransformer`, etc.
- **Extractor**: Weight-based selection (tries in descending weight order)

### Key Extractors

`TikaExtractor`, `PdfExtractor`, `MsWordExtractor`, `MsExcelExtractor`, `MsPowerPointExtractor`, `ZipExtractor`, `HtmlExtractor`, `MarkdownExtractor`, `EmlExtractor`

### Helpers

- **RobotsTxtHelper**: RFC 9309 parsing, user-agent matching, crawl-delay, sitemaps
- **SitemapsHelper**: Sitemap XML parsing, index handling
- **MimeTypeHelper**: MIME detection via Tika
- **EncodingHelper**: Charset detection with BOM
- **UrlConvertHelper**: URL normalization
- **ContentLengthHelper**: Content length limits per MIME type

---

## Development Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codelibs/fess-crawler](https://github.com/codelibs/fess-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
