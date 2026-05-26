---
layout: docs_page
title: service.yaml Specification
permalink: /docs/service-yaml-specification/
section: Core Docs
section_order: 2
weight: 4
summary: Understand the service and route configuration file used with `--services`, including routes, backends, policies, transforms, BFF composition, and protocol-specific options.
audience: [operator, developer]
topics: [configuration, routing, reference]
---

# service.yaml Specification

`service.yaml` describes the traffic surface Iket should expose: services, base paths, routes, upstream backends, route policies, transforms, limits, protocol behavior, and optional BFF composition.

Use it with `config.yaml` when starting the gateway:

```bash
iket-server --config ./config/config.yaml --services ./config/service.yaml
```

The source-of-truth shape comes from `iket/pkg/config/service_config.go`, `iket/pkg/config/route_config.go`, and the route policy structs under `iket/pkg/config`.

<div class="callout">
  <p><strong>Mental model:</strong> <code>config.yaml</code> configures the gateway process. <code>service.yaml</code> configures what the gateway routes.</p>
</div>

## Specification Summary

| Item | Value |
|---|---|
| Document name | `service.yaml` |
| Primary CLI flag | `--services ./config/service.yaml` |
| Purpose | Service, route, backend, protocol, transform, limit, and BFF route configuration. |
| Format | YAML. The same structs also expose JSON tags for management API and persistence paths. |
| Source structs | `ServiceConfig`, `Service`, `RouterConfig`, `Backend`, `BFFConfig`, `BFFStepConfig`, `CORSConfig`, `WebSocketOptions`. |
| Companion file | `config.yaml`, passed through `--config ./config/config.yaml`. |
| Path composition | `service.base_path` + `route.path` becomes the exposed route path. |
| Duration format | Go duration strings such as `100ms`, `2s`, `5m`, or `1h`. |

## Document Contract

- A valid document MUST include positive `version`.
- A valid document MUST include at least one service.
- Each service MUST include unique `name`, valid `host`, and at least one route.
- Each route MUST include `path` and either `method` or `methods`.
- Ordinary proxy routes MUST include at least one `backend` entry.
- Routes MAY omit `backend` when they are plugin, internal, or enabled BFF routes.
- Routes MAY reference auth strategies, authorization policies, and global presets from `config.yaml`.

## Minimal Example

```yaml
version: 1
services:
  - name: "Product Service"
    description: "Product catalog API"
    host: "http://localhost:7105"
    base_path: "/api"
    tags: ["public", "catalog"]
    group: "commerce"
    routes:
      - path: "/products/{rest:.*}"
        methods: ["GET", "POST"]
        protocol: "http"
        requireAuth: false
        stripPath: false
        enabled: true
        backend:
          - url_pattern: "/v1/products/{rest:.*}"
            weight: 100
```

With the example above, Iket exposes `/api/products/{rest:.*}` because `base_path` and route `path` are joined.

## Top-Level Fields

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `version` | integer | Yes | None | Service config version. Must be positive. |
| `services` | list of `Service` | Yes | None | Service definitions. At least one service is required. |
| `cache_ttl` | duration string | No | Empty | Optional default cache TTL for the service config block. |
| `timeout` | duration string | No | Empty | Optional default timeout for the service config block. |

Iket can load multiple `ServiceConfig` blocks internally, but a standalone `service.yaml` normally uses one top-level `version` and one top-level `services` list.

## Schema Definitions

This section uses the same object names as the Go structs. Field names are the YAML property names users write in `service.yaml`.

### ServiceConfig

```yaml
version: integer
cache_ttl: duration
timeout: duration
services:
  - Service
```

### Service

```yaml
name: string
description: string
host: string
base_path: string
identityProjectionPresets:
  preset-name: IdentityProjectionConfig
aiPolicyPresets:
  preset-name: AIPolicyPreset
tags:
  - string
group: string
scopes:
  - string
routes:
  - RouterConfig
```

### RouterConfig

