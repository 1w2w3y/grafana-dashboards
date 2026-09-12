# OpenClaw

Monitoring dashboard for the OpenClaw gateway via Application Insights OpenTelemetry data, ingested through either the classic Application Insights pipeline or native OTLP. Tracks message processing across channels (including cron runs), model requests and token usage, request payload sizes and time to first byte, tool executions and shell exec outcomes, gateway memory and event-loop health, liveness warnings and stuck sessions, context size, run triggers and harness runs. Includes recent-messages and recent-failures tables with trace IDs for correlation.

## Screenshots
![OpenClaw – Main](https://github.com/1w2w3y/grafana-dashboards/raw/master/openclaw/openclaw-main.png)

## Issues and feedback
https://github.com/1w2w3y/grafana-dashboards/issues

## Features
- Summary
  - Message Summary (messages, failures, processing duration)
  - Model Request Summary (requests and tokens from request spans with valid usage)
  - Usage Coverage (share of model requests reporting valid token usage)
  - Cached Input Share
  - Model Request Payload (request / response bytes)
- Latency & Phases
  - Request Time to First Byte (avg / p95)
  - Phase Timings (per-phase duration from diagnostic phase events)
- Message Activity
  - Messages by Channel
  - Processing Time by Channel (cron separated from interactive channels)
- LLM Token Usage
  - Request Token Usage (uncached input / cached input / cache write / output)
  - Model Request Outcomes (success / failure)
  - Model Request Breakdown (per model / provider)
- Tools & Exec
  - Tools: Volume, Failures & Latency
  - Shell Exec Outcomes (success / failure / timeout / unknown)
- Gateway Health
  - Memory Observations (heap / RSS)
  - Event Loop During Liveness Warnings (p99 delay, utilization)
- Health & Errors
  - Errors & Warnings (latest 100)
  - Liveness & Stuck-Session Events
  - Warning & Error Volume
  - Recent Failures & Stuck Sessions (latest 100)
- Context & Activity
  - Context Size (avg kchars: prompt / history / system prompt)
  - Processing & Delivery Events
  - Run Triggers
  - Harness Runs (per plugin, with completion rate and item totals)
- Channel Details
  - Message Outcomes by Channel
  - Recent Messages (latest 100, with trace ID)
  - Telemetry Coverage (last observed trace / log timestamps)

## How it works
- Grafana queries Application Insights via the Azure Monitor data source using Log Analytics (KQL) queries against the `dependencies`, `requests`, `traces` and `customMetrics` tables. Every query is restricted to `cloud_RoleName == "openclaw-gateway"`, so other applications sharing the resource are excluded.
- Spans and events used: `openclaw.message.processed` and `openclaw.message.delivery` (messages), `openclaw.model.call` (model requests; harness turns and aggregate usage reports are excluded from token totals), `openclaw.tool.execution`, `openclaw.exec`, `openclaw.run`, `openclaw.context.assembled`, `openclaw.diagnostic.phase`, `openclaw.liveness.warning`, `openclaw.session.stuck`, `openclaw.harness.run` and `openclaw.memory.pressure`, plus the `openclaw.memory.heap_used_bytes` / `openclaw.memory.rss_bytes` custom metrics.
- Attributes: `openclaw.channel`, `openclaw.outcome`, `openclaw.model` / `openclaw.provider` (with `gen_ai.request.model` / `gen_ai.provider.name`), token usage from `gen_ai.usage.*` and `openclaw.model_call.usage.*`, `openclaw.model_call.time_to_first_byte_ms`, `openclaw.model_call.request_bytes` / `response_bytes`, `openclaw.toolName` / `gen_ai.tool.name`, `openclaw.exec.timed_out`, `openclaw.trigger`, `openclaw.harness.*`, `openclaw.liveness.*`, `openclaw.context.*_chars` and `openclaw.ageMs`.
- No cost estimate is shown; Usage Coverage replaces it so that unknown usage is reported rather than priced.
- Variables:
  - `am_ds` — Azure Monitor data source
  - `sub` — Azure subscription
  - `rg` — Resource group (resolved via Azure Resource Graph for `microsoft.insights/components`)
  - `res` — Application Insights resource
- Default time range is 7 days; default refresh 30 minutes.

## Requirements
- Grafana 11.6+ with the Azure Monitor data source configured with access to the subscription containing your Application Insights resource.
- OpenClaw gateway telemetry flowing into Application Insights via OpenTelemetry with `cloud_RoleName = "openclaw-gateway"` and `openclaw.*` span names and attributes. See [Ingest data into Application Insights via OpenTelemetry Collector](https://aka.ms/amg/dash-doc/otlp-appinsights) for the end-to-end setup — running the OTel Collector with the Azure Monitor Exporter, the OpenClaw gateway telemetry config (`serviceName: "openclaw-gateway"`), and KQL verification queries.

## Change history
- 9/11/2026 Rebuilt for classic and native OTLP Application Insights ingestion, with every query scoped to `openclaw-gateway`. Replaced the estimated-cost panel with Usage Coverage; token totals now exclude aggregate usage events and harness turns, and count cached input once. New rows: Latency & Phases (time to first byte, diagnostic phase timings), Tools & Exec (tool volume, failures and latency; shell exec outcomes), Gateway Health (memory observations, event loop during liveness warnings) and Context & Activity (context size, processing vs delivery events, run triggers, harness runs). Added Model Request Payload, Model Request Outcomes, Message Outcomes by Channel, Recent Failures & Stuck Sessions, Warning & Error Volume and Telemetry Coverage. Cron message processing is shown separately from interactive channels.
- 5/1/2026 Point data-ingestion link to the published Microsoft Learn guide (aka.ms/amg/dash-doc/otlp-appinsights)
- 4/17/2026 Initial version
