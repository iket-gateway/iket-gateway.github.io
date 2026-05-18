---
layout: page
title: Iket API Gateway
---

Iket is a lightweight, extensible API gateway written in Go for teams that need modern edge controls without dragging in a heavy platform. It combines secure remote administration, hot-reloadable plugins, protocol-aware routing, and operator-friendly rollout tooling in one focused gateway.

[Read the full documentation](/docs/) | [View the GitHub project](https://github.com/bhangun/iket) | [Open the latest releases](https://github.com/bhangun/iket/releases)

The documentation now lives on this website, so installation, operations, plugin, and API guides are maintained here as first-class website pages.

## Why Iket

- **Gateway features that ship together**: reverse proxying, route policies, JWT and client auth, rate and concurrency limits, WebSockets, GraphQL, gRPC, SSE, and structured observability.
- **Secure-by-default administration**: mTLS, TLS 1.3, enrollment flows, certificate tooling, and environment-aware CLI contexts for remote operations.
- **Safe change management**: config diffs, proposals, approvals, canary rollouts, shadow traffic checks, revision history, and alert-ready queue views.
- **Built for extension**: hot-reloadable plugins and middleware hooks let you add custom auth, policy, storage, or gateway behavior without forking the core.

## Quickstart

Install the gateway or CLI:

```bash
curl -fsSL https://raw.githubusercontent.com/bhangun/iket/main/scripts/install.sh | bash
```

Start the gateway:

```bash
iket-server --config ~/.iket/config.yaml
```

Connect the CLI:

```bash
iket setup
iket gateway status
```

For installation modes, Docker setup, route configuration, proposal workflows, and plugin examples, head to the [documentation hub](/docs/).

## Core Capabilities

### Traffic and protocol management

Iket supports HTTP, GraphQL, gRPC, gRPC-Web, WebSocket, and SSE traffic with service-based routing, backend weighting, retries, hedging, adaptive latency routing, and shadow traffic evaluation.

### Policy and AI guardrails

Routes can enforce model allowlists, tool allowlists, request and response transforms, JSON shaping, prompt and output limits, upstream host pinning, content regex checks, and PII blocking before traffic leaves the gateway.

### Rollouts and operations

Operators can preview changes, propose them for later approval, promote them between environments, run canaries, inspect queue urgency, replay notifications, and restore previous revisions when needed.

### Deployment flexibility

Iket supports local installs, Docker-based deployments, and remote administration with the same CLI experience. File-backed and PostgreSQL-backed configurations are both supported depending on how you want to run the gateway.
