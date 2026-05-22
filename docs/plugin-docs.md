---
layout: page
title: Plugin Docs
permalink: /docs/plugin-docs/
description: Extension guides for building, operating, and understanding Iket plugins and middleware patterns.
section: Plugin Docs
---

# Plugin Docs

Use this section when you want to extend Iket behavior instead of only configuring built-in gateway capabilities.

<div class="callout">
  <p><strong>Suggested path:</strong> start with the plugin quickstart, move into the architecture guide next, then compare examples and middleware patterns once your first plugin is working.</p>
</div>

<div class="docs-grid">
  <article class="doc-card">
    <h3><a href="{{ '/docs/plugin-quickstart/' | relative_url }}">Plugin Quickstart</a></h3>
    <p>Build the smallest useful plugin first so the load, naming, and validation path is clear.</p>
  </article>
  <article class="doc-card">
    <h3><a href="{{ '/docs/plugin-development/' | relative_url }}">Plugin Development</a></h3>
    <p>Dive into interfaces, lifecycle hooks, configuration parsing, health reporting, and reloadable behavior.</p>
  </article>
  <article class="doc-card">
    <h3><a href="{{ '/docs/plugin-examples/' | relative_url }}">Plugin Examples</a></h3>
    <p>Compare example implementations and patterns once your first extension is in place.</p>
  </article>
  <article class="doc-card">
    <h3><a href="{{ '/docs/middleware-plugins/' | relative_url }}">Middleware Plugins</a></h3>
    <p>Review middleware-oriented extension patterns and where they fit inside the request path.</p>
  </article>
</div>
