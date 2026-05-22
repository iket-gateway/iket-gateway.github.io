---
layout: docs_page
title: API Key Management & Authentication
permalink: /docs/api-key-management/
section: Core Docs
section_order: 2
weight: 1
summary: Configure API key authentication, client grouping, and scoped access patterns for applications that call your gateway.
audience: [developer, operator]
topics: [authentication, api-keys, security]
---

# 🔑 API Key Management & Authentication

The `apikey` plugin provides a robust way to manage and authenticate client applications using API keys. It supports scopes and grouping for fine-grained access control.

---

## 🚀 Overview

The API Key system allows you to:
- **Authenticate** clients via a header (`X-API-Key`) or query parameter (`api_key`).
- **Group** clients (e.g., `internal`, `partner`, `external`) to restrict access to specific service sets.
- **Scope** permissions (e.g., `read`, `write`, `admin`) at both service and route levels.
- **Inject** client identity and metadata into the request context for backend usage.

---

## 🛠️ Global Configuration

Enable the plugin in your `config.yaml`:

```yaml
plugins:
  apikey:
    enabled: true
    header_name: "X-API-Key"  # Optional, default: X-API-Key
    query_param: "api_key"    # Optional, default: api_key
    usage_observer_timeout: "100ms" # Optional, default observer timeout
    usage_observer_async: false     # Optional, set true for non-blocking exports
    usage_observer_async_max_in_flight: 1024 # Optional async safety cap
    clients:
      - id: "mobile-app"
        name: "Official iOS App"
        key: "secret-key-123"
        group: "external"
        scopes: ["read", "metrics:read"]
        tags: ["prod", "mobile"]
```

---

## 🛣️ Service & Route Protection

Apply API Key protection and define requirements in your service definitions.

### 1. Group-Based Restriction
Restrict a service to a specific client group.

```yaml
name: "Inventory Service"
host: "http://inventory-api:8080"
group: "internal"  # Only clients in the 'internal' group can access this service
routes:
  - path: "/api/v1/stock"
    requireAuth: true
    auth_plugin: "apikey"
```

### 2. Scope-Based Authorization
Require specific scopes for routes. Scopes can be defined at the service level (inherited by all routes) or route level.

```yaml
name: "User Management"
host: "http://user-service:8081"
scopes: ["users:read"] # All routes in this service require 'users:read'
routes:
  - path: "/api/v1/users"
    method: "GET"
    requireAuth: true
    auth_plugin: "apikey"
    # Inherits 'users:read'

  - path: "/api/v1/users"
    method: "POST"
    requireAuth: true
    auth_plugin: "apikey"
    scopes: ["users:write"] # Requires BOTH 'users:read' AND 'users:write'
```

---

## 💻 CLI Management

Manage clients dynamically without restarting the gateway.

### List Clients
```bash
iket client list
```

### Add a Client
```bash
iket client add "partner-xyz" \
  --name "Partner Integration" \
  --key "part-key-789" \
  --group "partner" \
  --scopes "read,orders:create" \
  --tags "external,partner"
```

### Delete a Client
```bash
iket client delete "part-key-789"
```

---

## 🧠 Technical Reference

### Context Injection
When a request is authenticated, the following values are injected into the Go `request.Context()`:

| Key | Type | Description |
|-----|------|-------------|
| `apikey_client_id` | `string` | The unique ID of the client app. |
| `apikey_group` | `string` | The group the client belongs to. |
| `apikey_scopes` | `[]string` | The list of scopes assigned to the client. |

### Usage & Metering Events
The API key plugin also emits a redacted usage event for each client-attributed
request. Community builds can use these events for local visibility and basic
operational reporting, while enterprise or billing plugins can subscribe through
the `ClientUsageObserver` contract and export billable usage without changing
the core gateway path.

Each event is designed to be safe to forward to metering systems:

| Field | Description |
|-------|-------------|
| `schema_version` | Versioned event shape so downstream consumers can evolve safely. |
| `event_id` | Unique event identifier for idempotent export and deduplication. |
| `quantity` | Countable usage amount for the request, typically one request unless a plugin maps a different billable unit. |
| Normalized dimensions | Stable client, group, route, service, method, and status-class style dimensions for aggregation. |
| Request attribution | Redacted request identity such as client ID, route/service match, method, path template, and request ID when available. |
| Response outcome | Status code, response bytes, and duration so billing and analytics can separate successful, rejected, and failed calls. |
| Panic-safe outcome | Recovered handler panics are reported as `500` outcomes instead of dropping the usage record. |

Before exporting an event, observer plugins should consume the normalized
metering shape and validate it with the core contract. Validation checks the
required `schema_version`, `event_id`, `provider`, `quantity`, `occurred_at`,
canonical dimensions, response bounds, and sensitive identity redaction.
The metering wrapper is transparent to normal handlers and preserves streaming
and advanced response-writer capabilities such as flush, delegated stream copy,
HTTP/2 push, and close notifications when the underlying server supports them.

By default, usage observers are flushed before the authenticated request
returns. For enterprise billing/export plugins that should never add tail
latency to gateway traffic, set `usage_observer_async: true`. Async delivery
detaches the observer context from request cancellation while preserving context
values, then still applies the configured observer timeout. Async mode also
uses `usage_observer_async_max_in_flight` as a bounded safety valve; when the
cap is saturated, new observer deliveries are dropped instead of creating
unbounded goroutines on the request path.

Management plugin list responses expose a compact diagnostics summary through
`diagnostics_available`, `diagnostics_status`, and warning codes when present.
Plugin detail/status endpoints expose the full structured API-key diagnostics
payload for automation. The full payload includes configured client count,
usage observer count, named/unnamed observer counts, safe observer names,
delivery mode, timeout, async max in-flight, current in-flight, and dropped
async delivery count. It also includes a machine-readable `status` plus
`warning_codes` such as `unnamed_observers_registered` and
`async_deliveries_dropped`, so enterprise exporters can alert without parsing
the human status string.

Usage events never include raw API keys, raw secrets, or unredacted credential
headers/query values. Treat client IDs and dimensions as reporting identifiers,
not secret material.

### Authorization Logic
1. **Authentication**: Validates that the provided key exists in the active client list.
2. **Group Check**: If the service has a `group` defined, the client's group must match exactly.
3. **Scope Check**: If the service or route has `scopes` defined, the client must possess **at least one** of the required scopes from the combined set (OR logic within the check, but effectively AND if you consider service vs route inheritance).

### Error Responses
- `401 Unauthorized`: Missing or invalid API key.
- `403 Forbidden`: Client group mismatch or insufficient scopes.

---

## 📝 Complete Example

**Client Definition:**
- ID: `monitoring-bot`
- Key: `bot-123`
- Group: `ops`
- Scopes: `["metrics:read", "health:check"]`

**Service Definition:**
```yaml
name: "Status Dashboard"
host: "http://status:80"
group: "ops"
routes:
  - path: "/api/metrics"
    requireAuth: true
    auth_plugin: "apikey"
    scopes: ["metrics:read"]
```

**Result:**
- `curl -H "X-API-Key: bot-123" http://gateway/api/metrics` -> **200 OK**
- `curl -H "X-API-Key: bot-123" http://gateway/api/other-internal-service` -> **403 Forbidden** (Group mismatch)
- `curl http://gateway/api/metrics` -> **401 Unauthorized** (Missing key)