```yaml
path: string
method: string
methods:
  - string
protocol: "http" | "graphql" | "grpc" | "grpc-web" | "websocket" | "sse" | "bff"
requireAuth: boolean
auth_strategy: string
authorization_policy: string
enabled: boolean
stripPath: boolean
backend:
  - Backend
bff: BFFConfig
cors: CORSConfig
websocket: WebSocketOptions
```

### Backend

```yaml
url_pattern: string
host: string
weight: integer
timeout: duration
failureThreshold: integer
cooldown: duration
healthCheckPath: string
healthInterval: duration
healthTimeout: duration
```

## Service Fields

| Field | Type | Required | Purpose |
|---|---|---|---|
| `name` | string | Yes | Unique service name inside the config block. |
| `description` | string | No | Human-readable description for operators. |
| `host` | string | Yes | Default upstream host for routes and backends. Must be a valid service host. |
| `base_path` | string | No | Optional public prefix for all service routes. Must start with `/` when set. |
| `identityProjectionPresets` | map | No | Service-local identity projection presets. |
| `aiPolicyPresets` | map | No | Service-local AI/policy presets. |
| `tags` | list | No | Search, grouping, and operator metadata. |
| `group` | string | No | Logical service group. |
| `scopes` | list | No | Service-level scopes metadata. |
| `routes` | list | Yes | Route definitions. At least one route is required. |

Example with service-local presets:

```yaml
version: 1
services:
  - name: "Agent API"
    host: "https://agent-api.internal"
    group: "ai"
    identityProjectionPresets:
      upstream-user:
        subjectHeader: "X-User-Sub"
        tenantHeader: "X-Tenant"
        scopesHeader: "X-Scopes"
        joinWith: ","
    aiPolicyPresets:
      strict-agent:
        allowedModels: ["gpt-4.1-mini"]
        allowedUpstreamHosts: ["api.openai.com"]
        requiredRequestHeaders: ["X-Agent-Session"]
    routes:
      - path: "/chat"
        method: POST
        requireAuth: true
        auth_strategy: "workforce-jwt"
        identityProjectionPreset: "upstream-user"
        aiPolicyPreset: "strict-agent"
        backend:
          - url_pattern: "/v1/chat/completions"
```

## Route Identity And Matching

Every route needs a `path` and either `method` or `methods`.

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `path` | string | Yes | None | Public route path. Must start with `/`. Supports path variables used by backend patterns. |
| `method` | string | Conditional | None | Single HTTP method. Required when `methods` is absent. |
| `methods` | list | Conditional | None | Multiple HTTP methods. Required when `method` is absent. |
| `matchHeaders` | map | No | Empty | Route only matches when incoming headers equal the configured values. |
| `matchPercent` | integer | No | `0` | Percentage-style route matching for canary or gradual traffic behavior. |
| `name` | string | No | Empty | Operator-friendly route name. |
| `description` | string | No | Empty | Operator-friendly route description. |
| `tags` | list | No | Empty | Route metadata. |
| `group` | string | No | Empty | Route grouping metadata. |
| `priority` | integer | No | `0` | Non-negative route priority. |
| `enabled` | boolean | No | Enabled when omitted | False disables the route. |

Example:

```yaml
routes:
  - path: "/orders/{order_id}"
    method: GET
    matchHeaders:
      X-Tenant: "prod"
    priority: 10
    enabled: true
    backend:
      - url_pattern: "/internal/orders/{order_id}"
```

## Backend Fields

`backend` is a list. A route normally needs at least one backend unless it is a plugin, internal, or enabled BFF route.

| Field | Type | Required | Purpose |
|---|---|---|---|
| `url_pattern` | string | Yes | Upstream path pattern. Path variables from the route can be reused. |
| `host` | string | No | Backend-specific host override. Falls back to service `host`. |
| `weight` | integer | No | Non-negative traffic weight for weighted backends. |
| `timeout` | duration string | No | Backend-specific timeout. |
| `failureThreshold` | integer | No | Circuit breaker failure threshold. |
| `cooldown` | duration string | No | Circuit breaker cooldown duration. |
| `halfOpenMaxRequests` | integer | No | Max trial requests during half-open recovery. |
| `recoverySuccessThreshold` | integer | No | Successful trial requests needed for recovery. |
| `outlierLatencyThreshold` | duration string | No | Latency threshold for outlier detection. |
| `outlierConsecutiveSlowResponses` | integer | No | Slow responses before marking an outlier. |
| `outlierCooldown` | duration string | No | Outlier cooldown duration. |
| `healthCheckPath` | string | No | Health check path. Must start with `/` when set. |
| `healthInterval` | duration string | No | Health check interval. |
| `healthTimeout` | duration string | No | Health check timeout. |

