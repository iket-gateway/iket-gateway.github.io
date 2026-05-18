---
layout: docs_page
title: Example service.yaml
permalink: /docs/routes-example/
section: Core Docs
section_order: 2
weight: 6
summary: Use a concrete `service.yaml` example as a reference for routes, methods, backends, and common path patterns.
audience: [developer, operator]
topics: [routing, examples, configuration]
---

# Example service.yaml

```yaml
version: 1
services:
  - name: "Example Service"
    host: "http://localhost:7112"
    routes:
      - path: "/{rest:.*}"
        method: GET
        requireAuth: false
      - path: "/swagger-ui/{rest:.*}"
        method: GET
        requireAuth: false
      - path: "/api/*"
        method: GET
        requireAuth: false
```
