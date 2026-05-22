---
layout: docs_page
title: Iket Command Reference
permalink: /docs/cli-commands/
section: Start Here
section_order: 1
weight: 2
summary: Learn the main `iket` CLI workflows for setup, remote administration, config changes, and operational checks.
audience: [operator, developer]
topics: [cli, administration, operations]
---

# 🛠️ Iket Command Reference

The `iket` command is your remote control for the Iket API Gateway. It allows you to manage multiple environments, sync configurations, and modify services interactively. The gateway/server binary is `iket-server`.

<div class="doc-note">
  <p><strong>Best reading strategy:</strong> start with setup and contexts, then move into config and service workflows, and only after that dive into the deeper revision, canary, and alerting commands.</p>
</div>

---

## 🌍 Setup & Contexts
Manage different environment profiles (Local, Docker, Staging, Production).

<div class="doc-note">
  <p><strong>Practical workflow:</strong> most operators start with `iket setup`, verify access with `iket context test`, and only then move into config diffs, proposals, and apply flows.</p>
</div>

| Command | Description |
|---------|-------------|
| `iket setup` | Guided wizard to connect to a new gateway. |
| `iket setup docker` | On a trusted server host, import an existing client bundle or mint a local admin client certificate from the server CA and create a ready-to-use mTLS context. |
| `iket server init --mode docker` | Generate a prebuilt Docker deployment scaffold with `config.yaml`, `service.yaml`, compose, and optional `.env`/systemd files. |
| `iket server init --mode host` | Generate a host-native scaffold with `config.yaml`, `service.yaml`, cert/log directories, SQLite path, and optional `.env`/systemd files. |
| `iket server doctor --mode docker` | Check scaffold files, Docker state, `certs/` and `logs/` ownership against `IKET_UID` / `IKET_GID`, container health, local ports, TLS handshakes, generated certs, server-cert SAN coverage versus configured `serverNames`/`serverIPs`, and optionally verify the cert against a real target URL via `--url` or a CLI context via `--context`. |
| `iket server doctor --mode host` | Check host scaffold files, local ports, TLS handshakes, Iket binary availability, and an optional CLI context. |
| `iket enroll create-token` | Create a short-lived enrollment token from the current admin context. |
| `iket enroll list-tokens` | List current and historical enrollment tokens. |
| `iket enroll revoke-token <id>` | Revoke an enrollment token before it is used. |
| `iket enroll use` | Redeem an enrollment token and create a local CLI context without copying the full cert bundle. |
| `iket context list` | List all saved environments. |
| `iket context use <name>` | Switch the active environment. |
| `iket context test [name]` | Verify that a context's cert files exist and that the gateway is reachable. |
| `iket context add <name>` | Manually add a context with `--url`, `--ca`, `--cert`, or `--cert-dir`. |
| `iket context delete <name>` | Remove a context profile. |
| `iket simulate <url-or-path>` | Simulate route matching and upstream path rewriting without calling the upstream service, using either the active context or `--config` / `--services` local files. |
| `iket test <url-or-path>` | Alias for `iket simulate`. |

---

## 🔄 Configuration Sync (GitOps)
Push local files to remote or pull remote state to local files.

<div class="doc-warning">
  <p><strong>Safer path for live environments:</strong> use `diff` and `propose` before `apply` whenever you are working against shared or production gateways. That keeps review, canary, and rollback options open.</p>
</div>

| Command | Description |
|---------|-------------|
| `iket config apply <file> --replace` | Replace the full remote gateway config from a file. |
| `iket config apply <file> --merge` | Merge a gateway config file into the current remote config. |
| `iket config propose <file> --replace` | Create a pending proposal for full remote gateway config replacement. Supports `--proposer`, `--env`, and `--not-before`. |
| `iket config propose <file> --merge` | Create a pending proposal for merging a gateway config file into the current remote config. Supports `--proposer`, `--env`, and `--not-before`. |
| `iket config diff <file> --replace` | Preview how a full gateway config replacement would change the remote state. |
| `iket config diff <file> --merge` | Preview how a gateway config merge would change the remote state. |
| `iket service apply <file> --replace` | Replace the full remote services state from a file. |
| `iket service apply <file> --merge` | Merge a service definition file into the current remote services state. |
| `iket service propose <file> --replace` | Create a pending proposal for full remote services replacement. Supports `--proposer`, `--env`, `--not-before`, `--canary-service`, `--canary-route`, and `--canary-header`. |
| `iket service propose <file> --merge` | Create a pending proposal for merging a service definition file into the current remote services state. Supports `--proposer`, `--env`, `--not-before`, `--canary-service`, `--canary-route`, and `--canary-header`. |
| `iket service diff <file> --replace` | Preview how a full service replacement would change the remote state. |
| `iket service diff <file> --merge` | Preview how a service merge would change the remote state. |
| `iket push config <file>` | Export local gateway settings to remote. |
| `iket push services <file>` | Export local service/route definitions to remote. |
| `iket pull config [file]` | Fetch remote settings and save locally (YAML/JSON). |
| `iket pull services [file]` | Fetch remote services and save locally. |

**Strategies for Push:**
- `--strategy merge` (Default): Update existing items and add new ones.
- `--strategy replace`: Overwrite the remote state entirely with your local file.

---

## 📦 Service Management
Manage backends and groups of routes.

<div class="doc-tip">
  <p><strong>Common operator flow:</strong> define or update services in files, preview with `service diff`, then promote the change through proposal-based workflows instead of editing live state blindly.</p>
</div>

| Command | Description |
|---------|-------------|
| `iket service list` | List all services and their routes. |
| `iket service create -i` | **Interactive Wizard** to build a service step-by-step. |
| `iket service create <file>` | Create service from a YAML/JSON file. |
| `iket service apply <file> --replace` | Replace the entire remote services set from a service definition file. |
| `iket service apply <file> --merge` | Merge a service definition file into the current remote services set. |
| `iket service propose <file> --replace` | Create a pending proposal for replacing the entire remote services set. Supports `--proposer`, `--env`, `--not-before`, `--canary-service`, `--canary-route`, and `--canary-header`. |
| `iket service propose <file> --merge` | Create a pending proposal for merging a service definition file into the current remote services set. Supports `--proposer`, `--env`, `--not-before`, `--canary-service`, `--canary-route`, and `--canary-header`. |
| `iket service diff <file> --replace` | Preview replacing the entire remote services set from a service definition file. |
| `iket service diff <file> --merge` | Preview merging a service definition file into the current remote services set. |
| `iket service set <name>` | Update attributes like `--host` or `--desc` for a specific service. |
| `iket service delete <name>` | Remove a service and all its routes. |

---

## 🛣️ Route Management
Fine-grained control over individual API endpoints.

| Command | Description |
|---------|-------------|
| `iket route list` | List all routes across all services. |
| `iket route set <svc> <path> <method>` | Update a specific route (e.g., `--auth true`, `--enabled false`). |
| `iket route diff-create <file>` | Preview creating a route from a YAML/JSON file. |
| `iket route diff-update <id> <file>` | Preview updating a route from a YAML/JSON file. |
| `iket route enable <id>` | Enable a route by ID. |
| `iket route disable <id>` | Disable a route by ID. |

---

## ⚡ Gateway & Plugins
Monitor and control the gateway core.