Weighted backend example:

```yaml
backend:
  - host: "https://orders-v1.internal"
    url_pattern: "/orders/{order_id}"
    weight: 90
    timeout: "2s"
  - host: "https://orders-v2.internal"
    url_pattern: "/orders/{order_id}"
    weight: 10
    timeout: "2s"
```

## Route Field Matrix

This matrix groups the main route fields by the part of request handling they affect.

| Group | Fields |
|---|---|
| Identity and auth | `requireAuth`, `requireJwt`, `auth_plugin`, `auth_strategy`, `authorization_policy`, `roles`, `scopes`, `identityProjectionPreset`, `identityProjectionPresetChain`, `identityProjectionPresets`, `identityProjection` |
| Matching and metadata | `path`, `method`, `methods`, `matchHeaders`, `matchPercent`, `name`, `description`, `tags`, `group`, `priority`, `enabled` |
| Backend and routing | `backend`, `stripPath`, `protocol`, `validateSchema`, `adaptiveLatencyRouting`, `allowedUpstreamHosts` |
| Resilience | `retryCount`, `retryBackoff`, `retryJitter`, `retryStatusCodes`, `retryUnsafeMethods`, `hedgeDelay`, `hedgeUnsafeMethods` |
| Shadow traffic | `shadowTrafficPercent`, `shadowUnsafeMethods`, `shadowMinRequests`, `shadowMaxErrorRate`, `shadowMaxLatencyDelta` |
| Limits | `rateLimit`, `rateLimitPolicy`, `concurrencyLimitPolicy`, `limitAlertPolicy`, `maxRequestBodyBytes`, `maxResponseBodyBytes` |
| Request mutation | `requestHeaders`, `removeRequestHeaders`, `requestRedactHeaders`, `requiredRequestHeaders`, `requiredRequestHeaderRegex`, `queryParams`, `removeQueryParams`, `requestJSONFields`, `removeRequestJSONFields`, `requestRedactJSONFields` |
| Request inspection | `requestBodyBlockRegex`, `requestBodyRequireRegex`, `requestPIIBlockTypes` |
| Response mutation | `responseHeaders`, `removeResponseHeaders`, `responseRedactHeaders`, `successResponseFields`, `errorResponseFields`, `responseJSONFields`, `removeResponseJSONFields`, `responseRedactJSONFields` |
| Response inspection | `responseBodyBlockRegex`, `responseBodyRequireRegex`, `responsePIIBlockTypes`, `responseTransformStatusCodes`, `responseTransformStatusClasses`, `responseTransformWhenHeaders`, `responseTransformHeaderRegex` |
| Protocol-specific | `cors`, `websocket`, GraphQL fields, `bff` |
| AI and agent policy | `aiPolicyPreset`, `aiPolicyPresetChain`, `aiPolicyPresets`, `allowedModels`, `allowedToolNames`, token limits, agent context, approval, signature, action ID, and replay fields |

## Authentication And Authorization

Routes can require identity and then apply named authorization policies from `config.yaml`.

| Field | Type | Purpose |
|---|---|---|
| `requireAuth` | boolean | Requires authenticated access. |
| `requireJwt` | boolean | Requires JWT validation for the route. |
| `auth_plugin` | string | Uses a named auth plugin. |
| `auth_strategy` | string | References `auth.strategies` from `config.yaml`. |
| `authorization_policy` | string | References `authorization.policies` from `config.yaml`. |
| `roles` | list | Route role metadata or simple role requirements depending on active auth mode. |
| `scopes` | list | Route scope metadata or simple scope requirements depending on active auth mode. |

