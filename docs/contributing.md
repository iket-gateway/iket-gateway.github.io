---
layout: docs_page
title: Contributing to Iket
permalink: /docs/contributing/
section: Start Here
section_order: 1
weight: 3
summary: Learn where maintainer and coding-agent guidance lives, how to validate changes, and which workflows matter most when contributing to Iket.
audience: [maintainer, developer]
topics: [contributing, workflow, architecture, governance]
---

# Contributing to Iket

Use this guide when you are changing the Iket codebase itself rather than only operating a deployed gateway.

<div class="doc-note">
  <p><strong>Best contributor path:</strong> start with the repository guidance files, run focused package tests while iterating, and keep operator docs aligned whenever defaults or CLI behavior change.</p>
</div>

## Where Guidance Lives

There are three different layers of guidance in the Iket project:

- **Website docs**: operator-facing installation, CLI, deployment, and runtime docs
- **`AGENT.md`**: repository architecture assumptions, operational gotchas, and validation expectations for maintainers and coding agents
- **`SKILLS.md`**: lightweight repo-local workflow routing for repeated areas like server lifecycle, routing, TLS, Docker UX, config storage, and CLI/admin work

Use them this way:

- read the website docs when a change affects end users or operators
- read `AGENT.md` when changing behavior, defaults, runtime assumptions, or recovery flows
- read `SKILLS.md` when your change falls into one of the repeated workflow buckets

## Maintainer Jump Links

<div class="docs-grid">
  <article class="doc-card">
    <h3><a href="{{ site.repository_url }}/blob/main/AGENT.md">Open AGENT.md</a></h3>
    <p>Use the repo-level architecture, gotcha, and validation guide when you need the current maintainer contract for Iket.</p>
  </article>
  <article class="doc-card">
    <h3><a href="{{ site.repository_url }}/blob/main/SKILLS.md">Open SKILLS.md</a></h3>
    <p>Use the repo-level workflow catalog when your change falls into a repeated area like server lifecycle, TLS, routing, Docker UX, or CLI/admin work.</p>
  </article>
</div>

## Current Contributor Defaults

These are the current product assumptions contributors should preserve unless a change is intentional:

- `iket server run` is file-backed by default
- PostgreSQL is explicit opt-in through `--database`
- local server scaffolds:
  - `config/config.yaml`
  - `config/service.yaml`
  - `docker-compose.yaml`
- daemon artifacts default to scaffold-local `./logs`
- scaffold backup and restore are supported local workflows, not just internal helpers

<div class="doc-tip">
  <p><strong>Last-updated defaults snapshot:</strong> local startup is file-backed, `--database` is explicit opt-in, daemon artifacts default to scaffold-local `./logs`, and scaffold recovery supports `backups`, per-file restore, full-set restore, preview, confirmation, and `--force` bypass.</p>
</div>

## Focused Validation Workflow

When iterating on a change, prefer focused packages first:

```bash
go test ./pkg/config
go test ./pkg/core/gateway
go test ./cmd/iket
go test ./cmd/iket-cli
```

Then use broader coverage when the change crosses multiple layers:

```bash
go test ./...
```

## High-Value Contribution Areas

### Server lifecycle and local UX

If you touch:
- `cmd/iket/main.go`
- `cmd/iket-cli/run.go`
- scaffold generation
- backup or restore flows
- daemon artifact behavior

Then also check:
- reset-defaults behavior
- backup/restore preview and confirmation
- log and pid defaults
- install and CLI docs

### TLS and remote admin flows

If you touch:
- bootstrap TLS
- cert generation/import
- `setup docker`
- enrollment

Then also check:
- remote SAN coverage
- distinction between server `./certs` and client `~/.iket/certs`
- operator error clarity
- install docs

### Routing and rewrite behavior

If you touch:
- route matching
- `url_pattern`
- `stripPath`
- `base_path`
- `simulate`

Then also check:
- wildcard path cases
- upstream path joining
- CLI simulation
- self-test or management parity

### Config storage behavior

If you touch:
- provider defaults
- early config parsing
- database/file switching
- bootstrap loading

Then also check:
- docs and scaffold templates
- environment expansion
- CLI assumptions about push/pull and local startup

## Common Contributor Pitfalls

- Do not treat server-side `./certs` and client-side `~/.iket/certs` as the same storage location.
- Do not change CLI defaults without updating the website docs.
- Do not add shorthand flags that collide with the global `-f` / `--force`.
- Do not introduce local lifecycle behavior without considering `server doctor`, backups, restore preview, and operator recovery.
- Do not assume the live config inside a container matches the host file without checking it directly.

## Useful Commands

```bash
go test ./pkg/config ./pkg/core/gateway ./cmd/iket ./cmd/iket-cli

iket server run --reset-defaults --init-only
iket server run -d
iket server logs --tail 100
iket server restore --all --latest --preview

docker exec iket sh -lc 'sed -n "1,80p" /app/config/config.yaml'
docker exec iket sh -lc 'sed -n "1,80p" /app/config/service.yaml'
docker exec iket sh -lc 'ls -la /app/certs /app/logs'
```

## Related Guides

- [Installation Guide]({{ '/docs/install/' | relative_url }})
- [CLI Command Reference]({{ '/docs/cli-commands/' | relative_url }})
- [Production Deployment]({{ '/docs/production-deployment/' | relative_url }})
- [Core Docs]({{ '/docs/core-docs/' | relative_url }})
