---
layout: docs_page
title: Plugin Quick Start Guide
permalink: /docs/plugin-quickstart/
section: Plugin Docs
section_order: 3
weight: 1
summary: Build a first custom plugin quickly and understand the smallest useful path to extending Iket behavior.
audience: [developer]
topics: [plugins, extension, quickstart]
---

# Plugin Quick Start Guide

This guide will help you create your first plugin for the Iket API Gateway in under 10 minutes.

<div class="doc-note">
  <p><strong>Use this guide when:</strong> you want the shortest path to a working plugin and do not need the full architecture details yet. It is the best first stop before the deeper plugin development guide.</p>
</div>

## Prerequisites

- Go 1.23 or later
- Basic understanding of Go and HTTP middleware
- Iket API Gateway installed

## Step 1: Create Plugin Directory

```bash
mkdir my-first-plugin
cd my-first-plugin
go mod init my-first-plugin
```

## Step 2: Create Your Plugin

<div class="doc-tip">
  <p><strong>Keep the first plugin small:</strong> start with simple request logging, headers, or validation behavior first. Once the load and lifecycle flow is clear, move on to more stateful or policy-heavy plugins.</p>
</div>

Create a file called `plugin.go`:

```go
package main

import (
    "fmt"
    "net/http"
    "sync"
)

// MyFirstPlugin implements a simple request logger
type MyFirstPlugin struct {
    enabled bool
    mu      sync.RWMutex
}

func (p *MyFirstPlugin) Name() string { return "my_first_plugin" }

func (p *MyFirstPlugin) Initialize(config map[string]interface{}) error {
    p.mu.Lock()
    defer p.mu.Unlock()
    p.enabled = true
    if enabled, ok := config["enabled"].(bool); ok {
        p.enabled = enabled
    }
    return nil
}

func (p *MyFirstPlugin) Middleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !p.enabled {
            next.ServeHTTP(w, r)
            return
        }
        fmt.Printf("Request: %s %s\n", r.Method, r.URL.Path)
        w.Header().Set("X-My-Plugin", "Hello from Iket!")
        next.ServeHTTP(w, r)
    })
}

var Plugin = &MyFirstPlugin{}
```

## Step 3: Build the Plugin

```bash
go build -buildmode=plugin -o my-first-plugin.so plugin.go
```

## Step 4: Configure the Gateway

<div class="doc-warning">
  <p><strong>Important:</strong> make sure the plugin name in your exported Go object, compiled `.so` file, and gateway configuration stay aligned. Most first-run plugin issues come from naming or path mismatches.</p>
</div>

Update your `config/config.yaml`:

```yaml
server:
  port: 8080
  pluginsDir: "./plugins"  # Directory for .so files

plugins:
  my_first_plugin:
    enabled: true
```

## Step 5: Load the Plugin

1. Create the plugins directory:
```bash
mkdir plugins
```

2. Copy your plugin:
```bash
cp my-first-plugin.so plugins/
```

3. Start the gateway:
```bash
iket-server --config config/config.yaml
```

## Step 6: Verify with `iket`

Use the CLI to verify that your plugin is loaded and active:

```bash
# List all plugins
iket plugin list
```

## Step 7: Test Your Plugin

Make a request through your gateway:

```bash
curl -i http://localhost:8080/some-route
```

You should see:
- The request logged in the gateway console.
- A custom header `X-My-Plugin: Hello from Iket!` in the response.

---

## 🛠️ Managing Plugins via CLI

Iket allows you to manage plugins in real-time using `iket`.

### Enable a Plugin
```bash
iket plugin enable my_first_plugin
```

### Disable a Plugin
```bash
iket plugin disable my_first_plugin
```

---

## Resources

- [Plugin Development Guide]({{ '/docs/plugin-development/' | relative_url }}) - Comprehensive guide.
- [Plugin Examples]({{ '/docs/plugin-examples/' | relative_url }}) - More examples and patterns.
- [Secure Administration]({{ '/docs/production-deployment/' | relative_url }}) - Learn how to secure your management API.

## Next Steps

<div class="docs-grid">
  <article class="doc-card">
    <h3><a href="{{ '/docs/plugin-development/' | relative_url }}">Go deeper into plugin architecture</a></h3>
    <p>Continue here when you need lifecycle hooks, health checks, reload behavior, and cleaner implementation patterns.</p>
  </article>
  <article class="doc-card">
    <h3><a href="{{ '/docs/plugin-examples/' | relative_url }}">Review more plugin patterns</a></h3>
    <p>Use the examples page once your first plugin works and you want to compare structures or extend the pattern safely.</p>
  </article>
</div>