```yaml
routes:
  - path: "/admin/catalog"
    methods: ["POST", "PUT"]
    requireAuth: true
    auth_strategy: "workforce-jwt"
    authorization_policy: "catalog-writers"
    backend:
      - url_pattern: "/catalog/admin"
```

## Identity Projection

Identity projection forwards normalized identity information to upstream services.

| Field | Type | Purpose |
|---|---|---|
| `identityProjectionPreset` | string | Uses one named preset. |
| `identityProjectionPresetChain` | list | Applies multiple presets in order. |
| `identityProjectionPresets` | map | Route-local preset definitions. |
| `identityProjection` | object | Inline projection for a single route. |

Inline projection example:

```yaml
identityProjection:
  subjectHeader: "X-User-Sub"
  tenantHeader: "X-Tenant"
  rolesHeader: "X-Roles"
  scopesHeader: "X-Scopes"
  joinWith: ","
```

## Request Transformation And Validation

Request controls run before the upstream request is sent.

| Field | Type | Purpose |
|---|---|---|
| `headers` | map | Legacy/static header map. |
| `requestHeaders` | map | Add or replace upstream request headers. |
| `removeRequestHeaders` | list | Remove request headers before proxying. |
| `requestRedactHeaders` | list | Redact sensitive request headers in logs or policy handling. |
| `requiredRequestHeaders` | list | Require specific request headers. |
| `requiredRequestHeaderRegex` | map | Require headers matching regex patterns. |
| `queryParams` | map | Add or replace upstream query parameters. |
| `removeQueryParams` | list | Remove query parameters. |
| `requestJSONFields` | map | Add or update JSON request fields. |
| `removeRequestJSONFields` | list | Remove JSON request fields. |
| `requestRedactJSONFields` | list | Redact JSON request fields. |
| `requestBodyBlockRegex` | list | Block request bodies matching any regex. |
| `requestBodyRequireRegex` | list | Require request bodies to match configured regex. |
| `requestPIIBlockTypes` | list | Block configured PII classes in request bodies. |
| `maxRequestBodyBytes` | integer | Maximum buffered request body size. |

{% raw %}
```yaml
requestHeaders:
  X-Request-Trace: "{{request_id}}"
requiredRequestHeaders: ["X-Agent-Session"]
requiredRequestHeaderRegex:
  Authorization: "^Bearer .+"
requestJSONFields:
  meta.request_id: "{{request_id}}"
  meta.enabled: "json:true"
removeRequestJSONFields: ["legacy"]
requestBodyBlockRegex: ["(?i)ignore\\s+previous\\s+instructions"]
maxRequestBodyBytes: 65536
```
{% endraw %}

Conditional transform fields:

| Field | Purpose |
|---|---|
| `transformWhenHeaders` | Apply transforms only when headers match exact values. |
| `transformWhenQueryParams` | Apply transforms only when query params match exact values. |
| `transformWhenHeaderRegex` | Apply transforms only when headers match regex patterns. |
| `transformWhenQueryRegex` | Apply transforms only when query params match regex patterns. |
| `transformMethods` | Apply transforms only for listed methods. |
| `transformScopes` | Restrict transform categories. Allowed values include `request_headers`, `query`, `request_json`, `response_headers`, and `response_json`. |

## Response Transformation And Validation

Response controls run after the upstream response returns.

| Field | Type | Purpose |
|---|---|---|
| `responseHeaders` | map | Add or replace response headers. |
| `removeResponseHeaders` | list | Remove response headers. |
| `responseRedactHeaders` | list | Redact response headers. |
| `successResponseFields` | map | Build a success response envelope. |
| `errorResponseFields` | map | Build an error response envelope. |
| `responseJSONFields` | map | Add or update JSON response fields. |
| `removeResponseJSONFields` | list | Remove JSON response fields. |
| `responseRedactJSONFields` | list | Redact JSON response fields. |
| `responseBodyBlockRegex` | list | Block responses matching a regex. |
| `responseBodyRequireRegex` | list | Require responses to match a regex. |
| `responsePIIBlockTypes` | list | Block configured PII classes in responses. |
| `responseTransformStatusCodes` | list | Apply response transforms only to listed status codes. |
| `responseTransformStatusClasses` | list | Apply transforms to classes such as `2xx`, `4xx`, or `5xx`. |
| `responseTransformWhenHeaders` | map | Apply transforms only when response headers match. |
| `responseTransformHeaderRegex` | map | Apply transforms only when response headers match regex. |
| `maxResponseBodyBytes` | integer | Maximum buffered response body size. |

