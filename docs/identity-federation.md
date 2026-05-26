---
layout: docs_page
title: Identity Federation & Advanced Auth Roadmap
permalink: /docs/identity-federation/
section: Core Docs
section_order: 2
weight: 4
summary: Understand the recommended direction for federation-agnostic authentication, authorization, and IAM integration in Iket.
audience: [developer, operator]
topics: [authentication, authorization, iam, oidc, keycloak, architecture]
---

# Identity Federation & Advanced Auth Roadmap

Iket should support advanced authentication and authorization without locking deployments into one provider.

The right long-term shape is:

- built-in core auth modules inside Iket
- federation-agnostic IAM integration
- normalized identity inside the gateway
- provider-specific adapters only where they add value

## Recommendation

Iket should keep stable built-in modules for:

- API key authentication
- JWT verification
- OAuth2 token introspection
- mTLS identity extraction
- route authorization
- upstream identity propagation

Those are common gateway responsibilities and should not depend on one external provider.

## Current Phase 1 Foundation

The repository now includes an active Phase 1 foundation for this direction:

- top-level `identity.providers`
- top-level `identity.claim_maps`
- top-level `auth.strategies`
- top-level `authorization.policies`
- route-level `auth_strategy`
- route-level `authorization_policy`
- normalized internal identity context helpers
- runtime route enforcement for `auth_strategy`
- runtime route enforcement for `authorization_policy`
- normalized JWT claim propagation for tenant-aware and claim-aware authorization

The current runtime slice is intentionally provider-agnostic:

- `oidc_jwt` strategies can validate the authenticated principal source, issuer, and allowed audiences
- `oidc_jwt` strategies can now authenticate directly from provider `jwks_uri`
- `oidc_jwt` strategies can also discover `jwks_uri` from standard OIDC issuer metadata
- `oauth2_introspection` strategies can now authenticate directly from provider `introspection_uri`
- `oauth2_introspection`, `api_key`, `mtls`, and `anonymous` strategy types now have runtime matching hooks
- authorization policies can evaluate normalized scopes, roles, groups, client IDs, tenants, and mapped claims
- configured `claim_maps` can now shape JWT-derived roles, groups, scopes, client IDs, and tenant claims from provider-specific layouts
- configured `claim_maps` also shape introspection-derived identity from opaque-token IAM responses
- provider secret refs now support `env:`, `file:`, `literal:`, and raw inline values
- JWT secret and public-key material now support the same shared ref model across plugin and built-in gateway paths
- TLS certificate, private-key, and client-CA material now support the same shared ref model across built-in server TLS and TLS or mTLS plugins
- route-level `identityProjection` can now forward selected normalized identity fields to upstream services through explicit headers
- `identityProjectionPresets` can now be defined globally, at the service level, or on a route, then selected with `identityProjectionPreset` or layered with `identityProjectionPresetChain`
- built-in projection presets now include `minimal_user`, `service_identity`, and `tenant_only`, and configured presets can still override those names intentionally
- `iket gateway route-policy` and the matching management API inspection now surface the resolved identity projection preset stack, effective forwarded headers, and per-field source attribution

Full remote OIDC discovery, JWKS federation, and provider-specific helpers are still follow-on steps. The current code now provides both the contract and a real enforcement path, so future IAM integrations can extend behavior without redesigning the route model.

## Avoid Provider Lock-In

Iket should not model routes directly around one IAM product such as Keycloak.

Instead, the route contract should target:

- an `auth_strategy`
- an `authorization_policy`
- a normalized identity context

That way the same route can work with:

- Keycloak
- Auth0
- Okta
- Azure AD / Entra ID
- custom OIDC servers
- future enterprise IAM systems

## Suggested Config Model

