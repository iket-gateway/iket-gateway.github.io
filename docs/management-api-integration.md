---
layout: docs_page
title: Iket Management API Integration Guide
permalink: /docs/management-api-integration/
section: Start Here
section_order: 1
weight: 4
summary: Integrate with the management API securely using mTLS and understand the main administrative endpoints and real-time channels.
audience: [developer, operator]
topics: [api, management, security]
---

# Iket Management API Integration Guide

This guide explains how to integrate and secure the management API of the Iket API Gateway, including the use of `iket` for secure remote administration via mTLS.

<div class="doc-note">
  <p><strong>Use this guide when:</strong> you are moving beyond manual CLI-only administration and want automation, dashboards, or external systems to talk to Iket’s administrative surface safely.</p>
</div>

## Overview

The Iket Management API provides REST endpoints and WebSocket connections for:
- Gateway status monitoring
- Plugin management (enable/disable/config)
- Service and Route configuration
- Real-time metrics and logs
- Configuration reloading

All management operations are secured by default using **mTLS (Mutual TLS)** and **HTTP Basic Authentication**.

<div class="doc-warning">
  <p><strong>Security first:</strong> treat the management API like privileged infrastructure, not like a public application API. Strong certificates, restricted network reachability, and careful credential handling should be the baseline.</p>
</div>

---

## 🔒 Security Configuration

### 1. Enable mTLS in Gateway

To secure the management endpoints, you must configure TLS and client certificate verification in your `config.yaml`:

```yaml
security:
  tls:
    enabled: true
    certFile: "/app/certs/server.crt"
    keyFile: "/app/certs/server.key"
    clientCAFile: "/app/certs/ca.crt"
    clientAuthType: "RequireAndVerifyClientCert" # MANDATORY for secure admin
  
  enableBasicAuth: true
  basicAuthUsers:
    admin: "your-secure-password"
```

### 2. Management Authentication Flow

1. **Transport Layer**: The client must provide a certificate signed by the `clientCAFile` configured on the server.
2. **Application Layer**: The client must provide valid Basic Auth credentials (username/password) defined in `basicAuthUsers`.

---

## 🛠️ Administrative Tool: `iket`

The `iket` client is the recommended way to interact with the Management API. It handles the mTLS handshake and credential management automatically.

<div class="doc-tip">
  <p><strong>Recommended rollout path:</strong> get the same workflow working through `iket` first, then mirror it programmatically. That makes it much easier to separate API mistakes from certificate or network issues.</p>
</div>

### Installation
```bash
# Using the ultimate installer
curl -fsSL https://raw.githubusercontent.com/bhangun/iket/main/scripts/install.sh | bash
```

### Configuration (`cli-config.yaml`)
`iket` manages its settings in `~/.iket/cli-config.yaml`. The easiest way to configure it is via the guided setup:
```bash
iket setup
```

Alternatively, you can manage multiple **Contexts** manually:
```bash
# Add a remote production context
iket context add prod \
  --url https://api.yourdomain.com:8443 \
  --ca ~/.iket/certs/ca.crt \
  --cert ~/.iket/certs/client.crt \
  --key ~/.iket/certs/client.key
```

---

## API Endpoints

All Management API endpoints are prefixed with `/api/v1`.

### Gateway Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/gateway/status` | Get gateway health, uptime, and request stats |
| GET | `/api/v1/gateway/config` | Get current configuration (secrets redacted) |
| PUT | `/api/v1/gateway/config` | Update gateway configuration |
| POST | `/api/v1/gateway/reload` | Trigger a hot-reload of services and routes |

### Service & Route Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/services` | List all services and their defined routes |
| POST | `/api/v1/services` | Register a new service definition |
| PUT | `/api/v1/services/{id}` | Update an existing service |
| DELETE | `/api/v1/services/{id}` | Remove a service |
| POST | `/api/v1/routes/{id}/enable` | Enable a specific route |
| POST | `/api/v1/routes/{id}/disable` | Disable a specific route |