{% raw %}
```yaml
responseHeaders:
  X-Gateway-Trace: "{{request_id}}"
successResponseFields:
  data: "{{response_body}}"
  request_id: "{{request_id}}"
errorResponseFields:
  error.message: "{{response_body}}"
  error.status: "json:{{response_status}}"
responseJSONFields:
  meta.request_id: "{{request_id}}"
removeResponseHeaders: ["X-Powered-By"]
responseBodyBlockRegex: ["(?i)api_key"]
```
{% endraw %}

## Rate And Concurrency Limits

Simple legacy field:

| Field | Type | Purpose |
|---|---|---|
| `rateLimit` | integer | Basic route rate limit. |

Structured rate limit policy:

| Field | Type | Purpose |
|---|---|---|
| `rateLimitPolicy.requestsPerSecond` | number | Token refill rate. |
| `rateLimitPolicy.burst` | integer | Burst capacity. |
| `rateLimitPolicy.keyBy` | string | Limit key source, such as IP, header, JWT subject, or route. |
| `rateLimitPolicy.keyHeader` | string | Header name when `keyBy` uses a header. |
| `rateLimitPolicy.exemptMethods` | list | Methods skipped by the limiter. |
| `rateLimitPolicy.classPolicies` | list | Class-specific rate policy overrides or presets. |

Structured concurrency policy:

| Field | Type | Purpose |
|---|---|---|
| `concurrencyLimitPolicy.maxInFlight` | integer | Maximum active requests. |
| `concurrencyLimitPolicy.keyBy` | string | Concurrency key source. |
| `concurrencyLimitPolicy.keyHeader` | string | Header name when keyed by header. |
| `concurrencyLimitPolicy.queueTimeout` | duration string | How long queued requests may wait. |
| `concurrencyLimitPolicy.maxQueueDepth` | integer | Maximum queued requests. |
| `concurrencyLimitPolicy.exemptMethods` | list | Methods skipped by the limiter. |
| `concurrencyLimitPolicy.classPolicies` | list | Class-specific concurrency overrides or presets. |

```yaml
rateLimitPolicy:
  requestsPerSecond: 20
  burst: 40
  keyBy: "header"
  keyHeader: "X-Agent-Session"
  exemptMethods: ["OPTIONS"]
concurrencyLimitPolicy:
  maxInFlight: 8
  keyBy: "jwt_sub"
  queueTimeout: "150ms"
  maxQueueDepth: 32
```

## Retry, Hedge, Shadow, And Adaptive Routing

| Field | Type | Purpose |
|---|---|---|
| `retryCount` | integer | Number of retry attempts. |
| `retryBackoff` | duration string | Delay between retries. |
| `retryJitter` | duration string | Jitter added to retry timing. |
| `retryStatusCodes` | list | Status codes that trigger retries. |
| `retryUnsafeMethods` | boolean | Allows retry for unsafe methods such as POST/PATCH. |
| `hedgeDelay` | duration string | Delay before sending a hedged request. |
| `hedgeUnsafeMethods` | boolean | Allows hedging unsafe methods. |
| `adaptiveLatencyRouting` | boolean | Enables latency-aware backend selection. |
| `shadowTrafficPercent` | integer | Percentage of matching traffic mirrored to shadow behavior. |
| `shadowUnsafeMethods` | boolean | Allows shadowing unsafe methods. |
| `shadowMinRequests` | integer | Minimum request count before evaluating shadow health. |
| `shadowMaxErrorRate` | number | Maximum accepted shadow error rate. |
| `shadowMaxLatencyDelta` | duration string | Maximum accepted shadow latency delta. |

## Protocol Options

`protocol` can be `http`, `graphql`, `grpc`, `grpc-web`, `websocket`, `sse`, or `bff`.

