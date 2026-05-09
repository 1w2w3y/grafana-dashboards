# OpenCode

Monitoring dashboard for [OpenCode](https://opencode.ai) usage via Application Insights OpenTelemetry data. Tracks sessions, user prompts, LLM calls, token consumption, tool executions, prompt latency, cache hit ratio, exceptions, and per-model/provider breakdowns. Includes drill-down tables for top tools, top WebFetch hosts, top sessions, top exceptions, and finish reasons.

## Screenshots
![OpenCode – Main](https://github.com/1w2w3y/grafana-dashboards/raw/master/opencode/opencode-main.png)

## Issues and feedback
https://github.com/1w2w3y/grafana-dashboards/issues

## Features
- Summary KPIs
  - Total Sessions, User Prompts, LLM Calls, Total Tokens, Cache Read %, Exceptions
- Operational KPIs
  - Tool Executions, Tool Failure %, Avg Tokens / Prompt, Prompt p95 Latency, Avg Output Tokens / Call, Cache Write %
- Activity Over Time
  - LLM Calls by Model
  - Token Usage Over Time (input / output / cacheRead)
- Tool & Latency Breakdowns
  - Top Tools Called (calls, failures, totalSec, p95)
  - Top WebFetch Hosts (calls, 2xx/4xx/5xx)
  - LLM Latency by Model (calls, p95)
- Sessions, Errors & Providers
  - Top Sessions by Tokens
  - Top Exceptions
  - Provider Split (calls, totalTokens)
- Iteration & Cache
  - LLM Finish Reason
  - Prompt Latency Over Time
  - Cache Hit Ratio Over Time

## How it works
- Grafana queries Application Insights via the Azure Monitor data source using Log Analytics (KQL) queries against the `traces`, `dependencies`, and `exceptions` tables, filtered by `cloud_RoleName == "opencode"`. Metrics are grouped by `opencode.*` custom dimensions (run_id, model, provider, tool, session).
- Variables:
  - `am_ds` — Azure Monitor data source
  - `sub` — Azure subscription
  - `rg` — Resource group (resolved via Azure Resource Graph for `microsoft.insights/components`)
  - `res` — Application Insights resource
- Default time range is 7 days; default refresh 30 minutes.

## Requirements
- Grafana 11.6+ with the Azure Monitor data source configured with access to the subscription containing your Application Insights resource.
- OpenCode telemetry flowing into Application Insights via OpenTelemetry with `cloud_RoleName = "opencode"` and `opencode.*` custom dimensions. See [Ingest data into Application Insights via OpenTelemetry Collector](https://aka.ms/amg/dash-doc/otlp-appinsights) for the end-to-end setup — running the OTel Collector with the Azure Monitor Exporter, the OpenCode telemetry config (`serviceName: "opencode"`), and KQL verification queries.

## Change history
- 5/8/2026 Initial version
