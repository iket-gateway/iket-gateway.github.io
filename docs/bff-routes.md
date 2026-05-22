---
layout: docs_page
title: BFF Routes
permalink: /docs/bff-routes/
section: Core Docs
section_order: 2
weight: 6
summary: Compose multiple upstream calls into a client-focused Backend-for-Frontend response without changing simple proxy routes.
audience: [developer, operator]
topics: [routing, bff, composition]
---

# BFF Routes

BFF routes are optional route-level composition blocks. Use them when a frontend, mobile app, partner SDK, or workflow client needs one stable response made from several internal APIs.

Simple proxy routes do not change. A route only becomes a BFF route when it declares `bff`.

## Basic Example

```yaml
version: 1
services:
  - name: "App BFF"
    host: "http://gateway.local"
    routes:
      - path: "/app/users/{user_id}/home"
        method: GET
        protocol: bff
        requireAuth: true
        bff:
          timeout: 2s
          maxStepResponseBodyBytes: 1048576
          steps:
            - name: profile
              url: "http://profile-service/users/{{var.user_id}}"
              retryCount: 1
              retryBackoff: 100ms
              cacheTTL: 30s
              headers:
                X-Request-Id: "{{request_id}}"
            - name: subscription
              url: "http://billing-service/users/{{var.user_id}}/subscription"
              required: false
          responseFields:
            user.id: "{{step.profile.json.id}}"
            user.name: "{{step.profile.json.name}}"
            plan: "{{step.subscription.json.plan}}"
            subscription_available: "{{step.subscription.ok}}"
          responseStatus: "200"
          responseHeaders:
            Cache-Control: "private, max-age=30"
            X-BFF-User: "{{step.profile.json.id}}"
```

## Behavior

- `steps` are called in parallel by default.
- `mode: sequential` runs steps in order, which is useful when later steps use earlier step output.
- `required` defaults to `true`; required failures return `502`.
- Step templates that reference `{{step.name...}}` automatically wait for that step while unrelated steps still run in parallel.
- `successStatusCodes` lets a step treat domain-valid statuses such as `404` as successful.
- Optional step failures can still be reflected with fields such as `{{step.subscription.ok}}`.
- `responseFields` builds the final JSON response with the same dotted field style used by route transforms.
- `responseStatus` defaults to `200`; set it to a literal status or a template when a BFF command needs another success code.
- `responseHeaders` sets final client response headers after templates are resolved.
- `maxStepResponseBodyBytes` caps buffered upstream step bodies; each step can override it with `maxResponseBodyBytes`.
- `bff.enabled` can be changed through live config updates or provider reloads without restarting the gateway.

## Step Dependencies

When a step template references another step with `{{step.name...}}`, Iket infers that dependency and waits for the referenced step before resolving the template. Use `dependsOn` when a step must wait even though it does not reference the dependency directly.

```yaml
bff:
  steps:
    - name: profile
      url: "http://profile-service/users/{{var.user_id}}"
    - name: entitlements
      url: "http://billing-service/users/{{var.user_id}}/entitlements"
    - name: recommendations
      url: "http://recommendation-service/users/{{step.profile.json.id}}?tier={{step.entitlements.json.tier}}"
```

In default parallel mode, steps without dependencies start immediately. A dependent step waits for every inferred or listed dependency, then receives those dependency results as template inputs. Dependencies must reference existing steps and cannot contain cycles.

Iket validates `{{step.name...}}` template references at load time. Response templates can reference any step. Step-local templates such as `url`, `headers`, `queryParams`, `body`, `when`, `cacheKey`, and `fallback` infer dependencies automatically in default parallel mode, or can reference earlier steps when `mode: sequential` is used.

## Conditional Steps

Conditional steps let one BFF shape serve basic and richer clients without duplicating routes. A skipped step is treated as intentionally absent, even when `required` is left at its default.

```yaml
bff:
  steps:
    - name: recommendations
      url: "http://recommendation-service/users/{{var.user_id}}"
      when: "{{query.include_recs}} == true"
      whenHeaders:
        X-Client-Mode: full
      whenQueryParams:
        client: mobile
```

`when` supports truthy values and simple `==` / `!=` comparisons after templates are resolved. `whenHeaders` and `whenQueryParams` match the incoming request exactly after their expected values are resolved.