| Field | Purpose |
|---|---|
| `protocol` | Selects protocol-specific handling. Empty generally behaves like HTTP. |
| `stripPath` | Controls whether the matched route path is stripped before proxying. |
| `validateSchema` | Schema validation reference or mode for compatible routes. |
| `cors` | CORS policy object. |
| `websocket` | WebSocket-specific options. |

SSE routes are streaming routes. They do not support buffered response body transforms such as `responseJSONFields`, response envelopes, body regex inspection, PII response checks, or `maxResponseBodyBytes`.

## CORS

```yaml
cors:
  allowedOrigins: ["https://app.example.com"]
  allowedMethods: ["GET", "POST", "OPTIONS"]
  allowedHeaders: ["Authorization", "Content-Type"]
  exposedHeaders: ["X-Trace-Id"]
  allowCredentials: true
  maxAge: 600
```

| Field | Purpose |
|---|---|
| `allowedOrigins` | Allowed origins. |
| `allowedMethods` | Allowed methods. |
| `allowedHeaders` | Allowed request headers. |
| `exposedHeaders` | Response headers visible to browser clients. |
| `allowCredentials` | Enables credentialed CORS requests. |
| `maxAge` | Browser preflight cache duration in seconds. |

## WebSocket

```yaml
websocket:
  timeout: "30s"
  bufferSize: 4096
  dnsRoundRobin: true
  injectHeaders:
    X-Gateway: "iket"
  allowedSubprotocols: ["graphql-transport-ws"]
  maxConnections: 1000
  maxConnectionsPerIP: 20
  rateLimit: 100
  insecureSkipVerify: false
```

| Field | Purpose |
|---|---|
| `timeout` | WebSocket timeout duration. |
| `bufferSize` | Buffer size for WebSocket traffic. |
| `dnsRoundRobin` | Enables DNS round-robin behavior. |
| `injectHeaders` | Headers injected into upstream WebSocket requests. |
| `allowedSubprotocols` | Allowed WebSocket subprotocols. |
| `maxConnections` | Total connection cap. |
| `maxConnectionsPerIP` | Per-IP connection cap. |
| `rateLimit` | Route WebSocket rate limit. |
| `insecureSkipVerify` | Allows self-signed upstream `wss://` certificates. Prefer proper trust in production. |

## GraphQL

GraphQL route fields apply when `protocol: graphql`.

| Field | Purpose |
|---|---|
| `graphqlAllowIntrospection` | Allows or blocks introspection. |
| `graphqlRequirePersistedQuery` | Requires persisted query usage. |
| `graphqlPersistedQueryField` | JSON field containing persisted query ID. |
| `graphqlAllowedPersistedQueries` | Persisted query allowlist. |
| `graphqlAllowedOperations` | Allowed operation names. |
| `graphqlAllowedVariables` | Variable allowlist. |
| `graphqlRequiredVariables` | Required variables. |
| `graphqlVariableRegex` | Regex validation for variable values. |
| `graphqlVariableAllowedValues` | Variable value allowlists. |
| `graphqlOperationPresets` | Named GraphQL operation policy presets. |
| `graphqlOperationPolicies` | Per-operation policies. |
| `graphqlOperationNameRequired` | Requires operation name. |
| `graphqlMaxDepth` | Maximum query depth. |
| `graphqlMaxFields` | Maximum field count. |

## AI And Agent Guardrails

AI policy fields can live directly on routes or inside `aiPolicyPresets`.

| Field | Purpose |
|---|---|
| `aiPolicyPreset` | Apply one named AI policy preset. |
| `aiPolicyPresetChain` | Apply several presets in order. |
| `aiPolicyPresets` | Route-local preset map. |
| `allowedModels` | Model allowlist. |
| `modelField` | JSON field where the model is read. |
| `allowedToolNames` | Tool name allowlist. |
| `toolField` | JSON field where tools are read. |
| `maxMessages` | Maximum message count. |
| `messagesField` | JSON field for messages. |
| `maxToolCalls` | Maximum tool call count. |
| `toolCallsField` | JSON field for tool calls. |
| `maxInputTokens` | Maximum requested input tokens. |
| `inputTokensField` | JSON field for input token budget. |
| `maxOutputTokens` | Maximum requested output tokens. |
| `outputTokensField` | JSON field for output token budget. |
| `allowedUpstreamHosts` | Allowed upstream hostnames. |
| `requireAgentContext` | Requires agent context headers. |
| `requireAgentApproval` | Requires approval headers. |
| `requireAgentApprovalSignature` | Requires signed approval metadata. |
| `requireAgentActionID` | Requires an action ID. |
| `rejectDuplicateAgentActionID` | Rejects replayed action IDs. |

