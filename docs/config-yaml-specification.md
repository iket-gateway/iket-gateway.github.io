---
layout: docs_page
title: config.yaml Specification
permalink: /docs/config-yaml-specification/
section: Core Docs
section_order: 2
weight: 3
summary: Understand the main Iket gateway configuration file used with `--config`, including server, TLS, storage, identity, authorization, plugin, and policy settings.
audience: [operator, developer]
topics: [configuration, reference, security]
---

# config.yaml Specification

`config.yaml` describes the gateway process itself: ports, timeouts, TLS, administrative security, storage mode, identity providers, authorization policies, global presets, and plugin configuration.

Use it with `service.yaml` when starting the gateway:

```bash
iket-server --config ./config/config.yaml --services ./config/service.yaml
```

The source-of-truth shape comes from `iket/pkg/config/config.go` and the nested structs under `iket/pkg/config`.

<div class="callout">
  <p><strong>Practical rule:</strong> keep process, security, identity, storage, and plugin settings in <code>config.yaml</code>. Keep services and routes in <code>service.yaml</code>. Iket can still load embedded <code>services:</code> blocks from <code>config.yaml</code>, but the split-file flow is cleaner for real deployments.</p>
</div>

## Specification Summary

| Item | Value |
|---|---|
| Document name | `config.yaml` |
| Primary CLI flag | `--config ./config/config.yaml` |
| Purpose | Gateway process, security, identity, storage, policy, and plugin configuration. |
| Format | YAML. The same structs also expose JSON tags for API and persistence paths. |
| Source structs | `Config`, `ServerConfig`, `SecurityConfig`, `StorageConfig`, `IdentityConfig`, `AuthConfig`, `AuthorizationConfig`, `TLSConfig`. |
| Companion file | `service.yaml`, passed through `--services ./config/service.yaml`. |
| Environment expansion | Supported in plugin config maps and common scaffolded values such as `${IKET_POSTGRES_URL}`. |
| Duration format | Go duration strings such as `250ms`, `10s`, `5m`, or `1h`. |

## Document Contract

- A valid document MUST include a valid `server.port`.
- A valid document MUST use supported storage mode values when `storage.mode` is set.
- A document using `storage.mode: postgres` MUST include `storage.postgres_url`.
- A document SHOULD keep route/service definitions outside this file and load them through `--services`.
- A document MAY define reusable identity, authentication, authorization, AI policy, and identity projection presets for routes to reference.
- A document MAY include a top-level `services` array for embedded service configuration, but split-file configuration is the recommended operator workflow.

## Minimal Example

```yaml
server:
  port: 8080
  readTimeout: "10s"
  writeTimeout: "10s"
  idleTimeout: "60s"
  enableLogging: true

security:
  tls:
    enabled: true
    port: 8443
    certFile: "./certs/server.crt"
    keyFile: "./certs/server.key"
    clientCAFile: "./certs/ca.crt"
    clientAuthType: "RequireAndVerifyClientCert"
    minVersion: "TLS1.3"
  enableBasicAuth: false
  ipWhitelist: []
  headers:
    X-Gateway: "iket"

storage:
  mode: "sqlite"
  sqlite_path: "./data/iket.db"
  mirror_files: true

plugins: {}
```

## Top-Level Fields

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `server` | `ServerConfig` | Yes | None | Listener, timeout, logging, plugin directory, and optional server-local TLS settings. |
| `security` | `SecurityConfig` | Recommended | Empty object behavior depends on active features | TLS, mTLS, basic auth, API clients, JWT, mutation policy, alerting, and notification controls. |
| `storage` | `StorageConfig` | No | `mode: sqlite`, `mirror_files: true` | Selects file, SQLite, or PostgreSQL storage and whether file mirrors are maintained. |
| `identity` | `IdentityConfig` | No | Empty | Defines identity providers and claim maps used by route auth strategies. |
| `auth` | `AuthConfig` | No | Empty | Names reusable authentication strategies for routes. |
| `authorization` | `AuthorizationConfig` | No | Empty | Names reusable authorization policies for routes. |
| `identityProjectionPresets` | map of `IdentityProjectionConfig` | No | Empty plus built-ins where applicable | Global identity-to-header projection presets available to services and routes. |
| `aiPolicyPresets` | map of `AIPolicyPreset` | No | Empty | Global route policy presets for model, tool, transform, and agent guardrails. |
| `services` | list of `ServiceConfig` | No | Empty | Optional embedded service config. Prefer `--services ./config/service.yaml` for new setups. |
| `plugins` | map | No | Empty | Plugin-specific configuration keyed by plugin name. |