| Command | Description |
|---------|-------------|
| `iket gateway status` | Check health, uptime, and request metrics. |
| `iket gateway edition` | Show the active Iket edition and advertised capabilities. |
| `iket gateway capabilities [--category gateway]` | List active edition capabilities, optionally filtered by category. |
| `iket gateway capability <key>` | Check whether the active edition supports one capability key. |
| `iket gateway extension <name>` | Show one registered management route extension and its edition/capability support status. |
| `iket gateway extensions [--search billing --stage preview --compatibility incompatible --provider-kind enterprise --provider "Iket Enterprise" --permission billing.read --route-prefix /enterprise --link-rel docs --supported or --unsupported --status capability_unavailable --capability commercial.billing --unsupported-capability commercial.billing --category commercial --tag billing]` | List registered management route extensions and filter by edition/capability support status plus permission/compatibility/provider/route/link/release discovery metadata. |
| `iket gateway config` | View the live gateway configuration (secrets redacted). |
| `iket gateway reload` | Force a hot-reload of all configurations. |
| `iket gateway route-policy --path /foo --method GET` | Inspect the matched route's resolved AI guardrails, applied preset stack, effective policy fields, and per-field source attribution. Supports repeatable `--header Name:Value` plus optional `--bucket-key` for percent-gated routes. |
| `iket gateway route-policy-diff --from-path /a --to-path /b` | Compare two matched route policies and list only the effective guardrail fields that differ, including each side's value and source layer. Supports per-side `--from-method` / `--to-method`, repeatable `--from-header` / `--to-header`, and `--from-bucket-key` / `--to-bucket-key`. |
| `iket gateway limit-class-digest-profile --webhook pager` | Inspect one notification webhook's resolved limiter digest receiver view, including the applied digest profile chain, effective receiver fields, and per-field source attribution. |
| `iket gateway limit-class-digest-profile-diff --from-webhook pager --to-profile base` | Compare two resolved limiter digest receiver views, including webhook-versus-webhook or webhook-versus-profile comparisons with each side's values and source attribution. |
| `iket gateway limit-class-digest-profile-explain --webhook pager --field limitClassDigestMinSeverity` | Explain how one resolved limiter digest receiver field got its final value, including the step-by-step chain/profile/webhook precedence trace. |
| `iket gateway limit-class-digest-profile-explain-diff --from-webhook pager --to-profile base --field limitClassDigestMinSeverity` | Compare the precedence trace for one resolved limiter digest receiver field, including stage-by-stage differences across both sides. |
| `iket gateway limit-class-digest-profile-explain-bundle --webhook pager --bundle ops-core --field limitClassDigestTypes` | Inspect one resolved limiter digest receiver view and return explain traces for multiple selected fields or named explain bundles in a single call. |
| `iket gateway limit-class-digest-profile-explain-bundle-diff --from-webhook pager --to-profile base --diff-profile pager-audit --from-role baseline --to-role pager` | Compare a multi-field explain bundle across two targets and return per-field precedence diffs in one response, optionally using a named diff profile with expected drift rules and target roles. |
| `iket gateway limit-class-digest-assertion-preset --preset prod-strict` | Inspect one resolved limiter digest assertion preset chain, including inherited preset stages, effective rules/groups, and source attribution for inherited rules and groups. |
| `iket gateway limit-class-digest-assertion-preset-diff --from-preset prod-strict --to-preset base-safety` | Compare two resolved limiter digest assertion preset chains and show which effective preset sections differ between them. |
| `iket gateway limit-class-digest-assertion-preset-explain --preset prod-strict --kind rules` | Explain the resolved `rules` or `groups` section for one limiter digest assertion preset, including the step-by-step preset chain that produced the final result. |
| `iket gateway limit-class-digest-assertion-preset-explain-bundle --preset prod-strict --kind rules --kind groups` | Inspect one resolved limiter digest assertion preset chain and return explain traces for multiple preset sections in a single call. |
| `iket gateway limit-class-digest-assertion-preset-explain-diff --from-preset prod-strict --to-preset base-safety --kind rules` | Compare the stage-by-stage resolution of one preset section across two limiter digest assertion presets, including where inherited rules or groups diverge. |
| `iket gateway limit-hits [--window 5m]` | Show aggregated route limiter saturation counters for native `rateLimitPolicy` and `concurrencyLimitPolicy`, grouped by limiter type and route with a recent-window summary. Also includes current in-flight and queued depth per route, queued-admission visibility, queue-full rejections, and queue wait timing. |
| `iket gateway limit-buckets [--window 5m --min-count 1]` | Show recent limiter pressure grouped by anonymized bucket ID, with route, limiter type, key type, hit counts, and queue-pressure stats so hot tenants stand out directly. |
| `iket gateway limit-classes [--window 5m --min-count 1]` | Show recent limiter pressure grouped by named bucket class, with route, limiter type, key type, hit counts, and queue-pressure stats so class-level overload trends stand out directly. |
| `iket gateway limit-class-alerts [--window 5m --min-count 3]` | Detect recent limiter saturation spikes grouped by named bucket class, with severity plus queue-full and queued-admission context so class-level incidents stand out directly. |
| `iket gateway limit-class-snoozes [--bucket-class vip-jwt --service agent --route /ai/chat --limit-type concurrency_queue_full --key-type jwt_sub --expiring-within 15m]` | Show currently snoozed limiter class incidents, including remaining snooze time and an expiring-soon view for incidents that will resume alerting shortly. |
| `iket gateway notify-limit-class-snoozes [--bucket-class vip-jwt --service agent --route /ai/chat --limit-type concurrency_queue_full --key-type jwt_sub --expiring-within 15m]` | Emit digest and per-incident notifications for snoozed limiter class incidents that are nearing the end of their snooze window. |
| `iket gateway limit-class-incidents [--severity critical --bucket-class vip-jwt --service agent --route /ai/chat --limit-type concurrency_queue_full --key-type jwt_sub]` | Show currently open limiter incidents grouped by named bucket class, with optional exact-match filters for severity, class, service, route, limiter type, and key type. |
| `iket gateway acknowledge-limit-class-incident --incident-id lcal-... --reviewer ops-lead [--note "tracking under incident board"]` | Mark an open limiter class incident as acknowledged so the live incident view and webhook consumers can distinguish known overload from unattended overload. |
| `iket gateway snooze-limit-class-incident --incident-id lcal-... --reviewer ops-lead --duration 15m [--note "muting repeated alerts during active mitigation"]` | Snooze an open limiter class incident for a bounded duration so repeated class-alert notifications are muted while mitigation is already in progress. |
| `iket gateway notify-limit-class-alerts [--window 5m --min-count 3]` | Emit recent gateway limiter class alert digests and per-alert webhook events through configured notification targets. |

