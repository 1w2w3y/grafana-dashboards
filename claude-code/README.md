# Claude Code

Monitoring dashboard for Claude Code usage and performance via Application Insights OpenTelemetry data. Tracks cost, token consumption, API requests, tool usage, tool success rate and latency, MCP server health, the most expensive prompts, rejected tool calls, sessions, active coding time, commits, code edit acceptance rate, per-user breakdown, and error details.

## Screenshots
![Claude Code – Main](https://github.com/1w2w3y/grafana-dashboards/raw/master/claude-code/claude-code-main.png)

## Issues and feedback
https://github.com/1w2w3y/grafana-dashboards/issues

## Features
- Summary KPIs
  - Total Cost, Total Sessions, User Prompts, API Requests, API Errors, Active Time
- Cost & Token Trends
  - Daily Cost by Model
  - Daily Token Usage by Type (input, output, cacheRead, cacheCreation)
- Model Usage Breakdown
  - Token Usage by Model
  - Cost by Model
  - API Requests by Model
- Reliability & Cost Efficiency
  - Tool Success Rate & p95 Latency (per tool)
  - MCP Server Health (connection outcomes per server)
  - Top Expensive Prompts (top 20 prompts by cost)
  - Tool Reject Counts (tool calls the user did not accept)
- Tool Usage Analytics
  - Daily Tool Usage (stacked by tool name)
- Productivity Metrics
  - Daily Commits
  - Daily User Prompts
  - Code Edit Accept Rate
- Session & Activity
  - Daily Sessions
  - Active Time per Day
- Per-User Metrics
  - User Summary
  - Per-User Cost by Model
- Errors & Reliability
  - API Errors Over Time
  - Error Details

## How it works
- Grafana queries Application Insights via the Azure Monitor data source using Log Analytics (KQL) queries against the `customMetrics` table (`claude_code.*` metrics such as `claude_code.lines_of_code.count`) and the `traces` table (Claude Code log events `api_request`, `api_error`, `user_prompt`, `tool_result` and `tool_decision`).
- Attributes used: `model`, `cost_usd`, `input_tokens`, `output_tokens`, `cache_read_tokens`, `cache_creation_tokens`, `duration_ms`, `tool_name`, `success`, `decision`, `server_name`, `status`, `status_code`, `error`, `prompt.id`, `session.id` and `user.id`. Top Expensive Prompts joins `user_prompt` to `api_request` events via `prompt.id`.
- Variables:
  - `am_ds` — Azure Monitor data source
  - `sub` — Azure subscription
  - `rg` — Resource group (resolved via Azure Resource Graph for `microsoft.insights/components`)
  - `res` — Application Insights resource
- Default time range is 7 days; default refresh 30 minutes.

## Requirements
- Grafana 11.6+ with the Azure Monitor data source configured with access to the subscription containing your Application Insights resource.
- Claude Code telemetry data flowing into Application Insights via OpenTelemetry. See [Ingest data into Application Insights via OpenTelemetry Collector](https://aka.ms/amg/dash-doc/otlp-appinsights) for the end-to-end setup — running the OTel Collector with the Azure Monitor Exporter, the Claude Code env vars (`CLAUDE_CODE_ENABLE_TELEMETRY`, `OTEL_EXPORTER_OTLP_ENDPOINT`, etc.) in `settings.json`, and KQL verification queries against `customMetrics`.

## Change history
- 9/11/2026 New Reliability & Cost Efficiency row: Tool Success Rate & p95 Latency, MCP Server Health, Top Expensive Prompts (user_prompt joined to api_request via `prompt.id`) and Tool Reject Counts. Tool Usage Analytics consolidated into a single Daily Tool Usage stacked series (replacing Tool Usage (Top Tools) and Daily Tool Usage (Stacked)); removed Lines of Code (Added vs Removed); Code Edit Accept Rate moved to Productivity Metrics.
- 5/1/2026 Point data-ingestion link to the published Microsoft Learn guide (aka.ms/amg/dash-doc/otlp-appinsights)
- 3/23/2026 Initial version