## Schema Definitions

This section uses the same object names as the Go structs. Field names are the YAML property names users write in `config.yaml`.

### Config

```yaml
server: ServerConfig
security: SecurityConfig
storage: StorageConfig
identity: IdentityConfig
auth: AuthConfig
authorization: AuthorizationConfig
identityProjectionPresets:
  preset-name: IdentityProjectionConfig
aiPolicyPresets:
  preset-name: AIPolicyPreset
services:
  - ServiceConfig
plugins:
  plugin-name:
    plugin-specific-key: plugin-specific-value
```

### ServerConfig

```yaml
port: integer
readTimeout: duration
writeTimeout: duration
idleTimeout: duration
pluginsDir: string
enableLogging: boolean
tls: TLSConfig
```

### StorageConfig

```yaml
mode: "sqlite" | "file" | "postgres"
sqlite_path: string
postgres_url: string
mirror_files: boolean
```

### IdentityConfig

```yaml
providers:
  provider-name: IdentityProviderConfig
claim_maps:
  claim-map-name: IdentityClaimMap
```

### AuthConfig

```yaml
strategies:
  strategy-name: AuthStrategyConfig
```

### AuthorizationConfig

```yaml
policies:
  policy-name: AuthorizationPolicyConfig
```

## Server

`server` controls the main gateway listener and process-level defaults.

| Field | Type | Notes |
|---|---|---|
| `port` | integer | Required. Must be between `1` and `65535`. Used as the default HTTP port and as the fallback TLS port. |
| `readTimeout` | duration string | Optional Go duration such as `10s`, `500ms`, or `2m`. |
| `writeTimeout` | duration string | Optional Go duration for response writes. |
| `idleTimeout` | duration string | Optional Go duration for idle keep-alive connections. |
| `pluginsDir` | string | Optional path for plugin discovery. |
| `enableLogging` | boolean | Enables gateway logging. |
| `tls` | object | Optional server-local TLS config using the same `TLSConfig` shape as `security.tls`. |

Example:

```yaml
server:
  port: 8080
  readTimeout: "10s"
  writeTimeout: "10s"
  idleTimeout: "60s"
  pluginsDir: "./plugins"
  enableLogging: true
```

## Security

`security` is the main administrative and gateway security block.

| Field | Type | Purpose |
|---|---|---|
| `tls` | object | TLS and mTLS settings for secure gateway/admin traffic. |
| `enableBasicAuth` | boolean | Enables username/password checks when configured. |
| `basicAuthUsers` | map | Username-to-password map for basic auth. Use secrets or environment substitution in real deployments. |
| `ipWhitelist` | list | Optional allowed client IP/CIDR list. |
| `headers` | map | Static security or gateway headers to inject. |
| `clients` | map | Client ID to client secret map for API client access patterns. |
| `jwt` | object | JWT validation settings. See existing JWT/security guides for full policy behavior. |
| `mutationPolicy` | object | Rules for config/service mutation, approvals, proposal age, apply windows, and queue alerts. |
| `notificationWebhooks` | list | Webhook destinations for proposal, alert, digest, and gateway events. |
| `limiterClassPresets` | map | Reusable rate/concurrency class presets referenced from routes. |
| `limitAlertBucketClasses` | map | Bucket classification rules for alert grouping. |
| `limitAlertProfiles` | map | Alert recipient/filter profiles for notification routing. |

## TLS And mTLS

`TLSConfig` appears under `security.tls` and can also appear under `server.tls`.