Class alerting can also run automatically through `security.mutationPolicy.limitClassAlertNotifications`, which emits `gateway.limit_class_alert_digest`, `gateway.limit_class_alert`, `gateway.limit_class_alert_opened`, `gateway.limit_class_alert_stage_changed`, and `gateway.limit_class_alert_resolved` as named bucket classes move through overload incidents. Snooze-expiry warnings can now run automatically through `security.mutationPolicy.limitClassSnoozeNotifications`, which uses the same `window` duration as the expiring-soon threshold and emits `gateway.limit_class_snooze_expiring_digest`, `gateway.limit_class_snooze_expiring`, `gateway.limit_class_snooze_stage_changed`, and `gateway.limit_class_snooze_resumed`. Those wake-up events now carry staged severity based on remaining snooze time, and the stage cutoffs are configurable with `snoozeElevatedWithin` and `snoozeCriticalWithin` on `security.mutationPolicy.limitClassSnoozeNotifications` instead of being fixed. Both class-alert and class-snooze notifiers now also support `detailedMinBucketClassPriority`, which keeps only higher-priority named classes in the shared digest detail list while lower-priority classes roll into grouped summary counts, and `detailedMaxBucketClasses`, which caps how many qualifying classes can appear in digest detail even within the same priority tier. Named bucket classes under `security.limitAlertBucketClasses` can now also override those same thresholds with their own `snoozeElevatedWithin` and `snoozeCriticalWithin`, restrict which snooze lifecycle signals are emitted at all with `snoozeEventTypes` such as `["resumed"]` or `["expiring","stage_changed"]`, and opt out of the shared expiring digest with `snoozeExcludeFromDigest` while still keeping per-incident lifecycle events. Digest-excluded, lower-priority, or over-quota classes no longer appear as detailed snooze rows in `gateway.limit_class_snooze_expiring_digest`, but they still contribute grouped summary counts ranked first by class `priority` and then by count so operators retain aggregate awareness without reopening all the noise. That lets a class like `vip-jwt` wake up or page faster than the global default, lets `minLimitAlertSeverity` treat broader heads-up warnings as `warning` or `elevated` while reserving `critical` for the most urgent expiries, lets downstream systems react specifically when a muted incident crosses from one wake-up stage into another, and provides an explicit resume event when the snooze window fully ends and normal paging is live again. Webhooks and limiter recipient profiles can now also use `limitClassSnoozeEventTypes` with values like `["expiring"]`, `["stage_changed"]`, or `["resumed"]` to route different snooze lifecycle signals to different destinations without relying only on broad event lists. Webhooks and limiter recipient profiles can also override digest visibility per receiver with `limitClassDigestTypes`, `limitClassDigestSummaryOnlyTypes`, `limitClassDigestMinSeverity`, `limitClassDigestSeverities`, `limitClassDigestMinBucketClassPriority`, `limitClassDigestMaxBucketClasses`, `limitClassDigestMinSummarySeverity`, `limitClassDigestMinSummaryBucketClassPriority`, `limitClassDigestSummarySortMode`, `limitClassDigestMinSummaryCount`, `limitClassDigestOtherBucketLabel`, `limitClassDigestOverflowReasons`, `limitClassDigestOverflowReasonLabels`, `limitClassDigestOverflowReasonGroups`, `limitClassDigestOverflowReasonOrder`, `limitClassDigestMaxOverflowReasons`, `limitClassDigestTruncatedReasonBucketLabel`, `limitClassDigestTruncatedReasonBucketMode`, `limitClassDigestTruncatedReasonBucketMinSeverity`, `limitClassDigestTruncatedReasonBucketSeverities`, `limitClassDigestTruncatedReasonBucketSortMode`, `limitClassDigestTruncatedReasonBucketDominantReasonStrategy`, `limitClassDigestTruncatedReasonBucketMaxReasons`, `limitClassDigestTruncatedReasonBucketReasonOrder`, `limitClassDigestTruncatedReasonBucketHiddenStrategyOrder`, `limitClassDigestTruncatedReasonBucketHiddenStrategyDominantMode`, `limitClassDigestTruncatedReasonBucketExactSeverityPolicy`, `limitClassDigestTruncatedReasonBucketExactSeverityDominantMode`, `limitClassDigestTruncatedReasonBucketMinSeverityPolicy`, `limitClassDigestTruncatedReasonBucketMinSeverityDominantMode`, `limitClassDigestTruncatedReasonBucketMaxReasonsPolicy`, `limitClassDigestTruncatedReasonBucketMaxReasonsDominantMode`, `limitClassDigestTruncatedReasonBucketHiddenStrategyPriorityWeight`, `limitClassDigestTruncatedReasonBucketHiddenStrategyReasonWeight`, `limitClassDigestTruncatedReasonBucketHiddenStrategyItemWeight`, `limitClassDigestTruncatedReasonBucketHiddenStrategyPriorityCap`, `limitClassDigestTruncatedReasonBucketHiddenStrategyReasonCap`, `limitClassDigestTruncatedReasonBucketHiddenStrategyItemCap`, `limitClassDigestTruncatedReasonBucketHiddenStrategyMinReasons`, `limitClassDigestTruncatedReasonBucketHiddenStrategyMinItems`, `limitClassDigestTruncatedReasonBucketExactSeverityPriorityCap`, `limitClassDigestTruncatedReasonBucketExactSeverityReasonCap`, `limitClassDigestTruncatedReasonBucketExactSeverityItemCap`, `limitClassDigestTruncatedReasonBucketExactSeverityPriorityWeight`, `limitClassDigestTruncatedReasonBucketExactSeverityReasonWeight`, `limitClassDigestTruncatedReasonBucketExactSeverityItemWeight`, `limitClassDigestTruncatedReasonBucketExactSeverityMinReasons`, `limitClassDigestTruncatedReasonBucketExactSeverityMinItems`, `limitClassDigestTruncatedReasonBucketMinSeverityPriorityCap`, `limitClassDigestTruncatedReasonBucketMinSeverityReasonCap`, `limitClassDigestTruncatedReasonBucketMinSeverityItemCap`, `limitClassDigestTruncatedReasonBucketMinSeverityPriorityWeight`, `limitClassDigestTruncatedReasonBucketMinSeverityReasonWeight`, `limitClassDigestTruncatedReasonBucketMinSeverityItemWeight`, `limitClassDigestTruncatedReasonBucketMinSeverityMinReasons`, `limitClassDigestTruncatedReasonBucketMinSeverityMinItems`, `limitClassDigestTruncatedReasonBucketMaxReasonsPriorityCap`, `limitClassDigestTruncatedReasonBucketMaxReasonsReasonCap`, `limitClassDigestTruncatedReasonBucketMaxReasonsItemCap`, `limitClassDigestTruncatedReasonBucketMaxReasonsPriorityWeight`, `limitClassDigestTruncatedReasonBucketMaxReasonsReasonWeight`, `limitClassDigestTruncatedReasonBucketMaxReasonsItemWeight`, `limitClassDigestTruncatedReasonBucketMaxReasonsMinReasons`, `limitClassDigestTruncatedReasonBucketMaxReasonsMinItems`, and `limitClassDigestMaxSummaryBucketClasses`, so one destination can shape only `alert` digests in detail while leaving `snooze` digests untouched, another can force `snooze` digests into grouped summary with no detailed rows at all, another can cap that grouped summary to only the top `N` classes while still receiving overflow counts for the hidden classes, another can hide low-priority classes from the grouped summary list entirely while still counting them in overflow, another can keep only `critical` or `elevated+` classes visible in the grouped summary while still counting lower-severity classes in overflow, another can reorder the grouped summary by `severity_first`, `count_first`, or the default `priority_first`, another can collapse low-count classes into the overflow bucket automatically before the grouped summary is rendered, another can expose that overflow under a named structured bucket like `other_low_count` instead of only raw counters, another can keep only selected overflow reasons like `low_priority` visible while folding `low_count` or `max_summary_budget` into hidden overflow metadata, another can relabel those visible reasons into friendlier receiver-specific terms like `priority_floor` or `digest_quota` while still preserving the raw reason alongside the display label, another can group multiple raw reasons like `low_priority` and `low_count` under one higher-level display category such as `policy_filtered` while still carrying `raw_reasons` underneath, another can cap the visible grouped overflow categories with `limitClassDigestMaxOverflowReasons` so only the top `N` grouped reason rows stay visible while the remainder folds into a structured `truncated_reason_bucket`, another can rename that truncation bucket itself with `limitClassDigestTruncatedReasonBucketLabel` to something like `suppressed_categories` or `digest_reason_overflow`, another can switch that truncation bucket from count-only to grouped detail with `limitClassDigestTruncatedReasonBucketMode: detailed` so it also emits grouped `reasons`, `dominant_reason`, and `dominant_raw_reason`, another can keep only stronger nested grouped reasons there with `limitClassDigestTruncatedReasonBucketMinSeverity`, another can constrain that nested detailed bucket to an exact severity set like `["critical"]` with `limitClassDigestTruncatedReasonBucketSeverities`, another can sort that nested bucket by `severity_first`, `count_first`, or `custom_order_first` with `limitClassDigestTruncatedReasonBucketSortMode`, another can control how the nested bucket picks `dominant_reason` and `dominant_raw_reason` with `limitClassDigestTruncatedReasonBucketDominantReasonStrategy` using `default`, `severity_first`, or `count_first`, another can cap those nested detailed rows with `limitClassDigestTruncatedReasonBucketMaxReasons` so the truncated bucket itself can still collapse excess nested reasons into `hidden_reason_count` metadata, another can see that nested hidden overflow broken down by cause through fields like `hidden_by_exact_severity_reason_count`, `hidden_by_min_severity_reason_count`, and `hidden_by_max_reasons_reason_count`, another can inspect which hidden-reason strategies were actually active through the structured `active_hidden_reason_strategies` block, including the configured severity and max-row values those strategies used, another can reorder those nested strategy explanations with `limitClassDigestTruncatedReasonBucketHiddenStrategyOrder`, and another can choose whether `dominant_hidden_reason_strategy` follows explicit order, highest hidden reason count, highest hidden item count, or a receiver-weighted blend with `limitClassDigestTruncatedReasonBucketHiddenStrategyDominantMode`. Receivers can now also override that dominant-mode per hidden strategy with `limitClassDigestTruncatedReasonBucketExactSeverityDominantMode`, `limitClassDigestTruncatedReasonBucketMinSeverityDominantMode`, and `limitClassDigestTruncatedReasonBucketMaxReasonsDominantMode`, and they can now override the weighted-score inputs per hidden strategy with `limitClassDigestTruncatedReasonBucketExactSeverityPriorityWeight`, `limitClassDigestTruncatedReasonBucketExactSeverityReasonWeight`, `limitClassDigestTruncatedReasonBucketExactSeverityItemWeight`, `limitClassDigestTruncatedReasonBucketMinSeverityPriorityWeight`, `limitClassDigestTruncatedReasonBucketMinSeverityReasonWeight`, and `limitClassDigestTruncatedReasonBucketMinSeverityItemWeight`, `limitClassDigestTruncatedReasonBucketMaxReasonsPriorityWeight`, `limitClassDigestTruncatedReasonBucketMaxReasonsReasonWeight`, and `limitClassDigestTruncatedReasonBucketMaxReasonsItemWeight`, and they can now bundle those per-strategy dominance, threshold, weight, and cap settings into `limitClassDigestTruncatedReasonBucketExactSeverityPolicy`, `limitClassDigestTruncatedReasonBucketMinSeverityPolicy`, and `limitClassDigestTruncatedReasonBucketMaxReasonsPolicy` instead of relying only on scattered flat fields. In weighted mode, those per-strategy values override the shared `limitClassDigestTruncatedReasonBucketHiddenStrategyPriorityWeight`, `limitClassDigestTruncatedReasonBucketHiddenStrategyReasonWeight`, and `limitClassDigestTruncatedReasonBucketHiddenStrategyItemWeight` for that hidden strategy only, while the strategy-specific caps or the shared `limitClassDigestTruncatedReasonBucketHiddenStrategyPriorityCap`, `limitClassDigestTruncatedReasonBucketHiddenStrategyReasonCap`, and `limitClassDigestTruncatedReasonBucketHiddenStrategyItemCap` still cap the raw contribution values before weighting so hidden item volume can’t dominate the explanation path more than the receiver allows. `LimitClassDigestTruncatedReasonBucketHiddenStrategyMinReasons` and `limitClassDigestTruncatedReasonBucketHiddenStrategyMinItems` still provide shared eligibility gates, while `limitClassDigestTruncatedReasonBucketExactSeverityMinReasons`, `limitClassDigestTruncatedReasonBucketExactSeverityMinItems`, `limitClassDigestTruncatedReasonBucketMinSeverityMinReasons`, `limitClassDigestTruncatedReasonBucketMinSeverityMinItems`, `limitClassDigestTruncatedReasonBucketMaxReasonsMinReasons`, and `limitClassDigestTruncatedReasonBucketMaxReasonsMinItems` let each hidden strategy override those minima independently before it can compete to become `dominant_hidden_reason_strategy`. That structured overflow bucket now also includes per-reason grouped metadata like `low_count`, `low_priority`, `low_severity`, or `max_summary_budget`, plus a dominant reason, so receivers can tell why classes were collapsed instead of only seeing aggregate overflow totals. Webhooks can independently debounce those wake-up warnings with `limitClassSnoozeExpiryCooldown`, subscribe only to the most urgent wake-ups with `limitClassSnoozeExpiryWithin`, explicitly route only selected wake-up stages with `limitClassSnoozeExpiryStages` like `["warning","elevated"]` for chatops and `["critical"]` for pager targets, or require a minimum bucket-class priority with `minLimitAlertBucketClassPriority` so only higher-priority named classes generate limiter deliveries for that receiver.
Hidden-strategy policy blocks can now also be reused through `security.limitClassDigestHiddenStrategyPolicyPresets`. Receivers and `limitAlertProfiles` can reference those catalog entries with `limitClassDigestTruncatedReasonBucketExactSeverityPolicyPreset`, `limitClassDigestTruncatedReasonBucketMinSeverityPolicyPreset`, and `limitClassDigestTruncatedReasonBucketMaxReasonsPolicyPreset`, while still keeping the inline `...Policy` blocks available for one-off overrides. If both are present, the inline per-strategy policy block wins.

