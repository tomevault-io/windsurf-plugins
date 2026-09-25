---
trigger: always_on
description: A curl-like client for OHTTP, supporting GET and POST requests over HTTP/2 and HTTP/3 with TLS.
---

# AGENTS.md — pvcli

A curl-like client for OHTTP, supporting GET and POST requests over HTTP/2 and HTTP/3 with TLS.

## STRUCTURE
```
pvcli/
├── src/
│   ├── bin/
│   │   └── pvcli/
│   │       └── main.rs         # Binary entry point (calls tunnel())
│   ├── client/
│   │   ├── mod.rs              # HttpClient trait, HttpClientKind enum, HttpResponse struct
│   │   ├── cert.rs             # X509ConnectionHook, CertSettings, build_ssl_context_builder
│   │   ├── http2.rs            # Http2Client: direct HTTP/2 requests with BoringSSL (hyper-boring)
│   │   ├── http3/              # HTTP/3 client module
│   │   │   ├── mod.rs          # Http3Client: QUIC/H3 requests via tokio-quiche
│   │   │   ├── body.rs         # H3Body type for streaming response bodies
│   │   │   └── connection.rs   # QUIC connection management, SendRequest handle
│   │   ├── ohttp.rs            # OHttpClient: OHTTP-encrypted requests via proxy (H2 or H3)
│   │   ├── request.rs          # RequestHandler trait: shared request building/dispatch logic
│   │   └── resolver.rs         # Resolver: hostname resolution and resolution overrides
│   ├── lib.rs                  # Core logic: run(), select_http_client(), logging setup
│   ├── args.rs                 # CLI argument definitions (clap), Args validation
│   └── error.rs                # PvcliError enum (thiserror)
├── crates/                     # Workspace crates
│   ├── hyper-binary/           # BHTTP encoding/decoding
│   ├── ohttp-hpke/             # OHTTP HPKE client implementation
│   ├── stream-buf/             # Stream buffer utilities
│   ├── stream-octets/          # Octet stream utilities
│   └── ohttp-gateway-worker/   # Local OHTTP gateway worker
├── tests/
│   ├── integration_tests.rs    # Integration tests for HTTP2, HTTP3, and OHTTP clients
│   ├── common/
│   │   └── mod.rs              # Mock server setup, test constants
│   └── testdata.txt            # Test fixture file
└── docker-compose.test.yml     # Docker config for integration testing
```

## WHERE TO LOOK
| Task | Location |
|---|---|
| Add/change CLI flags | `src/args.rs` |
| Add new HTTP client type | `src/client/mod.rs` (add to `HttpClientKind` enum) |
| Modify HTTP/2 behavior | `src/client/http2.rs` |
| Modify HTTP/3 behavior | `src/client/http3/mod.rs` |
| Modify QUIC connection logic | `src/client/http3/connection.rs` |
| Modify OHTTP behavior | `src/client/ohttp.rs` |
| QUIC/H3 TLS cert configuration | `src/client/cert.rs` (`X509ConnectionHook`) |
| Shared request building/dispatch | `src/client/request.rs` (`RequestHandler` trait) |
| Add new error variants | `src/error.rs` |
| Arg validation logic | `src/args.rs` (`Args::validate`) |
| Client selection logic | `src/lib.rs` (`select_http_client`) |
| DNS resolution | `src/client/resolver.rs` |
| Logging configuration | `src/lib.rs` (`configure_logging`) |
| Mock server routes | `tests/common/mod.rs` |
| Integration test cases | `tests/integration_tests.rs` |
| TLS configuration | `src/args.rs` (`TlsConfig`) |
| TLS cert configuration (H2 + H3) | `src/client/cert.rs` (`build_ssl_context_builder`, `X509ConnectionHook`) |

## CODE MAP
| Symbol | Type | Location | Role |
|---|---|---|---|
| `tunnel` | fn | `src/lib.rs` | Top-level async entry: parse args, configure logging, dispatch request |
| `run` | fn | `src/lib.rs` | Core request flow: validate args, select client, return body as string |
| `raw_run` | fn | `src/lib.rs` | Like `run` but returns `HttpResponse` instead of body string |
| `run_handle_error` | fn | `src/lib.rs` | Wrapper that handles errors and returns response body or logs error |
| `select_http_client` | fn | `src/lib.rs` | Returns `HttpClientKind` based on args (OHTTP vs HTTP/2 vs HTTP/3) |
| `Args` | struct | `src/args.rs` | Clap-parsed CLI arguments |
| `TlsConfig` | struct | `src/args.rs` | TLS configuration holder (cacert path) |
| `Args::validate` | method | `src/args.rs` | Calls setup_args, validates basic and proxy args |
| `RequestArgs` | struct | `src/args.rs` | Validated request parameters (method, url, headers, body, `proxy_connect`, `proxy_tls_config`, `proxy_header`) |
| `ResolveOverride` | struct | `src/args.rs` | `--resolve` value type: `host:port:addr[,addr]...` parsed via `FromStr` |
| `Method` | enum | `src/args.rs` | `Get` \| `Post` — case-insensitive via clap |
| `HttpClient` | trait | `src/client/mod.rs` | Trait for `send_request()` — implemented by client types |
| `HttpClientKind` | enum | `src/client/mod.rs` | `OHttp(OHttpClient)` \| `Http2(Http2Client)` \| `Http3(Http3Client)` |
| `ProxyClientKind` | enum | `src/client/mod.rs` | `Http2(Http2Client)` \| `Http3(Http3Client)` — proxy transport used by `OHttpClient` |
| `HttpResponse` | struct | `src/client/mod.rs` | `{ version, status, headers, body }` with helper methods |
| `HttpBody` | type alias | `src/client/mod.rs` | `BoxBody<Bytes, std::io::Error>` — unified body type |
| `X509ConnectionHook` | struct | `src/client/cert.rs` | `ConnectionHook` impl: configures BoringSSL TLS context for QUIC (custom CA, optional mTLS) |
| `build_ssl_context_builder` | fn | `src/client/cert.rs` | Shared TLS builder: sets PEER verify mode, loads custom CA or system defaults into an `SslContextBuilder` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflareresearch/pvcli](https://github.com/cloudflareresearch/pvcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
