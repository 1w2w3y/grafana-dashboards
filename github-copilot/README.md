# GitHub Copilot

Monitoring dashboard for GitHub Copilot usage and performance via Application Insights OpenTelemetry data, covering both VS Code Copilot Chat and the GitHub Copilot CLI. Tracks agent runs, LLM requests and token consumption, model latency (request duration and time-to-first-token), chat sessions, tool calls and tool latency, feature adoption, exceptions, tool failures, LLM call outcomes and telemetry freshness. Includes trace drill-down for debugging individual Copilot interactions.

## Screenshots
![GitHub Copilot – Main](https://github.com/1w2w3y/grafana-dashboards/raw/master/github-copilot/github-copilot-main.png)

## Issues and feedback
https://github.com/1w2w3y/grafana-dashboards/issues

## Features
- Summary
  - Agent Summary Statistics (agent runs and their per-run token roll-ups)
  - Chat & Tool Summary (LLM calls, chat sessions, tool calls)
- Usage Over Time
  - Operations Over Time (chat, execute_tool, invoke_agent, embeddings)
  - Token Consumption Over Time by Model
- Model Performance
  - Model Usage Distribution
  - LLM Request Duration by Model (Avg / P90)
  - Time to First Token by Model (P50 / P90)
- Agent & Feature Activity
  - Copilot Feature Usage
  - Top Tool Calls
  - Sessions, Agent Runs, LLM & Tool Calls Over Time
  - Tool Latency by Tool (P50 / P90)
  - Agent Run Duration by Source and Agent (with runs longer than 10 minutes)
- Token Efficiency & Cost
  - Token Usage by Model (including Cache Hit %)
  - Input vs Output Tokens Over Time
- Errors & Reliability
  - Exceptions Over Time
  - Exception Details
  - Tool Call Failures by Tool
  - LLM Call Outcomes Over Time (completed vs no response recorded)
  - Telemetry Freshness (last span per source)
- Trace Drill-Down
  - Recent Operations table (click a row to select a trace)
  - Trace Visualization

## How it works
- Grafana queries Application Insights via the Azure Monitor data source using Log Analytics (KQL) queries against the `dependencies` and `exceptions` tables, plus an Azure Traces query for the trace visualization.
- Telemetry sources are selected by `cloud_RoleName`: `copilot-chat` for VS Code Copilot and `github-copilot` for the Copilot CLI. Operations are classified by `gen_ai.operation.name` (`chat` = model request, `execute_tool` = tool call, `invoke_agent` = agent run, `embeddings`), models by `gen_ai.request.model` / `gen_ai.response.model`, features by `gen_ai.agent.name`, tools by `gen_ai.tool.name`, sessions by `copilot_chat.chat_session_id` / `gen_ai.conversation.id`.
- Tokens come from `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`, `gen_ai.usage.cache_read.input_tokens` and `gen_ai.usage.reasoning.output_tokens`. Copilot CLI sub-agent roll-up spans are excluded from request counts but included in token totals.
- Latency: request duration uses span duration for VS Code and `github.copilot.server_duration` for the CLI (its chat spans also wrap the turn's tool calls); time to first token uses `copilot_chat.time_to_first_token` (ms) for VS Code and `gen_ai.response.time_to_first_chunk` (seconds, converted to ms) for the CLI.
- Variables:
  - `am_ds` — Azure Monitor data source
  - `sub` — Azure subscription
  - `rg` — Resource group (resolved via Azure Resource Graph for `microsoft.insights/components`)
  - `res` — Application Insights resource
  - `source` — multi-select custom variable: `VS Code Copilot` (`copilot-chat`) and `Copilot CLI` (`github-copilot`); both selected by default
  - `copilotTraceId` — hidden variable for trace drill-down, populated from `invoke_agent` spans of the selected sources
- Default time range is 7 days; default refresh 30 minutes.

## Requirements
- Grafana 11.6+ with the Azure Monitor data source configured with access to the subscription containing your Application Insights resource.
- GitHub Copilot telemetry data flowing into Application Insights via OpenTelemetry. See [Ingest data into Application Insights via OpenTelemetry Collector](https://aka.ms/amg/dash-doc/otlp-appinsights) for the end-to-end setup — running the OTel Collector with the Azure Monitor Exporter, the VS Code `settings.json` keys (`github.copilot.chat.otel.enabled`, `otlpEndpoint`, etc.), and KQL verification queries against the `dependencies` table (`cloud_RoleName == "copilot-chat"`). Copilot CLI telemetry arrives with `cloud_RoleName == "github-copilot"`.

## Change history
- 9/11/2026 Added GitHub Copilot CLI as a second telemetry source alongside VS Code Copilot, selectable through the new `source` variable; panels split or filter by source. Latency panels rebuilt: LLM Request Duration by Model uses the model-service duration for the CLI so tool time inside a turn is not counted, and time to first token converts the CLI's seconds to ms. New panels: Tool Latency by Tool, Agent Run Duration by Source and Agent, Tool Call Failures by Tool, LLM Call Outcomes Over Time and Telemetry Freshness; Sessions, Turns & Tool Calls Over Time became Sessions, Agent Runs, LLM & Tool Calls Over Time. Removed the Latency Trends row (Avg Response Duration Over Time by Model, Time to First Token Over Time). Sub-agent roll-up spans are excluded from request counts.
- 5/1/2026 Point data-ingestion link to the published Microsoft Learn guide (aka.ms/amg/dash-doc/otlp-appinsights)
- 3/23/2026 Initial version