Those preset references can now be composed too with `limitClassDigestTruncatedReasonBucketExactSeverityPolicyPresetChain`, `limitClassDigestTruncatedReasonBucketMinSeverityPolicyPresetChain`, and `limitClassDigestTruncatedReasonBucketMaxReasonsPolicyPresetChain`. Chains resolve left-to-right, then the single `...PolicyPreset` applies, and finally the inline `...Policy` block remains the narrowest override.

Teams can now also reuse whole limiter digest receiver views through `security.limitClassDigestProfiles`. A webhook can reference one with `limitClassDigestProfile`, which applies the shared digest-shaping defaults before any explicit webhook fields, so repeated digest layouts no longer need to be copied receiver by receiver.

Those digest receiver views can now be layered too with `limitClassDigestProfileChain`. Chain entries apply left-to-right, then the single `limitClassDigestProfile` applies, and finally explicit webhook fields remain the narrowest override for that receiver.

Operators can inspect that fully resolved receiver view directly with `iket gateway limit-class-digest-profile --webhook <name>` or `iket gateway limit-class-digest-profile --profile <name>`, which returns the applied digest profile chain, effective receiver fields, and per-field source attribution.

When teams need to compare a live webhook against another webhook or a reusable named digest profile, `iket gateway limit-class-digest-profile-diff --from-webhook <name> --to-profile <name>` highlights only the differing resolved receiver fields and preserves each side's source attribution.