## BFF Routes

Set `protocol: bff` and add a `bff` block when one public route should compose multiple upstream calls.

{% raw %}
```yaml
routes:
  - path: "/users/{user_id}/home"
    method: GET
    protocol: "bff"
    requireAuth: true
    bff:
      timeout: "2s"
      maxStepResponseBodyBytes: 1048576
      steps:
        - name: profile
          url: "http://profile-service/users/{{var.user_id}}"
          requireJSON: true
        - name: subscription
          url: "http://billing-service/users/{{var.user_id}}/subscription"
          required: false
      responseFields:
        user.id: "{{step.profile.json.id}}"
        plan: "{{step.subscription.json.plan}}"
        subscription_available: "{{step.subscription.ok}}"
      responseStatus: "200"
```
{% endraw %}

BFF fields:

| Field | Purpose |
|---|---|
| `enabled` | Optional boolean. Omitted means enabled. |
| `mode` | Execution mode, such as default parallel behavior or sequential mode. |
| `timeout` | Overall BFF timeout. |
| `includeMeta` | Includes BFF metadata in output. |
| `allowPartialResponse` | Allows partial response behavior. |
| `maxStepResponseBodyBytes` | Default step response body cap. |
| `requireRequestJSON` | Requires valid incoming JSON before steps run. |
| `requiredRequestJSONPaths` | Required incoming JSON paths. |
| `steps` | Upstream calls. |
| `responseFields` | Final JSON response mapping. |
| `responseHeaders` | Final response headers. |
| `responseStatus` | Final response status literal or template. |
| `partialResponseStatus` | Status used for partial responses. |

Step fields:

| Field | Purpose |
|---|---|
| `name` | Step name used by templates. |
| `method` | Step method. Defaults to GET. |
| `url` | Upstream URL template. |
| `required` | Omitted means true. Required failures fail the route. |
| `dependsOn` | Explicit dependencies. |
| `when`, `whenHeaders`, `whenQueryParams` | Conditional step execution. |
| `requireJSON`, `requiredJSONPaths` | JSON validation for step responses. |
| `timeout`, `retryCount`, `retryBackoff`, `retryJitter`, `retryStatusCodes`, `retryUnsafeMethods` | Step resilience controls. |
| `successStatusCodes` | Treat selected non-default statuses as successful. |
| `cacheTTL`, `cacheKey`, `cacheStatusCodes`, `cacheUnsafeMethods`, `staleIfError` | Step cache controls. |
| `maxResponseBodyBytes` | Step-specific response body cap. |
| `fallback` | Static fallback response. |
| `headers`, `queryParams`, `body` | Step request shaping. |

## Complete Production-Style Example

This example shows a split-file `service.yaml` that references policies and presets expected to live in `config.yaml`.

