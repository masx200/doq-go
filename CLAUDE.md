# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

This is a DNS over QUIC (DoQ) client library written in Go, implementing
RFC9250. The library is built on top of
[quic-go](https://github.com/quic-go/quic-go) and
[miekg/dns](https://github.com/miekg/dns) libraries.

## Common Commands

### Testing

```bash
go test ./doq -v                    # Run tests with verbose output
go test ./doq -run TestName         # Run specific test
go test ./doq -cover                # Run tests with coverage
```

### Building

```bash
go build ./doq                      # Build the doq package
go build ./...                      # Build all packages
```

### Linting

```bash
golangci-lint run                   # Run linter (same as CI)
go vet ./...                        # Run go vet
go fmt ./...                        # Format code
```

### Dependencies

```bash
go mod tidy                         # Clean up dependencies
go mod download                     # Download dependencies
go get github.com/tantalor93/doq-go # Install as dependency
```

## Architecture

The codebase follows a simple, focused architecture:

### Core Components

- **`doq/client.go`**: Main Client struct that encapsulates DoQ logic
  - Thread-safe client with connection pooling
  - Handles QUIC connection management and stream multiplexing
  - Implements connection retry logic with health checking

- **`doq/opts.go`**: Configuration options using functional options pattern
  - `WithTLSConfig()`: Custom TLS configuration
  - `WithWriteTimeout()`: Write timeout configuration
  - `WithReadTimeout()`: Read timeout configuration
  - `WithConnectTimeout()`: Connection timeout configuration

- **`doq/client_test.go`**: Comprehensive test suite
  - Mock DoQ server for testing
  - Timeout testing scenarios
  - TLS certificate generation for test environment

### Key Design Patterns

1. **Functional Options Pattern**: Configuration through `Option` interface
2. **Connection Reuse**: Single QUIC connection with multiple parallel streams
3. **Context-based Timeout Handling**: Separate timeouts for connect, read, and
   write operations
4. **Thread Safety**: Connection management with proper locking

### Protocol Implementation

- Follows RFC9250 for DNS over QUIC
- Uses 2-octet length prefix for DNS messages (as per specification)
- ALPN protocol negotiation with "doq" string
- TLS 1.2+ requirement for security

## Testing Requirements

Tests require TLS certificates (`test.crt` and `test.key`) in the project root
for the mock DoQ server. If these files are missing, tests will panic when
trying to start the test server.

## Development Notes

- Target Go version: 1.21+ (see go.mod)
- Uses `quic-go` for QUIC protocol implementation
- Uses `miekg/dns` for DNS message handling
- Client is designed to be thread-safe and connection-aware
- Connection health is checked before each operation