When someone needs to answer "why did this field end up here?" instead of only "what differs?", `iket gateway limit-class-digest-profile-explain --webhook <name> --field <field>` returns the step-by-step precedence trace across profile-chain entries, the single digest profile, and explicit webhook overrides.

When teams need that same reasoning side by side, `iket gateway limit-class-digest-profile-explain-diff --from-webhook <name> --to-profile <name> --field <field>` compares the stage-by-stage precedence trace for one field and highlights which chain/profile/webhook steps only exist on one side or resolve differently.

When operators need several high-signal "why" traces at once, `iket gateway limit-class-digest-profile-explain-bundle --webhook <name> --field <field> --field <field>` returns the shared resolved inspection plus a keyed explanation block for each requested field instead of making one explain call per field.

Teams can now define reusable field sets through `security.limitClassDigestExplainBundles`, then call `iket gateway limit-class-digest-profile-explain-bundle --webhook <name> --bundle ops-core` to expand that named set automatically. Explicit `--field` flags still layer on top of the bundle and are deduplicated in the final explain set.

Those same named bundles can now be compared side by side with `iket gateway limit-class-digest-profile-explain-bundle-diff --from-webhook <name> --to-profile <name> --bundle ops-core`, which expands the shared field set once and returns a per-field explain diff plus a concise changed-field summary.

Teams can now also define reusable comparison presets through `security.limitClassDigestExplainDiffProfiles`. A diff profile can contribute `bundles`, extra `fields`, and `allowedChangedFields`, then be applied with `--diff-profile pager-audit` so the response can distinguish expected drift from `unexpected_changed_fields`.

Those diff profiles can now also declare `expectedFromValues` and `expectedToValues` for exact-value compliance checks. Expected values are compared against the resolved final field value on each side, and any mismatch is returned in `assertion_failures`, which makes the same diff response useful for both drift review and policy conformance checks.

They can also declare `assertFromRules` and `assertToRules` for rule-based checks using `exists`, `contains:<value>`, `not_contains:<value>`, `regex:<pattern>`, `lte:<number>`, or `gte:<number>`. Rule failures also land in `assertion_failures`, so one diff profile can mix exact-value assertions with presence, membership, pattern, and numeric threshold checks.

For more realistic field policies, diff profiles can also use structured `assertFromGroups` and `assertToGroups` entries with `operator: allOf`, `operator: anyOf`, or `operator: noneOf` plus a `rules` list. Those groups can now also nest through `groups: [...]`, which means one field can express boolean trees like “must exist, and one of these alternates must match” or “none of these risky subconditions may pass” without flattening everything into unrelated single rules.

Teams can now also define reusable `security.limitClassDigestAssertionPresets`, then attach them per field through `assertFromPresets` and `assertToPresets`. Preset rules and groups are merged before inline field assertions, so shared boolean trees can be reused across multiple diff profiles without giving up local overrides.

Those presets can now also compose through `presetChain`, so a catalog entry like `prod-strict` can inherit from `base-safety` before adding its own rules and groups. When a field references such a preset, Iket resolves the preset chain first and then applies the field’s inline assertions last.

Operators can now also compare two resolved preset chains directly with `limit-class-digest-assertion-preset-diff`, which highlights drift in the effective preset output without requiring a full diff profile around it.

When teams need to understand not just that two assertion presets differ but where their inherited `rules` or `groups` paths diverge, `iket gateway limit-class-digest-assertion-preset-explain-diff --from-preset <name> --to-preset <name> --kind rules` compares the stage-by-stage preset chain side by side.

When operators need the full resolved preset inspection plus multiple section traces at once, `iket gateway limit-class-digest-assertion-preset-explain-bundle --preset <name> --kind rules --kind groups` returns a shared preset inspection with keyed explanations for each requested section.

Those diff profiles can now also declare `fromRole` and `toRole`. When they do, the CLI/API requires matching `--from-role` and `--to-role` values, which prevents a preset like `baseline -> pager` from being used in the wrong direction and making the drift review misleading.

| `iket gateway limit-alerts [--window 5m --min-count 3]` | Detect recent limiter saturation spikes by route and limiter type and classify them by severity, including queue-full and queued-admission context. Routes with `limitAlertPolicy.groupBy: bucket` split alerts per anonymized limiter bucket as well, and `bucketPolicies` can apply stricter thresholds to matching tenants or API-key buckets. |
| `iket gateway notify-limit-alerts [--window 5m --min-count 3]` | Emit recent gateway limiter alert digests and per-alert webhook events through configured notification targets. |
| `iket gateway policy-hits [--window 5m]` | Show aggregated gateway policy-hit counters plus a recent-window summary grouped by reason and route. |
| `iket gateway policy-alerts [--window 5m --min-count 3]` | Detect recent policy-hit spikes by route and reason and classify them by severity. |
| `iket gateway notify-policy-alerts [--window 5m --min-count 3]` | Emit recent gateway policy alert digests and per-alert webhook events through configured notification targets. |
| `iket gateway shadow-report` | Show aggregated live-vs-shadow comparison data grouped by service and route. |
| `iket gateway shadow-evaluate` | Evaluate grouped shadow traffic against route-level shadow thresholds. |
| `iket plugin list` | List all available and active plugins. |
| `iket plugin diff-config <name> <file>` | Preview how a plugin config file would change the remote plugin configuration. |
| `iket plugin enable <name>` | Enable a plugin globally. |
| `iket plugin disable <name>` | Disable a plugin globally. |

---

## 🕘 Revisions
Inspect and restore recorded configuration revisions.

<div class="doc-note">
  <p><strong>Why this matters:</strong> revisions, proposals, and canaries are what make the CLI operationally safe. If you only learn one advanced area after setup, learn this one.</p>
</div>