### Plugin Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/plugins` | List all active and available plugins |
| GET | `/api/v1/plugins/{name}` | Get plugin details and full diagnostics when available |
| PUT | `/api/v1/plugins/{name}/config` | Update a plugin's global configuration |
| POST | `/api/v1/plugins/{name}/enable` | Enable a plugin globally |
| POST | `/api/v1/plugins/{name}/disable` | Disable a plugin globally |
| GET | `/api/v1/plugins/{name}/health` | Run plugin health checks when supported |
| GET | `/api/v1/plugins/{name}/status` | Get plugin status from `StatusReporter` or `DiagnosticsReporter` |

Plugin list/detail responses include a `capabilities` array derived from the
plugin interfaces that are implemented, so dashboards can decide whether to
show health checks, status, diagnostics, middleware, reload, lifecycle, or usage
observer controls without hard-coded plugin names. The status endpoint also
includes `status_source` so dashboards can tell whether the top-level status
came from a human-readable `StatusReporter` string or the machine-readable
diagnostics payload.

The plugin list endpoint can be filtered with `type`, `enabled`, `status`,
`diagnostics_status`, and repeatable or comma-separated
`capability`/`capabilities` query parameters. When multiple capabilities are
provided, Iket returns plugins that implement all requested capabilities.
Plugin list ordering is deterministic by plugin name, including entries backed
by plugin factories, so dashboards and CLIs can avoid noisy inventory diffs.
Responses also include a `summary` object with inventory-wide enabled/disabled
counts and facets by type, health status, diagnostics status, and capability;
`summary.matched` reports how many plugins matched the active filters.

---

## Programmatic Integration (Go)

If you are building a custom management dashboard in Go, use the `APIClient` provided in the Iket package.

```go
package main

import (
    "fmt"
    "github.com/bhangun/iket/pkg/api_client" // Use the provided client
)

func main() {
    // Initialize secure client
    client, err := api_client.NewAPIClient(
        "https://iket-server:8080",
        false,                      // skipVerify
        "/path/to/ca.crt",
        "/path/to/client.crt",
        "/path/to/client.key",
    )
    if err != nil {
        panic(err)
    }

    // Call management endpoints
    status, err := client.Do("GET", "/api/v1/gateway/status", nil)
    fmt.Println(string(status))
}
```

---

## WebSocket for Real-time Monitoring

Iket provides WebSockets for real-time streaming of events and metrics.

| Endpoint | Description |
|----------|-------------|
| `/api/v1/logs/stream` | Stream server logs in real-time |
| `/api/v1/metrics/system` | Get live system resource usage (CPU/Mem) |

---

## Troubleshooting

### 1. "TLS handshake error: remote error: tls: bad certificate"
- **Cause**: The client did not provide a certificate, or the certificate is not signed by the CA configured in `clientCAFile`.
- **Solution**: Ensure your `cli-config.yaml` points to the correct `cert_file` and `key_file`.

### 2. "401 Unauthorized"
- **Cause**: mTLS succeeded, but the Basic Auth credentials provided in the header are invalid.
- **Solution**: Check your `basicAuthUsers` in the server `config.yaml`.

### 3. "Connection refused"
- **Cause**: The server is not listening on the expected port or TLS is not enabled.
- **Solution**: Verify the `port` and `security.tls.enabled` settings in `config.yaml`.

## Next Steps

<div class="docs-grid">
  <article class="doc-card">
    <h3><a href="{{ '/docs/cli-commands/' | relative_url }}">Use the CLI as the baseline</a></h3>
    <p>Return to the command reference if you want to compare management API behavior with the safer, guided CLI workflow.</p>
  </article>
  <article class="doc-card">
    <h3><a href="{{ '/docs/production-deployment/' | relative_url }}">Harden production access</a></h3>
    <p>Move here next if the management API will be exposed in production and you need the surrounding network and mTLS posture to be solid.</p>
  </article>
</div>
