---
layout: docs_page
title: Config Storage
permalink: /docs/config-storage/
section: Core Docs
section_order: 2
weight: 3
summary: Understand how Iket stores configuration and state, including the default file-backed startup flow and the optional PostgreSQL-backed path.
audience: [operator, developer]
topics: [configuration, storage, operations]
---

# Config Storage

Iket now defaults to file-backed configuration for first local startup. When you run `iket server run` without database flags, the server uses:

- `config/config.yaml`
- `config/service.yaml`

as the active configuration source and generates those files automatically when they are missing.

If you want PostgreSQL as the primary store, opt in explicitly with `iket server run --database ...`.

## Future-Proof Storage Model

The codebase now includes a `MirroringProvider` in [pkg/config/mirroring_provider.go](https://github.com/bhangun/iket/blob/main/pkg/config/mirroring_provider.go).

This is the intended direction for storage strategies:

1. Choose one **primary provider** as the source of truth.
2. Optionally write the same config to one or more **mirror providers**.
3. Keep reads and watches bound to the primary provider.

Examples:

- Primary `FileProvider`, mirror local cache
- Primary SQLite, mirror YAML files for operator visibility
- Primary Postgres, mirror YAML files or local snapshots

## Recommendation

Current local-first default:

- server primary store: file-backed `config.yaml` and `service.yaml`
- user-facing format: the same files you edit and commit

Optional database-backed mode:

- server primary store: PostgreSQL
- user-facing mirror/export: `config.yaml` and optional `service.yaml`

That means admin changes made through `iket` can either stay file-primary for simple local deployments, or be persisted to PostgreSQL first when you explicitly choose database mode.

Practical startup choices:

- `iket server run` (default): file-backed startup
- `iket server run --database ...`: PostgreSQL-backed startup

## Schema Versioning

The database-backed providers maintain internal schema migration tables:

- `iket_schema_migrations`

Current schema version:

- PostgreSQL: `1`
- SQLite: `1`

Recommended direction:

- local development or single-host bring-up: file-backed startup
- managed or shared environments that need a database source of truth: PostgreSQL-backed startup
- operator UX: keep file import/export and optional mirroring

## Why Not Switch Immediately?

A database is not automatically faster for the current workload. Iket config is relatively small, so the real benefits of SQLite/Postgres are:

- transactional updates
- safer concurrent admin operations
- richer audit/history possibilities
- better multi-process or multi-node coordination

The current file-based mode is the default local experience. Database-backed modes preserve file import/export so operators can still work with declarative config files when they need a stronger shared source of truth.