| Command | Description |
|---------|-------------|
| `iket revision list` | List recorded remote configuration revisions with action metadata and service/route counts. |
| `iket revision show <id>` | Show one recorded revision, including stored config and summary metadata. |
| `iket revision diff <from> <to>` | Compare two revisions, or compare a revision with `current`. |
| `iket revision restore <id>` | Restore a recorded remote configuration revision. |
| `iket proposal list` | List pending and reviewed configuration proposals. |
| `iket proposal queue` | Show proposal queue entries with readiness summaries, blockers, aging/worklist signals, urgency classification, priority score/reason, next action hints, and optional `--env`, `--status`, `--next-action`, `--urgency`, `--ready-only`, or `--blocked-only` filters. |
| `iket proposal digest` | Show a compact rollout digest with queue summary, top ready proposals, top blocked proposals, top blocker reasons, and an `attention_required` section for SLA-breached overdue proposals. Supports the same queue filters, including `--next-action` and `--urgency`. |
| `iket proposal notify-digest` | Emit the current filtered proposal queue digest through configured notification webhooks. This always emits `proposal.digest`, and also emits `proposal.sla_breach` when the filtered queue contains overdue SLA-breached proposals. Supports the same queue filters as `proposal digest`. |
| `iket proposal export-digest <file>` | Export the compact rollout digest to JSON or YAML, with optional `--env`, `--status`, `--next-action`, `--urgency`, `--ready-only`, `--blocked-only`, and `--limit` filters. |
| `iket proposal export-queue <file>` | Export the filtered proposal queue to JSON or YAML, with optional `--env`, `--status`, `--next-action`, `--urgency`, `--ready-only`, `--blocked-only`, and `--limit` filters. |
| `iket proposal blocked-report` | Aggregate blocked proposals by blocker reason, with optional `--env` and `--status` filters. |
| `iket proposal export-blocked <file>` | Export the blocked proposal report to JSON or YAML, with optional `--env` and `--status` filters. |
| `iket proposal approve-ready` | Batch-approve approval-blocked proposals oldest-first, with `--reviewer`, optional `--env`, `--status`, `--next-action needs_approval`, `--urgency`, `--limit`, and global `--dry-run` support. |
| `iket proposal apply-ready` | Batch-apply ready proposals oldest-ready-first, with `--reviewer`, optional `--env`, `--status`, `--next-action apply`, `--urgency`, `--limit`, and global `--dry-run` support. |
| `iket proposal batch preview` | Preview a filtered batch action with `--action approve|apply`, shared `--env`, `--status`, `--next-action`, `--urgency`, `--limit`, and optional reviewer metadata. |
| `iket proposal batch blocked` | Summarize blockers for the current filtered batch slice using the shared batch selector flags, including `--urgency`. |
| `iket proposal batch explain` | Explain the dominant blocker, next action, remediation direction, suggested follow-up command, machine-friendly remediation steps, and a formal `suggested_plan` envelope with current-step metadata and `execution_hints` for the current filtered batch slice. Supports the shared queue selectors, including `--urgency`. |
| `iket proposal batch export <file>` | Export a filtered batch `--view queue|digest|blocked|explain` snapshot to JSON or YAML using the shared batch selector flags, including `--urgency` and the stateful `suggested_plan` envelope plus `execution_hints` for `explain`. |
| `iket proposal batch approve` | Batch-approve a filtered proposal queue slice through the shared batch surface, including `--urgency` filtering. |
| `iket proposal batch apply` | Batch-apply a filtered ready proposal queue slice through the shared batch surface, including `--urgency` filtering. |
| `iket proposal batch act` | Unified batch entrypoint for `--action approve|apply|blocked|explain|export`, with `--dry-run` preview support for approve/apply, `--output` for export, and the same selector model including `--urgency`. |
| `iket proposal explain-blocked <id>` | Show a concise blocked diagnosis, primary blocker, and next step for one proposal. |
| `iket proposal show <id>` | Show one stored proposal, including stored config and summary metadata. |
| `iket proposal verify <id>` | Verify proposal integrity, source lineage, and live drift before apply. |
| `iket proposal readiness <id>` | Show whether a proposal is ready to apply and list the current blockers. |
| `iket proposal approve <id>` | Record an approval for a stored proposal without applying it. Supports `--reviewer` and `--review-note`. |
| `iket proposal apply <id>` | Approve and apply a stored proposal. Supports `--reviewer` and `--review-note`. |
| `iket proposal canary status <id>` | Show the current canary rollout selectors, proposal status, and verification summary. |
| `iket proposal canary evaluate <id>` | Evaluate current canary health from recent request logs against configured canary thresholds. |
| `iket proposal canary advance <id>` | Advance an active percentage canary rollout to its next planned `--canary-step`. Supports `--reviewer` and `--review-note`. |
| `iket proposal canary reconcile <id>` | Evaluate an active canary and automatically advance, complete, or roll it back. Supports `--reviewer` and `--review-note`. |
| `iket proposal canary expand <id>` | Expand a planned or active canary rollout with additional `--canary-service`, `--canary-route`, `--canary-header`, or `--canary-percent` selectors. Active canaries also support `--reviewer`, `--review-note`, and canary guard flags. |
| `iket proposal canary complete <id>` | Finish an active canary rollout and apply the full stored proposal. Supports `--reviewer` and `--review-note`. |
| `iket proposal promote <id>` | Clone a stored proposal into the next environment with fresh approvals. Supports `--proposer`, `--env`, `--not-before`, `--canary-service`, `--canary-route`, `--canary-header`, `--canary-percent`, `--canary-step`, and canary guard flags. |
| `iket proposal reject <id>` | Reject a stored proposal without applying it. Supports `--reviewer` and `--review-note`. |

Revision-aware mutating commands can also attach metadata:
- `--label "tenant-rollout"`
- `--note "Enable new identity routes before chat deploy"`
- `--change-ref "CHG-1427"`

Gateway-side enforcement is also available through `security.mutationPolicy`, so non-CLI callers can be held to the same metadata requirements. It can also be scoped to `config`, `services`, `routes`, `plugins`, `clients`, `revisions`, or `high_impact` actions instead of applying to everything. Proposal creation can record `--proposer`, `--env`, and `--not-before`, while `proposal promote` preserves lineage into the next environment and resets approvals. Proposal apply can optionally require a different reviewer, a minimum number of fresh approvals, a scheduled not-before window, promoted-proposal verification, promoted shadow evaluation, a consecutive healthy shadow-verification streak, proposal expiration, or recurring blackout windows for high-impact changes before apply succeeds. Service proposals can also move through a staged canary lifecycle with `proposal canary status`, `evaluate`, `advance`, `reconcile`, `expand`, and `complete`, and canary routes can now be activated either with header matchers like `--canary-header X-Iket-Canary=identity-v2` or deterministic traffic splitting with `--canary-percent 10`. Step-based progression can be declared with repeated `--canary-step` flags such as `10, 25, 50, 100`. Canary proposals can also set rollout guards with `--canary-min-requests`, `--canary-max-error-rate`, and `--canary-max-p95-latency`. `proposal canary reconcile` can use those guardrails to decide the next action automatically, and proposals can enable background progression with `--canary-auto`, `--canary-auto-interval`, and `--canary-auto-reviewer`. When those guards fail during canary advance, reconcile, or completion, Iket automatically restores the pre-canary baseline configuration and marks the proposal as `canary_aborted`.

Proposal queue urgency can also be tuned in `security.mutationPolicy.proposalQueue`, with default thresholds plus per-environment overrides for `readyAgingAfter`, `readyOverdueAfter`, `blockedAgingAfter`, and `blockedOverdueAfter`. That lets `--urgency overdue` mean something stricter in `prod` than in `staging` without changing CLI usage.

`security.mutationPolicy.proposalQueue.notifications` can also run queue-digest notifications automatically in the background with `enabled`, `interval`, `minNotificationInterval`, `onlyOnSLABreach`, `onlyOnChange`, and optional `environments` scoping.