## Response Headers

BFF routes can set client-facing headers from request tokens or completed step results.

```yaml
bff:
  responseHeaders:
    Cache-Control: "private, max-age=30"
    Vary: "Authorization, X-Client-Mode"
    X-BFF-User: "{{step.profile.json.id}}"
    X-Request-Id: "{{request_id}}"
```

Iket still sets its normal JSON and gateway headers by default. Values in `responseHeaders` are applied last, so a BFF route can intentionally override them.

## Response Status

BFF routes return `200` by default after all required steps succeed. Use `responseStatus` when the composed endpoint should expose another final status.

```yaml
bff:
  responseStatus: "{{step.create.status}}"
  steps:
    - name: create
      method: POST
      url: "http://workflow-service/jobs"
```

Static values are validated at load time. Templated values are resolved after all steps finish and must produce a `2xx` to `5xx` HTTP status code.

## Request JSON Requirements

Use `requireRequestJSON` when a BFF route needs a valid incoming JSON body before any upstream steps run. Add `requiredRequestJSONPaths` when specific request fields are required by `{{request.json.path}}` templates.

```yaml
bff:
  requireRequestJSON: true
  requiredRequestJSONPaths: [sku, customer.id]
  steps:
    - name: create
      method: POST
      url: "http://order-service/orders/{{request.json.customer.id}}"
      body: 'json:{"sku":"{{request.json.sku}}"}'
```

If the body is empty, invalid JSON, or missing a required path, Iket returns `400` and does not call upstream steps. Setting any required request path also implies that the body must be valid JSON.

## Step Body Limits

BFF steps buffer upstream responses so templates can read `body`, `json`, and `header` values. Set a default cap to keep one dependency from making the gateway buffer too much data.

```yaml
bff:
  maxStepResponseBodyBytes: 1048576
  steps:
    - name: profile
      url: "http://profile-service/users/{{var.user_id}}"
    - name: report
      url: "http://report-service/users/{{var.user_id}}/summary"
      maxResponseBodyBytes: 262144
```

If a step exceeds its limit, Iket treats it as a step failure. Stale cache or a configured fallback can still recover the response.

## Step JSON Requirements

Use `requireJSON` when a step must return valid JSON for composition.

```yaml
bff:
  steps:
    - name: profile
      url: "http://profile-service/users/{{var.user_id}}"
      requireJSON: true
      requiredJSONPaths: [id, plan.tier]
      fallback:
        status: 200
        body: 'json:{"id":"{{var.user_id}}","source":"fallback"}'
```

Without `requireJSON`, Iket still parses valid JSON when present, but non-JSON bodies remain available through `{{step.name.body}}`. With `requireJSON: true`, empty or invalid JSON is treated as a step failure, so stale cache or fallback can recover it.

`requiredJSONPaths` validates that specific JSON fields exist. Setting any required path also implies that the body must be valid JSON.

## Step Retries

Retries are opt-in per step and use the same naming style as normal proxy routes.

```yaml
bff:
  steps:
    - name: profile
      url: "http://profile-service/users/{{var.user_id}}"
      retryCount: 2
      retryBackoff: 100ms
      retryJitter: 50ms
      retryStatusCodes: [429, 502, 503, 504]
```

GET, HEAD, OPTIONS, TRACE, PUT, and DELETE steps can retry by default. Set `retryUnsafeMethods: true` only when a POST/PATCH step is safe for your upstream.

## Step Success Statuses

By default, BFF steps treat `2xx` and `3xx` responses as successful. Configure `successStatusCodes` when an upstream uses a non-2xx status as a valid domain result.

```yaml
bff:
  steps:
    - name: profile
      url: "http://profile-service/users/{{var.user_id}}"
      successStatusCodes: [200, 404]
```

The configured status policy affects required-step failures, dependency handling, fallback decisions, metrics outcomes, and `{{step.name.ok}}`.

## Step Fallbacks

Fallbacks let a BFF keep a stable response shape when a dependency fails after retries and stale cache cannot recover it.

```yaml
bff:
  steps:
    - name: recommendations
      url: "http://recommendation-service/users/{{var.user_id}}"
      required: true
      fallback:
        status: 200
        body: 'json:{"items":[],"source":"fallback"}'
        headers:
          X-Fallback-Source: static
```