{% raw %}
```yaml
version: 1
services:
  - name: "Catalog Service"
    description: "Product catalog and inventory API"
    host: "https://catalog.internal.example.com"
    base_path: "/api"
    tags: ["catalog", "public"]
    group: "commerce"
    routes:
      - path: "/products/{product_id}"
        method: GET
        name: "Read Product"
        description: "Fetch one product by ID"
        protocol: "http"
        requireAuth: true
        auth_strategy: "workforce-jwt"
        authorization_policy: "catalog-readers"
        identityProjectionPreset: "upstream-default"
        enabled: true
        stripPath: false
        requestHeaders:
          X-Request-Id: "{{request_id}}"
        responseHeaders:
          X-Gateway: "iket"
        rateLimitPolicy:
          requestsPerSecond: 50
          burst: 100
          keyBy: "jwt_sub"
        concurrencyLimitPolicy:
          maxInFlight: 32
          keyBy: "jwt_sub"
          queueTimeout: "250ms"
          maxQueueDepth: 64
        retryCount: 2
        retryBackoff: "100ms"
        retryJitter: "25ms"
        retryStatusCodes: [502, 503, 504]
        backend:
          - url_pattern: "/v1/products/{product_id}"
            weight: 100
            timeout: "2s"
            healthCheckPath: "/health"
            healthInterval: "10s"

      - path: "/users/{user_id}/home"
        method: GET
        name: "Home BFF"
        protocol: "bff"
        requireAuth: true
        auth_strategy: "workforce-jwt"
        bff:
          timeout: "2s"
          maxStepResponseBodyBytes: 1048576
          steps:
            - name: profile
              url: "https://profile.internal.example.com/users/{{var.user_id}}"
              requireJSON: true
            - name: entitlements
              url: "https://billing.internal.example.com/users/{{var.user_id}}/entitlements"
              required: false
              cacheTTL: "30s"
          responseFields:
            user.id: "{{step.profile.json.id}}"
            user.name: "{{step.profile.json.name}}"
            entitlements: "{{step.entitlements.json.items}}"
            entitlements_available: "{{step.entitlements.ok}}"
```
{% endraw %}

## Template Tokens

Route and BFF templates use Iket tokens such as request IDs, path variables, headers, query params, request JSON fields, and step outputs.

{% raw %}
```yaml
requestHeaders:
  X-Tenant: "{{var.tenant_id}}"
  X-Trace: "{{request_id}}"
queryParams:
  user: "{{query.user}}"
responseHeaders:
  X-Upstream-Status: "{{response_status}}"
```
{% endraw %}

Common token families:

| Token Family | Meaning |
|---|---|
| {% raw %}`{{var.name}}`{% endraw %} | Route variable from the matched path. |
| {% raw %}`{{query.name}}`{% endraw %} | Incoming query parameter. |
| {% raw %}`{{header.Name}}`{% endraw %} | Incoming request header. |
| {% raw %}`{{request_id}}`{% endraw %} | Gateway request ID. |
| {% raw %}`{{request_body}}`{% endraw %} | Incoming request body. |
| {% raw %}`{{request.json.path}}`{% endraw %} | Incoming JSON field. |
| {% raw %}`{{response_status}}`{% endraw %} | Upstream response status. |
| {% raw %}`{{response_body}}`{% endraw %} | Upstream response body. |
| {% raw %}`{{response_header.Name}}`{% endraw %} | Upstream response header. |
| {% raw %}`{{step.name.json.path}}`{% endraw %} | BFF step JSON output. |

## Validation Notes

- `version` must be positive.
- Each service needs unique `name`, valid `host`, and at least one route.
- `base_path` must start with `/` when set.
- Each route needs `path` plus either `method` or `methods`.
- Route paths must start with `/`.
- Duplicate route paths within the same service are rejected after `base_path` is applied.
- Backend `url_pattern` is required for ordinary proxy routes.
- Backend `host`, when set, must be valid.
- Duration strings use Go duration syntax, such as `100ms`, `2s`, `5m`, or `1h`.
- `protocol` must be `http`, `graphql`, `grpc`, `grpc-web`, `websocket`, `sse`, or `bff`.
- `authorization_policy` must reference a policy from `config.yaml` and must be paired with route authentication.

## Related Docs

<div class="docs-grid">
  <article class="doc-card">
    <h3><a href="{{ '/docs/config-yaml-specification/' | relative_url }}">config.yaml Specification</a></h3>
    <p>Use this alongside service config to understand server, TLS, identity, storage, and plugin settings.</p>
  </article>
  <article class="doc-card">
    <h3><a href="{{ '/docs/routes-example/' | relative_url }}">Example service.yaml</a></h3>
    <p>Compare this reference against a smaller concrete service file.</p>
  </article>
  <article class="doc-card">
    <h3><a href="{{ '/docs/bff-routes/' | relative_url }}">BFF Routes</a></h3>
    <p>Go deeper on multi-step route composition and BFF templates.</p>
  </article>
</div>
