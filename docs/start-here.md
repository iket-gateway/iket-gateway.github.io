---
layout: page
title: Start Here
permalink: /docs/start-here/
description: The best entry path for first-time Iket users, from installation through CLI workflows and production readiness.
section: Start Here
---

# Start Here

Use this section when you are bringing Iket up for the first time, validating operator access, or preparing to move from a local setup into a safer production workflow.

<div class="callout">
  <p><strong>Recommended path:</strong> install Iket first, learn the CLI flow second, then move into production deployment or management API work once the basics are solid.</p>
</div>

<div class="start-rail">
  <article class="start-rail__card">
    <span class="start-rail__step">Step 1</span>
    <h3><a href="{{ '/docs/install/' | relative_url }}">Install Iket</a></h3>
    <p>Choose full gateway, CLI-only, source, or Docker setup based on where Iket will run and who will administer it.</p>
    <p class="start-rail__meta">Best for: first-time setup, bootstrap, local bring-up</p>
  </article>
  <article class="start-rail__card">
    <span class="start-rail__step">Step 2</span>
    <h3><a href="{{ '/docs/cli-commands/' | relative_url }}">Learn the CLI workflow</a></h3>
    <p>Verify connectivity, inspect gateway state, and understand diffs, proposals, and safe rollout habits before changing shared environments.</p>
    <p class="start-rail__meta">Best for: operators, diagnostics, safe changes</p>
  </article>
  <article class="start-rail__card">
    <span class="start-rail__step">Step 3</span>
    <h3><a href="{{ '/docs/production-deployment/' | relative_url }}">Prepare production deployment</a></h3>
    <p>Use the hardened deployment path when the gateway is moving beyond local validation and into a managed runtime environment.</p>
    <p class="start-rail__meta">Best for: deployment, hardening, production rollout</p>
  </article>
</div>

## Core Guides

<div class="docs-grid">
  <article class="doc-card">
    <h3><a href="{{ '/docs/install/' | relative_url }}">Installation Guide</a></h3>
    <p>Install Iket on local hosts, operator laptops, or Docker-based servers.</p>
  </article>
  <article class="doc-card">
    <h3><a href="{{ '/docs/cli-commands/' | relative_url }}">CLI Command Reference</a></h3>
    <p>Use the main `iket` command set for contexts, config changes, plugins, proposals, and operations.</p>
  </article>
  <article class="doc-card">
    <h3><a href="{{ '/docs/production-deployment/' | relative_url }}">Production Deployment</a></h3>
    <p>Deploy Iket with TLS, mTLS, Docker, and the operational pieces required for a secure setup.</p>
  </article>
  <article class="doc-card">
    <h3><a href="{{ '/docs/management-api-integration/' | relative_url }}">Management API Integration</a></h3>
    <p>Move into secure remote automation once the human-operated workflow is already working.</p>
  </article>
</div>
