# Codex

Monitoring dashboard for OpenAI Codex CLI (the `codex-app-server` service) via Application Insights OpenTelemetry data, ingested through either the classic Application Insights pipeline or native OTLP. Tracks completed turns, session initializations, response token usage (cached input and reasoning output), tool call volume, reliability and latency, sandbox policy, reported HTTP status codes, and telemetry coverage. Response usage and completed-turn counters are kept as separate populations, missing measurements are reported as unknown rather than zero, and tool metrics and tool logs are never added together for the same service role and tool.

## Screenshots
![Codex – Main](https://github.com/1w2w3y/grafana-dashboards/raw/master/codex/codex-main.png)

## Issues and feedback
https://github.com/1w2w3y/grafana-dashboards/issues

## Features
- Summary KPIs
  - Reported response tokens, Completed turns, Session initializations, Completed tool calls
  - Cached input share, Tool failure rate (known outcomes), p95 completed-turn duration, p95 response stream duration
- Models & Tokens
  - Response tokens by model (attributed through parent spans; unattributed usage stays visible)
  - Reasoning effort of completed turns
  - Completed turns by model (turn-level token counters, average tokens per turn, cache share)
  - Response input tokens (non-cached vs cached) and Response output tokens (reasoning vs non-reasoning)
  - Response tokens over time by model
- Usage Over Time
  - Session initializations over time
  - Completed turns over time
- Tool Health
  - Tool reliability by tool (completed calls, explicit failures, unknown outcomes)
  - Tool latency by tool (metric averages and log-based percentiles)
- Safety & Access
  - Sandbox policy observations (completed tool calls vs latest policy per conversation)
  - Initializations by service role
- High-usage turns
  - Highest-token completed turns (top 15, with thread and turn IDs)
- Operations & telemetry coverage
  - Completed-turn duration over time (p50 / p95)
  - Reported HTTP status codes
  - Telemetry coverage and freshness

## How it works
- Grafana queries Application Insights via the Azure Monitor data source using Log Analytics (KQL) queries against the `dependencies`, `requests`, `traces` and `customMetrics` tables. Every query filters on the exact service roles `codex-app-server` and `[coding-agent]/codex-app-server` (`cloud_RoleName`), so other applications sharing the resource are excluded.
- Turns, sessions and model requests come from Codex app-server spans: `session_init`, `session_task.turn`, `handle_responses`, `responses_websocket.stream_request`, `receiving_stream` and `try_run_sampling_request`. Token usage is read from `codex.usage.total_tokens`, `codex.usage.reasoning_output_tokens`, `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`, `gen_ai.usage.cache_read.input_tokens` and the turn-level `codex.turn.token_usage.*` counters; reasoning effort from `codex.turn.reasoning_effort`.
- Tool calls use the `codex.tool.call` and `codex.tool.call.duration_ms` metrics where a service role/tool reports them, otherwise `codex.tool_result` log events deduplicated by service role, conversation ID and call ID. Sandbox policy comes from the `sandbox_policy` attribute on tool metrics and on `codex.conversation_starts` logs.
- No cost estimate is shown: the observed models have no supported price list, and unknown prices are not converted to $0.
- Variables:
  - `am_ds` — Azure Monitor data source
  - `sub` — Azure subscription
  - `rg` — Resource group (resolved via Azure Resource Graph for `microsoft.insights/components`)
  - `res` — Application Insights resource
- Default time range is 7 days; default refresh 30 minutes.

## Requirements
- Grafana 11.6+ with the Azure Monitor data source configured with access to the subscription containing your Application Insights resource.
- Codex CLI telemetry data flowing into Application Insights via OpenTelemetry, with the app server reporting as `cloud_RoleName` `codex-app-server` (or `[coding-agent]/codex-app-server`). See [Ingest data into Application Insights via OpenTelemetry Collector](https://aka.ms/amg/dash-doc/otlp-appinsights) for the end-to-end setup — running the OTel Collector with the Azure Monitor Exporter, the Codex CLI OTLP configuration (`OTEL_EXPORTER_OTLP_ENDPOINT`), and KQL verification queries.

## Change history
- 9/11/2026 Rebuilt around the Codex app-server span model with support for both classic and native OTLP Application Insights ingestion, scoped to the exact `codex-app-server` service roles. Summary KPIs now separate reported response tokens from completed-turn and session-initialization counts. New Models & Tokens row: response tokens by model, reasoning effort, per-model turn table, and cached/non-cached input and reasoning/non-reasoning output token series. Tool reliability and latency combine tool metrics with deduplicated tool logs without double counting. New High-usage turns and Operations & telemetry coverage rows (highest-token turns, completed-turn duration over time, reported HTTP status codes, telemetry coverage and freshness). Removed the approval-outcomes, first-token latency, prompt-cache-hit-rate and sessions-by-client panels.
- 5/8/2026 Initial version
