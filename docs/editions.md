---
layout: docs_page
title: Iket Editions
permalink: /docs/editions/
section: Core Docs
section_order: 2
weight: 4
summary: See how community and enterprise capabilities are separated and how edition-aware gates shape the public Iket surface.
audience: [developer, operator]
topics: [product, editions, licensing]
---

# Iket Editions

Iket Community is the default open-source profile. It should stay focused on
single-tenant gateway operation, local management, community plugins, baseline
security, and basic observability.

Iket Enterprise should extend the community profile from a separate repository
or enterprise-only package. The shared core exposes edition metadata and
capability gates so enterprise modules can integrate without forking gateway,
config, or management API internals.

## Community Profile

The community binary uses `app.CommunityEditionInfo()` by default. Community
capabilities include:

- `gateway.runtime`
- `config.providers`
- `plugins.community`
- `management.local`
- `observability.basic`
- `security.baseline`

## Enterprise Profile

Enterprise builds can register a richer profile during package initialization:

```go
package enterprise

import "github.com/bhangun/iket/pkg/app"

func init() {
	app.RegisterEdition(app.EnterpriseEditionInfo(
		app.Capability{
			Key:         "commercial.usage_export",
			Name:        "Usage Export",
			Category:    "commercial",
			Description: "Export tenant usage for finance systems.",
		},
	))
}
```

The enterprise profile keeps community capabilities and adds shared enterprise
capability keys such as:

- `tenant.workspaces`
- `security.enterprise_rbac`
- `governance.audit_trails`
- `commercial.billing`
- `gateway.high_availability`
- `observability.advanced`

## Capability Gates

Use `app.RequireCapability(key)` in core logic, or
`managementAPI.WithCapability(key, handler)` for management API routes. Missing
capabilities should fail closed and return a structured `403 FORBIDDEN` response.

The management API exposes the active profile at `GET /api/v1/gateway/edition`.
The response includes both the flat `capabilities` list and derived
`capability_categories` summaries so admin UIs can render grouped community or
enterprise feature sets without duplicating grouping logic.

For capability-focused screens or scripts, use `GET /api/v1/gateway/capabilities`
or `GET /api/v1/gateway/capabilities?category=gateway`. The CLI equivalent is
`iket gateway capabilities --category gateway`.

Enterprise packages can also mount management routes without modifying the
community router. The registered module catalog is available at
`GET /api/v1/gateway/extensions`, one module can be inspected with
`GET /api/v1/gateway/extensions/{name}`, and the CLI equivalents are
`iket gateway extensions` and `iket gateway extension <name>`. The catalog can
be filtered with `?supported=false`, `?capability=commercial.billing`,
`?category=commercial`, or `?tag=billing`, which is useful for admin screens
that show installed enterprise modules separately from modules available in the
active edition. Extension metadata can declare one primary `Capability` for
older clients, a `Capabilities` list when a module needs multiple gates, and
optional `Category`/`Tags` values for UI and CLI discovery. Catalog responses
also include a top-level `support` summary plus `extension_categories` and
`extension_tags` summaries so clients can render edition readiness and grouped
extension browsers without rebuilding those facets themselves.

```go
package enterprise

import (
	"net/http"

	managementapi "github.com/bhangun/iket/pkg/api"
	"github.com/bhangun/iket/pkg/app"
	"github.com/gorilla/mux"
)

func init() {
	managementapi.RegisterManagementRouteExtensionInfo(managementapi.ManagementRouteExtensionInfo{
		Name:        "enterprise.billing",
		Description: "Billing, subscription, and usage management routes.",
		Category:    "commercial",
		Tags:        []string{"billing", "usage"},
		Capability:  app.CapabilityBillingIntegration,
		Capabilities: []string{
			app.CapabilityBillingIntegration,
			app.CapabilityAuditTrails,
		},
	}, func(api *managementapi.ManagementAPI, router *mux.Router) {
		router.HandleFunc("/enterprise/billing", api.WithCapabilities([]string{
			app.CapabilityBillingIntegration,
			app.CapabilityAuditTrails,
		}, billingHandler)).Methods(http.MethodGet)
	})
}
```
