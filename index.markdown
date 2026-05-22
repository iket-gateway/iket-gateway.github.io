---
layout: page
title: Iket API Gateway
---

Iket is a lightweight, extensible API gateway written in Go for teams that need modern edge controls without dragging in a heavy platform. It combines secure remote administration, hot-reloadable plugins, protocol-aware routing, and operator-friendly rollout tooling in one focused gateway.

The documentation now lives on this website, so installation, operations, plugin, and API guides are maintained here as first-class website pages.

<div class="release-banner">
  <span class="release-banner__label">Latest release</span>
  <strong>{{ site.latest_version }}</strong>
  <span>See published builds and release notes on GitHub.</span>
  <a href="{{ site.releases_url }}">Open releases</a>
</div>

## What’s New

<div class="changefeed-grid">
  {% for item in site.data.changefeed limit:3 %}
    <article class="changefeed-card">
      <span class="changefeed-card__label">{{ item.label }}</span>
      {% if item.url contains '://' %}
        <h3><a href="{{ item.url }}">{{ item.title }}</a></h3>
      {% else %}
        <h3><a href="{{ item.url | relative_url }}">{{ item.title }}</a></h3>
      {% endif %}
      <p>{{ item.summary }}</p>
    </article>
  {% endfor %}
</div>

<div class="stats-grid">
  <div class="stat-card">
    <h3>Protocol Coverage</h3>
    <p>Run HTTP, GraphQL, gRPC, gRPC-Web, WebSocket, and SSE traffic under one gateway.</p>
  </div>
  <div class="stat-card">
    <h3>Operational Safety</h3>
    <p>Use mTLS, proposals, diffs, revisions, canaries, and shadow checks to reduce rollout risk.</p>
  </div>
</div>

## Why Iket

<div class="feature-grid">
  <div class="feature-card">
    <h3>Gateway features that ship together</h3>
    <p>Reverse proxying, route policies, JWT and client auth, rate and concurrency limits, and strong observability out of the box.</p>
  </div>
  <div class="feature-card">
    <h3>Secure-by-default administration</h3>
    <p>mTLS, TLS 1.3, enrollment flows, certificate tooling, and environment-aware CLI contexts for remote operations.</p>
  </div>
  <div class="feature-card">
    <h3>Safe change management</h3>
    <p>Config diffs, approvals, canary rollouts, shadow traffic checks, revision history, and alert-ready queue views.</p>
  </div>
  <div class="feature-card">
    <h3>Built for extension</h3>
    <p>Hot-reloadable plugins and middleware hooks let you add custom auth, policy, storage, or gateway behavior without forking core code.</p>
  </div>
</div>

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

For installation modes, Docker setup, route configuration, proposal workflows, and plugin examples, head to the [documentation hub]({{ '/docs/' | relative_url }}).

<div class="callout">
  <p><strong>Best starting path:</strong> install the gateway, connect the CLI, then move into the docs hub for deployment, management, and plugin guides.</p>
</div>

## Core Capabilities

<div class="timeline-list">
  <div class="timeline-item">
    <h3>Traffic and protocol management</h3>
    <p>Iket supports HTTP, GraphQL, gRPC, gRPC-Web, WebSocket, and SSE traffic with service-based routing, backend weighting, retries, hedging, adaptive latency routing, and shadow traffic evaluation.</p>
  </div>
  <div class="timeline-item">
    <h3>Policy and AI guardrails</h3>
    <p>Routes can enforce model allowlists, tool allowlists, request and response transforms, JSON shaping, prompt and output limits, upstream host pinning, content regex checks, and PII blocking before traffic leaves the gateway.</p>
  </div>
  <div class="timeline-item">
    <h3>Rollouts and operations</h3>
    <p>Operators can preview changes, propose them for later approval, promote them between environments, run canaries, inspect queue urgency, replay notifications, and restore previous revisions when needed.</p>
  </div>
  <div class="timeline-item">
    <h3>Deployment flexibility</h3>
    <p>Iket supports local installs, Docker-based deployments, and remote administration with the same CLI experience. File-backed and PostgreSQL-backed configurations are both supported depending on how you want to run the gateway.</p>
  </div>
</div>

## Request Flow

<div class="architecture-flow">
  <article class="architecture-flow__stage">
    <span class="architecture-flow__step">01</span>
    <h3>Ingress and identity</h3>
    <p>Client traffic enters through one gateway edge where TLS, client identity, and route matching are established before the request moves deeper into policy evaluation.</p>
  </article>
  <div class="architecture-flow__connector" aria-hidden="true"></div>
  <article class="architecture-flow__stage">
    <span class="architecture-flow__step">02</span>
    <h3>Policy and guardrails</h3>
    <p>Iket can apply auth, rate and concurrency limits, transforms, tool and model allowlists, payload checks, and PII or content filters before upstream calls are allowed.</p>
  </article>
  <div class="architecture-flow__connector" aria-hidden="true"></div>
  <article class="architecture-flow__stage">
    <span class="architecture-flow__step">03</span>
    <h3>Routing and traffic strategy</h3>
    <p>Traffic can be sent through weighted backends, retries, hedging, shadow evaluation, and canary paths so rollout behavior is part of the request path instead of a separate operational layer.</p>
  </article>
  <div class="architecture-flow__connector" aria-hidden="true"></div>
  <article class="architecture-flow__stage">
    <span class="architecture-flow__step">04</span>
    <h3>Operator feedback loop</h3>
    <p>Diffs, revisions, proposals, alerts, and CLI-driven inspection close the loop so operators can safely understand changes, validate outcomes, and roll forward or back.</p>
  </article>
</div>