```yaml
identity:
  providers:
    main:
      type: oidc
      issuer: "https://iam.example.com/realms/prod"
      jwks_uri: "https://iam.example.com/realms/prod/protocol/openid-connect/certs"
      introspection_uri: "https://iam.example.com/realms/prod/protocol/openid-connect/token/introspect"

auth:
  strategies:
    user_jwt:
      type: oidc_jwt
      provider: "main"
      claim_map: "default"

    service_token:
      type: oauth2_introspection
      provider: "main"
      claim_map: "default"

authorization:
  policies:
    orders_read:
      all_of:
        - scopes_any: ["orders.read"]

identityProjectionPresets:
  minimal_user:
    subjectHeader: "X-Iket-Subject"
    clientIDHeader: "X-Iket-Client-ID"

services:
  - name: "orders"
    identityProjectionPresets:
      tenant_headers:
        tenantHeader: "X-Iket-Tenant"

    routes:
      - path: "/orders"
        method: "GET"
        auth_strategy: "user_jwt"
        authorization_policy: "orders_read"
        identityProjectionPresetChain: ["minimal_user", "tenant_headers"]
        identityProjection:
          scopesHeader: "X-Iket-Scopes"
          attributeHeaders:
            region: "X-Iket-Region"
```

This lets Iket standardize safe upstream identity shapes through reusable presets while still allowing route-local overrides when a backend needs one extra field.

Iket also ships with small built-in presets for common cases:

- `minimal_user`
- `service_identity`
- `tenant_only`

Those defaults are meant to speed up onboarding, not lock configuration down. If a team defines the same preset name in gateway, service, or route config, the configured version wins.

Supported secret ref forms for provider-backed auth:

- `env:IKET_IAM_CLIENT_SECRET`
- `file:/run/secrets/iket-iam-client-secret`
- `literal:super-secret-for-local-dev`
- raw inline value for backward compatibility

## Normalize Identity Before Authorization

Different IAMs emit claims in different shapes. Iket should convert successful authentication into one internal model before it evaluates route policy.

Suggested normalized fields:

- `subject`
- `issuer`
- `provider`
- `authentication_method`
- `client_id`
- `tenant`
- `roles`
- `groups`
- `scopes`
- `attributes`

This keeps authorization rules stable even if the upstream IAM changes claim structure.

## Keycloak Should Be First-Class, But Not Hard-Coded

Keycloak is a strong first IAM integration target because it is common and self-hostable.

Iket should support Keycloak through:

- generic OIDC and OAuth2 provider support in core
- optional Keycloak-specific claim helpers for:
  - realm roles
  - client roles
  - realm-to-tenant mapping
  - future token exchange / UMA support

That gives a good Keycloak experience without making the whole gateway Keycloak-shaped.

## Multi-Scenario Support

Iket should plan for more than one auth pattern:

- user-to-service JWTs
- service-to-service tokens
- API clients with API keys
- operator/admin access
- tenant-aware partner access

Those should be different strategies, not one overloaded auth block.

## Authorization Direction

Iket should grow toward layered, composable policy checks such as:

- `scopes_any`
- `scopes_all`
- `roles_any`
- `groups_any`
- `client_ids`
- `tenants`
- `claims_match`
- `all_of`
- `any_of`
- `not`

That is a better fit for federated IAM than only checking one raw scope field.

## Upstream Identity Propagation

After Iket authenticates and authorizes a request, it should be able to forward a controlled identity projection upstream.

Examples:

- `X-Iket-Subject`
- `X-Iket-Tenant`
- `X-Iket-Roles`
- `X-Iket-Scopes`
- `X-Iket-Client-ID`

This should be explicit and minimal so the gateway does not leak more identity detail than needed.

## Recommended Roadmap

### Phase 1

- add `identity.providers`
- add `auth.strategies`
- add normalized identity context
- add generic OIDC JWT validation

### Phase 2

- add OAuth2 introspection strategy
- add claim-map configuration
- add route authorization policies

### Phase 3

- add Keycloak helper mappings
- add tenant-aware authorization
- add controlled upstream identity propagation

### Phase 4

- add external PDP integration
- add richer policy composition
- add advanced delegation and token exchange scenarios

## Practical Direction for Iket

If you want Iket to be strong in enterprise and multi-team environments, the best next step is not “add one Keycloak plugin and stop”.

The better move is:

- build a stable internal auth architecture
- make federation generic
- make Keycloak excellent as the first serious adapter
- keep the route and policy model provider-neutral

That gives Iket room to integrate with existing IAM today and different IAM systems later.
