---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# QUIC Erlang Library (Quicer)

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Project Overview
Quicer is an Erlang library providing QUIC (next-generation transport protocol) support through NIF (Native Implemented Function) bindings to Microsoft's msquic library. Written in Erlang with C native code.

**CRITICAL BUILD TIMING**: Build and test processes require significant time - NEVER CANCEL builds or tests prematurely.

## Working Effectively

### Bootstrap and Build
- Install dependencies (Ubuntu/Debian):
  ```bash
  sudo apt-get update
  sudo apt-get install -y erlang build-essential cmake git curl wget clang-format-14
  ```
- Get rebar3:
  ```bash
  wget https://s3.amazonaws.com/rebar3/rebar3 && chmod +x rebar3
  sudo mv rebar3 /usr/local/bin/
  ```
- **Build the native library**: `make build-nif` -- takes 6-11 minutes. NEVER CANCEL. Set timeout to 20+ minutes.
- **Compile Erlang code**: `rebar3 compile` -- takes 30-60 seconds. Set timeout to 2+ minutes.
- **Run full CI**: `make ci` -- takes 10-20 minutes. NEVER CANCEL. Set timeout to 30+ minutes.

### Testing
- **Unit tests**: `make eunit` -- takes 2-5 minutes. NEVER CANCEL. Set timeout to 10+ minutes.
- **Integration tests**: `make ct` -- takes 5-15 minutes. NEVER CANCEL. Set timeout to 25+ minutes.
- **Property-based tests**: `make proper` -- takes 3-8 minutes. NEVER CANCEL. Set timeout to 15+ minutes.
- **All tests**: `make test` -- takes 8-20 minutes. NEVER CANCEL. Set timeout to 30+ minutes.

### Static Analysis and Quality
- **Type analysis**: `make dialyzer` -- takes 2-5 minutes. NEVER CANCEL. Set timeout to 10+ minutes.
- **Dead code detection**: `make hank` -- takes 30-60 seconds. Set timeout to 2+ minutes.
- **Code formatting**: `make fmt` -- takes 5-15 seconds. Set timeout to 1+ minute.
- **Format checking**: `make clang-format` -- takes 5-15 seconds. Set timeout to 1+ minute.

## Validation Requirements
ALWAYS manually validate any new code by running through complete user scenarios after making changes:

### Basic Functionality Test
After any code changes, ALWAYS run this validation scenario:
```erlang
% Start the application
application:ensure_all_started(quicer).

% Test basic connection to Google (external validation)
{ok, Conn} = quicer:connect("google.com", 443, [{alpn, ["h3"]}, {verify, verify_peer}], 5000).
quicer:shutdown_connection(Conn).

% Test local ping-pong scenario (requires build completion for certificates)
% After running 'make build-nif', certificates are available at:
% - ./msquic/submodules/openssl/test/certs/rootCA.pem
% - ./msquic/submodules/openssl/test/certs/servercert.pem  
% - ./msquic/submodules/openssl/test/certs/serverkey.pem
% Example server: 
% Port = 4567,
% {ok, L} = quicer:listen(Port, [{certfile, "./msquic/submodules/openssl/test/certs/servercert.pem"}, 
%                                {keyfile, "./msquic/submodules/openssl/test/certs/serverkey.pem"}, 
%                                {alpn, ["sample"]}]).
% Example client: 
% {ok, Conn} = quicer:connect("localhost", Port, [{alpn, ["sample"]}, {verify, none}], 5000).
```

### Build Validation Checklist
Before completing any work, ALWAYS run:
1. `make build-nif` - Ensure native library builds successfully
2. `make test` - Ensure all tests pass
3. `make dialyzer` - Ensure no type errors
4. `make hank` - Ensure no dead code
5. `make fmt` - Format all code properly

## Dependencies Requirements
- **OTP 25+** (Erlang/OTP runtime)
- **rebar3** (Erlang build tool)
- **cmake 3.16+** (C build system)
- **build-essential** (C compiler toolchain)
- **clang-format-14** (C code formatting, exact version required)
- **git, curl, wget** (build dependencies)

## Build Process Details
The build happens in these phases:
1. **msquic download**: `./get-msquic.sh v2.3.8` (~3 seconds)
2. **cmake configuration**: `cmake -B c_build` (~6 seconds)  
3. **native compilation**: `make -C c_build -j$(nproc)` (~5-10 minutes)
4. **erlang compilation**: `rebar3 compile` (~30-60 seconds)

## Troubleshooting

### Debug Logging
Enable debug logging with environment variable:
```bash
QUIC_LOGGING_TYPE=stdout make
QUIC_LOGGING_TYPE=stdout rebar3 ct --suite test/quicer_connection_SUITE.erl --case tc_conn_basic_verify_peer
```

### Advanced Build Configurations
Available environment variables:
- `QUICER_TLS_VER=sys` - Use system OpenSSL instead of bundled
- `CMAKE_BUILD_TYPE=Debug` - Debug build with symbols
- `QUIC_ENABLE_LOGGING=ON` - Enable msquic logging
- `QUIC_LOGGING_TYPE=stdout` - Log to stdout instead of lttng

### Sanitizer and Debugging Tools
For advanced debugging (requires special OTP builds):
```bash
tools/run/bin/sanitizer-check all  # Memory sanitizer tests
tools/run/bin/debug-check all      # Debug build tests  
tools/run/bin/valgrind-check all   # Valgrind analysis
```

### Wireshark Traffic Decryption
Use `sslkeylogfile` parameter in client connections:
```erlang
{ok, Conn} = quicer:connect("google.com", 443, [
    {verify, verify_peer},
    {sslkeylogfile, "/tmp/SSLKEYLOGFILE"},
    {alpn, ["h3"]}
], 5000)
```

### Common Build Issues

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emqx/quic](https://github.com/emqx/quic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