| Field | Type | Notes |
|---|---|---|
| `enabled` | boolean | Enables TLS. |
| `port` | integer | TLS listener port. Falls back to `server.port` when unset. |
| `http3Enabled` | boolean | Enables HTTP/3 listener support when available. |
| `http3Port` | integer | HTTP/3 listener port. Falls back to TLS port when unset. |
| `http3Datagrams` | boolean | Enables HTTP/3 datagram support. |
| `enrollmentPort` | integer | Optional certificate enrollment listener port. |
| `enrollmentMaxActive` | integer | Maximum active enrollment sessions. Defaults to `10` when unset. |
| `certFile` | string | Server certificate file. |
| `certRef` | string | Optional secret reference for the server certificate. |
| `keyFile` | string | Server private key file. |
| `keyRef` | string | Optional secret reference for the server key. |
| `clientCAFile` | string | CA bundle used to verify client certificates. |
| `clientCARef` | string | Optional secret reference for the client CA. |
| `clientAuthType` | string | One of Go TLS client auth modes such as `NoClientCert`, `RequestClientCert`, `VerifyClientCertIfGiven`, or `RequireAndVerifyClientCert`. |
| `minVersion` | string | TLS minimum version, commonly `TLS1.2` or `TLS1.3`. |
| `ciphers` | list | Optional cipher suite allowlist. |
| `serverNames` | list | DNS names to include when auto-generating server certificates. |
| `serverIPs` | list | IP SANs to include when auto-generating server certificates. |
| `autoGenerate` | boolean | When true, Iket can generate missing TLS assets. |
| `generateSharedClient` | boolean | Controls generation of a shared client certificate during bootstrap. |

Production mTLS example:

```yaml
security:
  tls:
    enabled: true
    port: 8443
    certFile: "./certs/server.crt"
    keyFile: "./certs/server.key"
    clientCAFile: "./certs/ca.crt"
    clientAuthType: "RequireAndVerifyClientCert"
    minVersion: "TLS1.3"
    serverNames: ["gateway.internal.example.com"]
    serverIPs: ["10.10.0.15"]
    autoGenerate: false
```

## Storage

`storage` selects the gateway configuration and state backend.

| Field | Type | Notes |
|---|---|---|
| `mode` | string | `sqlite`, `file`, or `postgres`. Empty mode defaults to `sqlite`. |
| `sqlite_path` | string | SQLite database path when `mode: sqlite`. |
| `postgres_url` | string | Required when `mode: postgres`. |
| `mirror_files` | boolean | Defaults to true when omitted. Keeps YAML mirrors aligned with the active store. |

SQLite example:

```yaml
storage:
  mode: "sqlite"
  sqlite_path: "./data/iket.db"
  mirror_files: true
```

PostgreSQL example:

```yaml
storage:
  mode: "postgres"
  postgres_url: "${IKET_POSTGRES_URL}"
  mirror_files: true
```

## Identity Providers

The `identity` block defines identity sources and how claims are normalized.

```yaml
identity:
  providers:
    workforce:
      type: "oidc"
      issuer: "https://identity.example.com/realms/platform"
      jwks_uri: "https://identity.example.com/realms/platform/protocol/openid-connect/certs"
      client_id: "iket-gateway"
      client_secret_ref: "env:IKET_OIDC_CLIENT_SECRET"
  claim_maps:
    default:
      subject: "sub"
      client_id: "azp"
      tenant: ["tenant", "realm"]
      scopes: ["scope"]
      roles: ["realm_access.roles"]
      groups: ["groups"]
```

Provider fields:

| Field | Type | Purpose |
|---|---|---|
| `type` | string | Provider type, such as OIDC or introspection-style provider. |
| `issuer` | string | Expected token issuer. |
| `jwks_uri` | string | JWKS endpoint for JWT verification. |
| `introspection_uri` | string | Token introspection endpoint when using opaque tokens. |
| `client_id` | string | Client ID used by the provider integration. |
| `client_secret_ref` | string | Secret reference for provider client secret. |

## Auth Strategies

`auth.strategies` names reusable authentication behavior that routes can reference with `auth_strategy`.

```yaml
auth:
  strategies:
    workforce-jwt:
      type: "jwt"
      provider: "workforce"
      claim_map: "default"
      allowed_audiences: ["iket-api"]
      allowed_algorithms: ["RS256"]
      cache_ttl: "5m"
      header: "Authorization"
```

| Field | Type | Purpose |
|---|---|---|
| `type` | string | Strategy type. |
| `provider` | string | Identity provider name from `identity.providers`. |
| `claim_map` | string | Claim map name from `identity.claim_maps`. |
| `allowed_audiences` | list | Accepted token audiences. |
| `allowed_algorithms` | list | Accepted signing algorithms. |
| `cache_ttl` | duration string | Optional auth cache TTL. |
| `header` | string | Header carrying the credential. |

## Authorization Policies

`authorization.policies` names reusable allow rules that routes can reference with `authorization_policy`.

```yaml
authorization:
  policies:
    catalog-writers:
      scopes_all: ["catalog:write"]
      roles_any: ["admin", "catalog-editor"]
      tenants: ["prod"]
```

