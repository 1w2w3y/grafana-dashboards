# Claude Code

Monitoring dashboard for Claude Code usage and performance via Application Insights OpenTelemetry data. Tracks cost, token consumption, API requests, tool usage, sessions, active coding time, lines of code, commits, code edit acceptance rate, per-user breakdown, and error details.

## Screenshots
![Claude Code – Main](https://github.com/1w2w3y/grafana-dashboards/raw/master/claude-code/claude-code-main.png)

## Issues and feedback
https://github.com/1w2w3y/grafana-dashboards/issues

## Features
- Summary KPIs
  - Total Cost, Total Sessions, User Prompts, API Requests, API Errors, Active Time
- Cost & Token Trends
  - Daily Cost by Model
  - Daily Token Usage by Type
- Model Usage Breakdown
  - Token Usage by Model
  - Cost by Model
  - API Requests by Model
- Tool Usage Analytics
  - Tool Usage (Top Tools)
  - Daily Tool Usage (Stacked)
- Productivity Metrics
  - Lines of Code (Added vs Removed)
  - Daily Commits
  - Daily User Prompts
- Session & Activity
  - Daily Sessions
  - Active Time per Day
  - Code Edit Accept Rate
- Per-User Metrics
  - User Summary
  - Per-User Cost by Model
- Errors & Reliability
  - API Errors Over Time
  - Error Details

## How it works
- Grafana queries Application Insights via the Azure Monitor data source using Log Analytics (KQL) queries against OpenTelemetry data.
- Variables:
  - `am_ds` — Azure Monitor data source
  - `sub` — Azure subscription
  - `rg` — Resource group (resolved via Azure Resource Graph for `microsoft.insights/components`)
  - `res` — Application Insights resource
- Default time range is 7 days.

## Requirements
- Grafana 11.6+ with the Azure Monitor data source configured with access to the subscription containing your Application Insights resource.
- Claude Code telemetry data flowing into Application Insights via OpenTelemetry. See [Ingest data into Application Insights via OpenTelemetry Collector](https://aka.ms/amg/dash-doc/otlp-appinsights) for the end-to-end setup — running the OTel Collector with the Azure Monitor Exporter, the Claude Code env vars (`CLAUDE_CODE_ENABLE_TELEMETRY`, `OTEL_EXPORTER_OTLP_ENDPOINT`, etc.) in `settings.json`, and KQL verification queries against `customMetrics`.

## Change history
- 5/1/2026 Point data-ingestion link to the published Microsoft Learn guide (aka.ms/amg/dash-doc/otlp-appinsights)
- 3/23/2026 Initial version