Rollout notifications can be configured through `security.notificationWebhooks`. Matching webhooks receive events like `proposal.approved`, `proposal.applied`, `proposal.promoted`, `proposal.rejected`, `proposal.shadow_ready`, `proposal.canary_started`, `proposal.canary_advanced`, `proposal.canary_completed`, `proposal.canary_aborted`, `proposal.digest`, `proposal.sla_breach`, `proposal.sla_stage_changed`, and `proposal.sla_resolved`. `format` can be `generic`, `slack`, or `teams`. Webhooks can also be filtered by `environments`, and `proposal.sla_breach` deliveries can require a minimum queue size with `minSLABreachCount`, a sustained consecutive breach streak with `minConsecutiveSLABreaches`, a minimum sustained breach age with `minSLABreachDuration`, or a severity floor with `minSLABreachTier` (`warning`, `elevated`, or `critical`). `slaBreachCooldown` adds repeat-suppression per webhook/environment so the same tier does not keep paging, while a higher breach tier can still break through immediately. For limiter incidents, `security.limitAlertBucketClasses` can define reusable tenant or bucket classes, `security.limiterClassPresets` can bundle live limiter budgets plus alert thresholds for those classes, and `security.limitAlertProfiles` can define reusable recipient profiles. A webhook can reference a limiter recipient profile with `limitAlertProfile` before optionally overriding `minLimitAlertSeverity`, `minLimitAlertBucketClassPriority`, `limitAlertTypes`, `limitAlertKeyTypes`, `limitAlertBucketClasses`, `limitAlertBucketIDRegex`, or `limitAlertCooldown` inline. Profiles and webhooks can now also carry `limitClassSnoozeExpiryCooldown`, `limitClassSnoozeExpiryWithin`, `limitClassSnoozeExpiryStages`, and `limitClassSnoozeEventTypes` so snooze wake-up warnings can be routed by debounce window, remaining snooze time, staged urgency, and snooze lifecycle event kind. `security.limitAlertBucketClasses` can now also carry class-specific `snoozeElevatedWithin`, `snoozeCriticalWithin`, `snoozeEventTypes`, and `snoozeExcludeFromDigest` so named classes can override the global snooze stage ladder, the lifecycle events emitted by `security.mutationPolicy.limitClassSnoozeNotifications`, and whether they appear inside the shared expiring digest. Those same limiter filters now apply to both route-level limiter incidents and class-aware events like `gateway.limit_class_alert` and `gateway.limit_class_alert_opened`, so teams can route or suppress named classes like `vip-jwt` independently, including by class priority through the emitted `bucket_class_priority`. Route `limitAlertPolicy.bucketPolicies` can reference the same named classes with `bucketClass` or consume a reusable limiter preset with `preset`. `limitAlertCooldown` now suppresses repeats per webhook plus bucket or class instead of suppressing different hot incident keys together. Queue SLA breach, stage-changed, and resolved events now also carry a shared `incident_id` so external systems can correlate one backlog incident across open, escalation, and close notifications. Webhooks can also be HMAC-signed with `signingSecret`, `signatureHeader`, and `timestampHeader`, retried with `retryCount` and `retryBackoff`, and inspected or replayed with `iket notification deliveries`, `iket notification show <id>`, `iket notification replay <id>`, and `iket notification replay-failed`.