| Field | Type | Purpose |
|---|---|---|
| `all_of` | list | All nested conditions must pass. |
| `any_of` | list | At least one nested condition must pass. |
| `not` | object | Negates a nested condition. |
| `scopes_any` | list | Any listed scope is enough. |
| `scopes_all` | list | All listed scopes are required. |
| `roles_any` | list | Any listed role is enough. |
| `roles_all` | list | All listed roles are required. |
| `groups_any` | list | Any listed group is enough. |
| `client_ids` | list | Allowed client IDs. |
| `tenants` | list | Allowed tenant values. |
| `claims_match` | map | Required exact claim matches. |

<div class="callout callout--warning">
  <p><strong>Route dependency:</strong> a route using <code>authorization_policy</code> must also enable <code>requireAuth</code>, <code>auth_strategy</code>, or <code>auth_plugin</code>. Otherwise validation rejects the route because there is no identity to authorize.</p>
</div>

## Global Presets

Global presets are reusable building blocks that routes and services can inherit.

### Identity Projection Presets

`identityProjectionPresets` map authenticated identity attributes into upstream headers.

```yaml
identityProjectionPresets:
  upstream-default:
    subjectHeader: "X-User-Sub"
    tenantHeader: "X-Tenant"
    rolesHeader: "X-Roles"
    scopesHeader: "X-Scopes"
    clientIDHeader: "X-Client-ID"
    joinWith: ","
```

| Field | Purpose |
|---|---|
| `subjectHeader` | Header for normalized subject. |
| `tenantHeader` | Header for tenant value. |
| `rolesHeader` | Header for roles. |
| `groupsHeader` | Header for groups. |
| `scopesHeader` | Header for scopes. |
| `clientIDHeader` | Header for client ID. |
| `issuerHeader` | Header for issuer. |
| `providerHeader` | Header for provider name. |
| `authMethodHeader` | Header for auth method. |
| `attributeHeaders` | Map of custom identity attributes to headers. |
| `joinWith` | Separator for multi-value fields. |

### AI Policy Presets

`aiPolicyPresets` reuse route policy fields for model, tool, request, response, and agent guardrails.

```yaml
aiPolicyPresets:
  org-safe-agent:
    allowedModels: ["gpt-4.1-mini", "gpt-4.1"]
    allowedToolNames: ["web_search", "file_lookup"]
    allowedUpstreamHosts: ["api.openai.com"]
    requiredRequestHeaders: ["X-Agent-Session"]
    maxInputTokens: 4096
    maxOutputTokens: 1024
```

## Mutation Policy

`security.mutationPolicy` controls how remote config/service changes are allowed, proposed, reviewed, and alerted.

Common fields:

| Field | Purpose |
|---|---|
| `enabled` | Enables mutation policy enforcement. |
| `enforcedScopes` | Mutation scopes the policy applies to, such as `all`. |
| `requireLabel` | Requires labels on mutations. |
| `requireNoteForHighImpact` | Requires notes for high-impact changes. |
| `requireChangeRefForHighImpact` | Requires change references for high-impact changes. |
| `requireDifferentReviewerForProposals` | Prevents self-review when enabled. |
| `minApproversForHighImpactProposals` | Minimum approvals for high-impact proposals. |
| `requireNotBeforeForHighImpactProposals` | Requires scheduled activation time for high-impact proposals. |
| `maxProposalAge` | Expiration window for pending proposals. |
| `maxApprovalAge` | Expiration window for approvals. |
| `blockedApplyWindows` | Time windows when apply is blocked. |
| `proposalQueue` | Queue urgency and notification settings. |
| `policyAlertNotifications` | Notification policy for policy alerts. |
| `limitAlertNotifications` | Notification policy for limit alerts. |

## Plugins

`plugins` is a map keyed by plugin name. Values are plugin-specific and are passed through as generic configuration.

```yaml
plugins:
  api-key-auth:
    enabled: true
    header: "X-API-Key"
    clients:
      dashboard:
        secretRef: "env:DASHBOARD_API_KEY"
        scopes: ["dashboard:read"]
```

Environment variables inside plugin config maps are expanded when Iket loads config.

## Complete Production-Style Example

This example shows the shape of a real split-file setup. Route definitions stay in `service.yaml`.

