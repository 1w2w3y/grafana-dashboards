# Gemini CLI

Monitoring dashboard for [Gemini CLI](https://github.com/google-gemini/gemini-cli) usage via Application Insights OpenTelemetry data. Tracks sessions, user prompts, LLM calls and errors, token composition, tool calls (native vs MCP), latency percentiles, quota throttling (429s), cost per session, and per-language file activity.

## Screenshots
![Gemini CLI – Main](https://github.com/1w2w3y/grafana-dashboards/raw/master/gemini-cli/gemini-cli-main.png)

## Issues and feedback
https://github.com/1w2w3y/grafana-dashboards/issues

## Features
- Summary
  - Message Summary (Messages, Unique Chats, Avg / P95 Response)
  - LLM & Health Summary (LLM Calls, Token totals, error rate)
  - Estimated Cost (USD)
  - Cache Hit %
- Reliability & Latency
  - LLM Calls & Errors by Status
  - LLM Latency p50 / p95 / p99
  - Quota Reset Delay Distribution (429s)
- Tool Analytics
  - Tools — Calls, Failures, Latency
  - Native vs MCP Tool Latency
  - Tool Approval Decisions
- Sessions & Users
  - Sessions
  - Top Users / Hosts
  - Cost Per Session (Top)
- Token Mix & Operations
  - Token Composition Over Time
  - Startup Phases (avg ms)
  - File Reads by Language

## How it works
- Grafana queries Application Insights via the Azure Monitor data source using Log Analytics (KQL) queries against OpenTelemetry data emitted by Gemini CLI. Metrics and traces are grouped by `gemini_cli.*` custom dimensions (model, tool, status, language, session, user).
- Variables:
  - `am_ds` — Azure Monitor data source
  - `sub` — Azure subscription
  - `rg` — Resource group (resolved via Azure Resource Graph for `microsoft.insights/components`)
  - `res` — Application Insights resource
- Default time range is 7 days; default refresh 30 minutes.

## Requirements
- Grafana 11.6+ with the Azure Monitor data source configured with access to the subscription containing your Application Insights resource.
- Gemini CLI telemetry flowing into Application Insights via OpenTelemetry. See [Ingest data into Application Insights via OpenTelemetry Collector](https://aka.ms/amg/dash-doc/otlp-appinsights) for the end-to-end setup — running the OTel Collector with the Azure Monitor Exporter, configuring Gemini CLI telemetry to OTLP, and KQL verification queries.

## Change history
- 5/8/2026 Initial version