Routes can also define multiple weighted backends by repeating `backend` entries with `weight` and optional `host` overrides. `timeout` adds per-backend attempt limits, `retryCount`, `retryBackoff`, `retryJitter`, and `retryStatusCodes` add route-level transient retry policy, and retries now default to idempotent methods unless `retryUnsafeMethods: true` is set. `hedgeDelay` adds request hedging for safe read methods, with `hedgeUnsafeMethods: true` available as an explicit opt-in. `shadowTrafficPercent` mirrors a slice of safe read traffic to an alternate healthy backend for validation, with `shadowUnsafeMethods: true` available as an explicit opt-in. Shadow requests are marked with `X-Iket-Shadow: true`, `iket gateway backends` surfaces per-backend shadow counters and latency comparison data, `iket gateway shadow-report` adds an aggregated service/route view for live-vs-shadow drift analysis, and `iket gateway shadow-evaluate` checks those grouped results against route-level guardrails like `shadowMinRequests`, `shadowMaxErrorRate`, and `shadowMaxLatencyDelta`. `adaptiveLatencyRouting: true` lets Iket bias steady-state traffic toward the faster healthy backend using observed latency EWMA instead of only static weights. `failureThreshold`, `cooldown`, `halfOpenMaxRequests`, and `recoverySuccessThreshold` now drive a configurable half-open circuit-breaker recovery flow. `outlierLatencyThreshold`, `outlierConsecutiveSlowResponses`, and `outlierCooldown` add slow-backend ejection. `healthCheckPath`, `healthInterval`, and `healthTimeout` add active health probing. Routes can also declare a native `rateLimitPolicy` with `requestsPerSecond`, `burst`, `keyBy`, `keyHeader`, and optional `exemptMethods`, which gives Iket first-class keyed rate limiting by `global`, `ip`, `header`, `api_key`, or `jwt_sub` without relying on the older global plugin path. `rateLimitPolicy.classPolicies` can now either define direct per-class overrides or reference `security.limiterClassPresets` with `preset`, so named tenant classes can reuse one live throttling profile across routes. Routes can also declare a native `concurrencyLimitPolicy` with `maxInFlight`, `keyBy`, `keyHeader`, optional `queueTimeout`, optional `maxQueueDepth`, and optional `exemptMethods`, plus the legacy `concurrent_calls` shorthand for a global cap. That gives Iket first-class in-flight protection by `global`, `ip`, `header`, `api_key`, or `jwt_sub`; `queueTimeout` lets a route wait briefly for capacity instead of always rejecting immediately; `maxQueueDepth` adds an explicit queue-full boundary that returns `503 Service Unavailable` once the wait queue is saturated; and `concurrencyLimitPolicy.classPolicies` can now also reference the same limiter class presets. `limitAlertPolicy` can now also override limiter incident grouping per route: `groupBy: route` keeps the default route aggregate, while `groupBy: bucket` splits overload alerts and incidents per anonymized limiter bucket so one hot tenant or API key is visible separately. `bucketPolicies` underneath that route policy can then apply stricter `minCount`, `minSeverity`, or per-limit-type thresholds to matching `jwt_sub`, `api_key`, `header`, or `ip` buckets without exposing raw bucket values in alert payloads, and can also reference `security.limiterClassPresets` with `preset` instead of repeating the alert thresholds inline. Routes can also declare a `cors` block with `allowedOrigins`, `allowedMethods`, `allowedHeaders`, `exposedHeaders`, `allowCredentials`, and `maxAge`; Iket answers browser preflight `OPTIONS` directly and adds CORS headers to actual responses without sending preflight requests upstream. For edge shaping, routes can also define `requestHeaders`, `removeRequestHeaders`, `requestRedactHeaders`, `requestJSONFields`, `removeRequestJSONFields`, `requestRedactJSONFields`, `requestBodyBlockRegex`, `requestBodyRequireRegex`, `requestPIIBlockTypes`, `queryParams`, `removeQueryParams`, `responseHeaders`, `removeResponseHeaders`, `responseJSONFields`, `removeResponseJSONFields`, `successResponseFields`, and `errorResponseFields`. Incoming query strings are preserved by default, then query add/remove transforms are applied before the upstream request is sent. Request and response JSON transforms apply only to top-level JSON object bodies, but their keys may use dot paths like `user.profile.realm`, indexed paths like `users[0].profile.realm`, and append syntax like `meta.tags[]`. Prefix a JSON transform value with `json:` to emit a typed JSON boolean, number, array, or object instead of a string. Add `transformWhenHeaders` / `transformWhenQueryParams` for exact-match gates, or `transformWhenHeaderRegex` / `transformWhenQueryRegex` for regex gates, to apply these edge transforms only to matching traffic. Add `transformMethods` to limit transforms to selected methods like `POST` or `PATCH`. Add `transformScopes` to limit that gating to `request_headers`, `query`, `request_json`, `response_headers`, or `response_json`; if omitted, the gate still applies to every transform on the route for compatibility. Response-side transforms can also be limited to specific statuses with `responseTransformStatusCodes` or `responseTransformStatusClasses` such as `2xx`, `4xx`, or `5xx`, and can also be gated by upstream response headers with `responseTransformWhenHeaders` or `responseTransformHeaderRegex`. `successResponseFields` can normalize backend success bodies into a stable JSON envelope, and `errorResponseFields` can do the same for 4xx/5xx errors. For agent-facing or security-sensitive routes, `requestRedactHeaders`, `requestRedactJSONFields`, `responseRedactHeaders`, `responseRedactJSONFields`, and `redactionValue` can mask secrets before data leaves or returns through the gateway, while `requestBodyBlockRegex` / `requestBodyRequireRegex` and `responseBodyBlockRegex` / `responseBodyRequireRegex` can enforce simple prompt/output content policy in both directions. `requestPIIBlockTypes` and `responsePIIBlockTypes` add named detectors for common sensitive classes like `email`, `phone`, `api_key`, and `card` without writing custom regex per route. Blocked policy responses now also emit `X-Iket-Policy-Hit` with a structured reason such as `request_content_policy`, `response_content_policy`, or `response_pii_policy`, and `iket gateway policy-hits` aggregates those guardrail hits by reason and route. `maxRequestBodyBytes` and `maxResponseBodyBytes` can also cap prompt and output sizes per route. For model-facing JSON APIs, top-level `aiPolicyPresets` can define an organization-wide shared catalog, service blocks can define `aiPolicyPresets` once as a service-local catalog, and routes can still define local `aiPolicyPresets` for closer-to-the-edge overrides. `aiPolicyPresetChain` composes multiple presets left-to-right, then `aiPolicyPreset` resolves one final shorthand layer against that merged catalog with precedence `route > service > global`, and local route fields can still add narrower overrides after that. `iket gateway route-policy` can inspect that resolved preset stack, the effective AI guardrails on the matched route, and which preset or local route field supplied each guardrail. `iket gateway route-policy-diff` can compare two matched routes and list only the differing effective guardrails with each side's value and source layer. Those presets can carry `allowedModels` / `modelField`, `allowedToolNames` / `toolField`, `maxMessages` / `messagesField`, `maxToolCalls` / `toolCallsField`, `maxInputTokens` / `inputTokensField`, `maxOutputTokens` / `outputTokensField`, `allowedUpstreamHosts`, `requiredRequestHeaders` / `requiredRequestHeaderRegex`, plus request and response content/PII policy. On GraphQL routes, `graphqlOperationPresets` can define reusable per-operation policy bundles, and `graphqlOperationPolicies` can reference one before overriding required request header, upstream host, model, tool, conversation, token, and request/response content-policy controls per named operation in addition to the existing variable, persisted-query, and complexity policy. `protocol` can also explicitly mark a route as `http`, `graphql`, `grpc`, or `websocket`, which lets Iket enforce GraphQL-compatible request patterns, `application/grpc` traffic, and websocket upgrade handshakes on a per-route basis. UDP is not proxied by this HTTP listener path. Transform values can interpolate `{{realm}}`, `{{request_id}}`, `{{query.name}}`, `{{var.name}}`, `{{header.Name}}`, and for response-side transforms `{{response_header.Name}}`, `{{response_status}}`, and `{{response_body}}`. The legacy `headers` field still works as a backward-compatible alias for upstream request header injection.

Proposal approvals and rejections can also record reviewer metadata:
- `--reviewer "ops-lead"`
- `--review-note "Approved after staging verification"`

---

## 📜 Logs
Read remote gateway logs.

| Command | Description |
|---------|-------------|
| `iket logs list` | Fetch recent logs from the remote gateway. |
| `iket logs tail` | Stream logs in real time. If SSE streaming is unavailable, it automatically falls back to polling recent logs. Both commands support `--service`, `--route`, and `--request-id` filters. |
| `iket logs trace` | Find a recent `request_id`, print a compact request summary, and follow only that request's logs. Supports `--service` and `--route` to narrow discovery. |

---

## 👥 Client & API Key Management
Manage client applications and their access permissions dynamically.

| Command | Description |
|---------|-------------|
| `iket client list` | List all registered client apps. |
| `iket client add <id>` | Register a new client (requires `--key`, supports `--group`, `--scopes`, `--name`). |
| `iket client delete <key>` | Remove a client app by its API key. |

---

## 🔐 Security & Safety

### Strict Mode
Enabled per context to prevent accidental changes in Production.
- **Trigger**: Any command that modifies state (create, set, delete, push, etc.).
- **Action**: Prompts for a manual `y/N` confirmation.
- **Revision metadata**: In strict contexts, mutating commands require `--label`, and high-impact commands like `--replace`, `delete`, `disable`, and `restore` also require both `--note` and `--change-ref` unless you use `--force`.
- **Bypass**: Use the global `--force` or `-f` flag for automation.

### mTLS Management
| Command | Description |
|---------|-------------|
| `iket cert status` | Check status of local certificates. |
| `iket cert gen` | Generate a new mTLS certificate chain (CA, Server, Client). |
| `iket cert regenerate-server` | Regenerate `server.crt` and `server.key` from the local CA using SANs from `--config` or explicit `--server-hostname` / `--server-name` / `--server-ip` flags. |
| `iket cert import` | Import `ca.crt`, `client.crt`, and `client.key` into a managed CLI context. The `--cert-dir` path must point at the source bundle you want to import, such as the server's copied Docker `certs/` directory, not just any existing `~/.iket/certs` folder. Hostname/SAN mismatch errors now include a hint to update `security.tls.serverNames` / `serverIPs`. |

`iket-cli` still works as a compatibility alias, but `iket` is now the primary client command. `iket-server` is the gateway binary.

---

## 💡 Global Flags
Applied to any command:
- `-f, --force`: Bypass Strict Mode confirmations.
- `--help`: View detailed help for any subcommand.

## Next Steps

<div class="docs-grid">
  <article class="doc-card">
    <h3><a href="{{ '/docs/management-api-integration/' | relative_url }}">Integrate the management API</a></h3>
    <p>Use this if you want automation or external systems to drive the same administrative surface as the CLI.</p>
  </article>
  <article class="doc-card">
    <h3><a href="{{ '/docs/production-deployment/' | relative_url }}">Harden production rollout</a></h3>
    <p>Use this next if your CLI workflows are heading toward a production deployment with mTLS and operational safeguards.</p>
  </article>
</div>