```yaml
server:
  port: 8080
  readTimeout: "10s"
  writeTimeout: "10s"
  idleTimeout: "60s"
  pluginsDir: "./plugins"
  enableLogging: true

security:
  tls:
    enabled: true
    port: 8443
    certFile: "./certs/server.crt"
    keyFile: "./certs/server.key"
    clientCAFile: "./certs/ca.crt"
    clientAuthType: "RequireAndVerifyClientCert"
    minVersion: "TLS1.3"
    serverNames: ["gateway.internal.example.com"]
    serverIPs: ["10.10.0.15"]
    autoGenerate: false
  enableBasicAuth: false
  ipWhitelist: ["10.0.0.0/8"]
  headers:
    X-Gateway: "iket"
  mutationPolicy:
    enabled: true
    enforcedScopes: ["all"]
    requireLabel: true
    requireNoteForHighImpact: true
    requireChangeRefForHighImpact: true
    minApproversForHighImpactProposals: 1
    maxProposalAge: "72h"
    proposalQueue:
      defaultUrgency:
        readyAgingAfter: "1h"
        readyOverdueAfter: "4h"
      notifications:
        enabled: true
        interval: "15m"
        onlyOnChange: true

storage:
  mode: "postgres"
  postgres_url: "${IKET_POSTGRES_URL}"
  mirror_files: true

identity:
  providers:
    workforce:
      type: "oidc"
      issuer: "https://identity.example.com/realms/platform"
      jwks_uri: "https://identity.example.com/realms/platform/protocol/openid-connect/certs"
      client_id: "iket-gateway"
      client_secret_ref: "env:IKET_OIDC_CLIENT_SECRET"
  claim_maps:
    default:
      subject: "sub"
      client_id: "azp"
      tenant: ["tenant"]
      scopes: ["scope"]
      roles: ["realm_access.roles"]

auth:
  strategies:
    workforce-jwt:
      type: "jwt"
      provider: "workforce"
      claim_map: "default"
      allowed_audiences: ["iket-api"]
      allowed_algorithms: ["RS256"]
      cache_ttl: "5m"
      header: "Authorization"

authorization:
  policies:
    catalog-writers:
      scopes_all: ["catalog:write"]
      roles_any: ["admin", "catalog-editor"]

identityProjectionPresets:
  upstream-default:
    subjectHeader: "X-User-Sub"
    tenantHeader: "X-Tenant"
    rolesHeader: "X-Roles"
    scopesHeader: "X-Scopes"
    joinWith: ","

aiPolicyPresets:
  org-safe-agent:
    allowedModels: ["gpt-4.1-mini", "gpt-4.1"]
    allowedUpstreamHosts: ["api.openai.com"]
    requiredRequestHeaders: ["X-Agent-Session"]
    maxInputTokens: 4096
    maxOutputTokens: 1024

plugins: {}
```

## Embedded Services

New deployments should usually keep routes in `service.yaml`, then start with:

```bash
iket-server --config ./config/config.yaml --services ./config/service.yaml
```

If `services:` is embedded in `config.yaml`, it uses the same `ServiceConfig` shape documented in the [service.yaml specification]({{ '/docs/service-yaml-specification/' | relative_url }}).

## Validation Notes

- `server.port` must be in the valid TCP port range.
- Duration strings use Go duration syntax such as `250ms`, `10s`, `5m`, or `1h`.
- `storage.mode` must be `sqlite`, `file`, or `postgres`.
- `storage.postgres_url` is required when `storage.mode` is `postgres`.
- Auth strategies referenced by routes must exist in `auth.strategies`.
- Authorization policies referenced by routes must exist in `authorization.policies`.

## Startup Contract

When both files are passed, Iket loads and validates the combined runtime config:

```bash
iket-server --config ./config/config.yaml --services ./config/service.yaml
```

The server config and service config are loaded together, so routes in `service.yaml` can reference auth strategies, authorization policies, identity projection presets, AI policy presets, and limiter presets defined in `config.yaml`.

## Related Docs

<div class="docs-grid">
  <article class="doc-card">
    <h3><a href="{{ '/docs/service-yaml-specification/' | relative_url }}">service.yaml Specification</a></h3>
    <p>Use this next to define services, routes, upstream backends, BFF routes, and route policies.</p>
  </article>
  <article class="doc-card">
    <h3><a href="{{ '/docs/config-storage/' | relative_url }}">Config Storage</a></h3>
    <p>Understand how file, SQLite, and PostgreSQL-backed configuration fit together.</p>
  </article>
</div>