<div class="architecture-notes">
  <div class="architecture-notes__item">
    <h3>Why this matters</h3>
    <p>Iket treats request handling and change safety as one system. That is the biggest difference between a simple reverse proxy and an operational gateway.</p>
  </div>
  <div class="architecture-notes__item">
    <h3>What teams get</h3>
    <p>A cleaner path from local setup to production rollout, with the same CLI, the same control model, and the same extension seams across environments.</p>
  </div>
</div>

## Jump In

<div class="docs-grid">
  <div class="doc-card">
    <h3><a href="{{ '/docs/install/' | relative_url }}">Install Iket</a></h3>
    <p>Set up the gateway locally, in Docker, or on a remote host.</p>
  </div>
  <div class="doc-card">
    <h3><a href="{{ '/docs/cli-commands/' | relative_url }}">Use the CLI</a></h3>
    <p>Manage contexts, configs, services, routes, plugins, and rollout workflows.</p>
  </div>
  <div class="doc-card">
    <h3><a href="{{ '/docs/plugin-quickstart/' | relative_url }}">Build Plugins</a></h3>
    <p>Create middleware and extensions without forking the gateway core.</p>
  </div>
  <div class="doc-card">
    <h3><a href="{{ '/docs/management-api-integration/' | relative_url }}">Secure Admin Access</a></h3>
    <p>Integrate the management API and remote administration safely with mTLS.</p>
  </div>
</div>

## Featured Docs

<div class="featured-docs-grid">
  {% for item in site.data.featured_docs %}
    <article class="featured-doc">
      <span class="featured-doc__tag">{{ item.tag }}</span>
      <h3><a href="{{ item.url | relative_url }}">{{ item.title }}</a></h3>
      <p>{{ item.summary }}</p>
    </article>
  {% endfor %}
</div>

## When Iket Fits Best

<div class="feature-grid">
  <div class="feature-card">
    <h3>You want operational control</h3>
    <p>Choose Iket when rollout safety, remote administration, and change visibility matter as much as raw request routing.</p>
  </div>
  <div class="feature-card">
    <h3>You need protocol flexibility</h3>
    <p>Choose Iket when one gateway has to handle classic APIs, internal services, and newer agent or model-facing traffic patterns.</p>
  </div>
  <div class="feature-card">
    <h3>You plan to extend behavior</h3>
    <p>Choose Iket when custom middleware, plugins, and policy controls are part of the real deployment plan, not an afterthought.</p>
  </div>
  <div class="feature-card">
    <h3>You prefer practical infrastructure</h3>
    <p>Choose Iket when you want a focused gateway with modern controls, without dragging in a full heavyweight platform just to ship changes safely.</p>
  </div>
</div>

## How It Feels

<div class="comparison-grid">
  <div class="comparison-card comparison-card--muted">
    <h3>Typical gateway pain</h3>
    <ul>
      <li>Routing works, but admin workflows feel bolted on.</li>
      <li>Change safety depends on manual process outside the gateway.</li>
      <li>Extensibility means awkward forks or brittle custom patches.</li>
    </ul>
  </div>
  <div class="comparison-card comparison-card--accent">
    <h3>Iket approach</h3>
    <ul>
      <li>Routing, policy, admin, and rollout tooling are designed as one surface.</li>
      <li>Diffs, proposals, canaries, revisions, and mTLS are part of the operating model.</li>
      <li>Plugin and middleware extension paths are first-class from the start.</li>
    </ul>
  </div>
</div>

## Community And Commercial

<div class="stats-grid">
  <div class="stat-card">
    <h3>Community by default</h3>
    <p>The core project stays open source and accessible for teams that want a capable gateway without lock-in.</p>
  </div>
  <div class="stat-card">
    <h3>Commercial-friendly foundation</h3>
    <p>The licensing and product shape support commercial deployment, managed usage, and future enterprise packaging without splitting the core story.</p>
  </div>
</div>

## Design Principles

<div class="principles-grid">
  <div class="principle-card">
    <h3>Secure by default</h3>
    <p>Administrative access, rollout safety, and certificate handling should not be optional afterthoughts.</p>
  </div>
  <div class="principle-card">
    <h3>Operationally legible</h3>
    <p>Operators should be able to see what changed, why it changed, and how to roll forward or back.</p>
  </div>
  <div class="principle-card">
    <h3>Extensible without drama</h3>
    <p>Customization should happen through plugins and clear extension seams, not endless forks.</p>
  </div>
  <div class="principle-card">
    <h3>Modern traffic aware</h3>
    <p>A gateway should serve both classic API workloads and newer AI or agent-facing patterns from one coherent control surface.</p>
  </div>
</div>

## Getting Started Checklist

<div class="checklist-grid">
  <div class="checklist-card">
    <span class="checklist-card__step">1</span>
    <div>
      <h3>Install the gateway or CLI</h3>
      <p>Start with the <a href="{{ '/docs/install/' | relative_url }}">installation guide</a> and choose full gateway, CLI-only, or source mode.</p>
    </div>
  </div>
  <div class="checklist-card">
    <span class="checklist-card__step">2</span>
    <div>
      <h3>Connect and verify access</h3>
      <p>Use <a href="{{ '/docs/cli-commands/' | relative_url }}">CLI workflows</a> to set up a context and confirm the gateway is reachable.</p>
    </div>
  </div>
  <div class="checklist-card">
    <span class="checklist-card__step">3</span>
    <div>
      <h3>Review rollout-safe operations</h3>
      <p>Learn the proposal, diff, canary, and revision workflows before changing production state.</p>
    </div>
  </div>
  <div class="checklist-card">
    <span class="checklist-card__step">4</span>
    <div>
      <h3>Extend only when needed</h3>
      <p>Once the core gateway flow is stable, move to <a href="{{ '/docs/plugin-quickstart/' | relative_url }}">plugin quickstart</a> for custom behavior.</p>
    </div>
  </div>
</div>
