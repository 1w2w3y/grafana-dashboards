# GitHub Copilot

Monitoring dashboard for GitHub Copilot usage and performance via Application Insights OpenTelemetry data. Tracks agent operations, LLM token consumption, model latency (response duration and time-to-first-token), chat sessions, tool calls, feature adoption, and errors. Includes trace drill-down for debugging individual Copilot interactions.

## Screenshots
![GitHub Copilot – Main](https://github.com/1w2w3y/grafana-dashboards/raw/master/github-copilot/github-copilot-main.png)

## Issues and feedback
https://github.com/1w2w3y/grafana-dashboards/issues

## Features
- Summary
  - Agent Summary Statistics (total operations, tokens, avg response time)
  - Chat & Tool Summary (total calls, tokens, avg TTFT)
- Usage Over Time
  - Operations Over Time
  - Token Consumption Over Time by Model
- Model Performance
  - Model Usage Distribution
  - Response Duration by Model (Avg / P90)
  - Time to First Token by Model (P50 / P90)
- Latency Trends
  - Avg Response Duration Over Time by Model
  - Time to First Token (P50 / P90) Over Time
- Agent & Feature Activity
  - Copilot Feature Usage
  - Top Tool Calls
  - Sessions, Turns & Tool Calls Over Time
- Token Efficiency & Cost
  - Token Usage by Model (Table)
  - Input vs Output Tokens Over Time
- Errors & Reliability
  - Exceptions Over Time
  - Exception Details
- Trace Drill-Down
  - Select a trace by operation
  - Recent Operations table
  - Trace Visualization

## How it works
- Grafana queries Application Insights via the Azure Monitor data source using Log Analytics (KQL) queries against OpenTelemetry data.
- Variables:
  - `am_ds` — Azure Monitor data source
  - `sub` — Azure subscription
  - `rg` — Resource group (resolved via Azure Resource Graph for `microsoft.insights/components`)
  - `res` — Application Insights resource
  - `copilotTraceId` — hidden variable for trace drill-down, populated from `dependencies` table filtered by `service.name == "copilot-chat"`
- Default time range is 7 days.

## Requirements
- Grafana 11.6+ with the Azure Monitor data source configured with access to the subscription containing your Application Insights resource.
- GitHub Copilot telemetry data flowing into Application Insights via OpenTelemetry. See [Dashboard Data Ingestion](https://github.com/1w2w3y/grafana-dashboards/blob/master/data-ingestion.md) for the end-to-end setup (OTel Collector + per-app OTLP config).

## Change history
- 3/23/2026 Initial version