Fallbacks apply to transport errors, response body read errors, and upstream failure status codes. A fallback with a 2xx status clears the step error, so even a required step can recover.

## Step Cache

Small per-step TTL caches reduce fan-out latency for repeated BFF reads. Cache is opt-in and in-memory per gateway replica.

```yaml
bff:
  steps:
    - name: profile
      url: "http://profile-service/users/{{var.user_id}}"
      cacheTTL: 30s
      staleIfError: 5m
      cacheKey: "profile:{{var.user_id}}:{{header.X-Tenant}}"
      cacheStatusCodes: [200]
```

Without `cacheKey`, Iket keys the cache by route, step, method, and resolved URL. GET and HEAD steps can cache by default. Set `cacheUnsafeMethods: true` only when another method is safe to cache for that upstream.

Concurrent misses for the same cache key are coalesced automatically. Iket runs one upstream call and shares the result with waiting requests, which helps prevent cache stampedes after cold starts or TTL expiry.

`staleIfError` keeps the last cached response available after `cacheTTL` expires. If the refresh attempt fails or returns a failure status, Iket serves the stale response until the `staleIfError` window closes.

## Observability

Each BFF upstream step emits structured logs with route, step, required flag, status, outcome, and latency. The metrics endpoint also exposes:

| Metric | Meaning |
|---|---|
| `gateway_bff_step_requests_total` | Count of BFF upstream step executions by route, step, status, required flag, and outcome |
| `gateway_bff_step_duration_seconds` | Histogram of BFF upstream step latency with the same labels |

## Template Tokens

| Token | Meaning |
|---|---|
| `{{var.name}}` | Route variable from the matched path |
| `{{query.name}}` | Incoming query parameter |
| `{{header.Name}}` | Incoming request header |
| `{{request_id}}` | Gateway request ID |
| `{{request_body}}` | Incoming request body for forwarding |
| `{{request.json.path}}` | Value from the incoming JSON request body |
| `{{step.name.status}}` | Upstream HTTP status |
| `{{step.name.ok}}` | `true` when the step completed with a 2xx or 3xx response |
| `{{step.name.body}}` | Raw upstream response body |
| `{{step.name.attempts}}` | Number of attempts used by the step, including retries |
| `{{step.name.cache_hit}}` | `true` when the step was served from the local BFF cache |
| `{{step.name.cache_stale}}` | `true` when the step used stale cache after an upstream failure |
| `{{step.name.coalesced}}` | `true` when the step reused an in-flight upstream call for the same cache key |
| `{{step.name.fallback}}` | `true` when the step used its configured fallback |
| `{{step.name.skipped}}` | `true` when the step did not run because its conditions did not match |
| `{{step.name.json.path}}` | Value from an upstream JSON response |
| `{{step.name.header.Name}}` | Upstream response header |

JSON paths use dot notation with optional array indexes, for example `{{request.json.items[0].sku}}` or `{{step.profile.json.roles[0]}}`. Use `[]` to collect array values, such as `{{request.json.items[].sku}}` or `{{step.catalog.json.products[].id}}`.

Add `| default:value` when optional data should fall back instead of resolving to an empty value, for example `{{request.json.locale | default:en-US}}` or `{{step.profile.json.name | default:Guest}}`. Numeric, boolean, and `null` defaults keep their JSON type when the whole template is a single value.

Use encoding filters when inserting dynamic values into structured strings: `| json` writes a JSON literal, `| urlquery` escapes a query value, and `| urlpath` escapes a path segment. Filters can be chained, for example `{{request.json.name | default:Guest | json}}`.

Wildcard arrays can be reshaped with `| len` or `| join:separator`, for example `{{request.json.items[].sku | len}}` or `{{step.catalog.json.products[].id | join:, | urlquery}}`.

## Growth Path

Start with one BFF route and a few read-only steps. As the use case grows, add route auth, scopes, rate/concurrency policies, timeout budgets, allowed upstream hosts, and optional fallback steps. Enterprise deployments can then layer proposals, canary rollout, policy alerts, usage metering, and dedicated plugins around the same route shape.
