# OpenCode

Monitoring dashboard for [OpenCode](https://opencode.ai) via Application Insights OpenTelemetry data, ingested through either the classic Application Insights pipeline or native OTLP and scoped exactly to `cloud_RoleName == "opencode"`. Tracks observed sessions, user prompt events, AI SDK LLM calls, reported token usage and cache read / write shares, tool executions and failures, prompt span latency, HTTP destinations, exceptions, and per-model / provider usage coverage. Missing usage, outcomes and exception text are reported as unknown rather than zero.

## Screenshots
![OpenCode – Main](https://github.com/1w2w3y/grafana-dashboards/raw/master/opencode/opencode-main.png)

## Issues and feedback
https://github.com/1w2w3y/grafana-dashboards/issues

## Features
- Summary KPIs
  - Observed Sessions, User Prompt Events, LLM Calls, Reported Tokens, Cache Read %, Exception Records
- Operational KPIs
  - Tool Executions, Tool Failure %, Token Reporting %, Prompt Span p95, Avg Reported Output / Call, Cache Write % (Reported)
- Activity Over Time
  - LLM Calls by Model
  - Token Usage Over Time (other input / cache-read input / output)
- Tool & Latency Breakdowns
  - Top Tools Called (calls, failures, unknown outcomes, total duration, p95)
  - HTTP Destinations (top 20 hosts for all instrumented HTTP client spans, with status classes)
  - LLM Latency by Model (calls, p50, p95)
- Sessions, Errors & Providers
  - Top Sessions by Tokens
  - Top Exceptions
  - Provider Split (calls, reported tokens)
- Iteration & Cache
  - LLM Finish Reason
  - Prompt Span Duration Over Time (avg / p95)
  - Cache Read Token Share
- Telemetry Coverage & Outcomes
  - LLM Outcomes Over Time (success / failure / unknown)
  - LLM Usage Coverage (per-model usage and cache-write reporting, last call)
  - Telemetry Signals (record counts and latest timestamp per signal)

## How it works
- Grafana queries Application Insights via the Azure Monitor data source using Log Analytics (KQL) queries against the `traces`, `dependencies`, `exceptions` and `customMetrics` tables, filtered by `cloud_RoleName == "opencode"`.
- LLM calls are `ai.streamText` AI SDK operation spans (nested `ai.streamText.doStream` attempts and separate `opencode.llm` observations are excluded); prompt latency uses `SessionPrompt.prompt` spans; tool executions are `Tool.execute` spans; HTTP destinations come from `http.client` and native HTTP client spans; user prompts are explicit `user_prompt` log events from the OpenCode telemetry hooks.
- Attributes: `ai.model.id`, `ai.model.provider`, `ai.response.finishReason`, `ai.usage.inputTokens` / `outputTokens` / `totalTokens`, `ai.usage.inputTokenDetails.cacheReadTokens` / `cacheWriteTokens`, `ai.telemetry.metadata.sessionId` / `session.id`, `tool.name`, `server.address` and `http.response.status_code`.
- Variables:
  - `am_ds` — Azure Monitor data source
  - `sub` — Azure subscription
  - `rg` — Resource group (resolved via Azure Resource Graph for `microsoft.insights/components`)
  - `res` — Application Insights resource
- Default time range is 7 days; default refresh 30 minutes.

## Requirements
- Grafana 11.6+ with the Azure Monitor data source configured with access to the subscription containing your Application Insights resource.
- OpenCode telemetry flowing into Application Insights via OpenTelemetry with `cloud_RoleName = "opencode"`. See [Ingest data into Application Insights via OpenTelemetry Collector](https://aka.ms/amg/dash-doc/otlp-appinsights) for the end-to-end setup — running the OTel Collector with the Azure Monitor Exporter, the OpenCode telemetry config (`serviceName: "opencode"`), and KQL verification queries.

## Change history
- 9/11/2026 Rebuilt for classic and native OTLP Application Insights ingestion with exact `cloud_RoleName == "opencode"` scoping. LLM metrics now count `ai.streamText` SDK operations only (nested doStream attempts and `opencode.llm` observations excluded) and treat missing usage as unknown rather than zero: Total Tokens became Reported Tokens, Avg Tokens / Prompt was replaced by Token Reporting %, and a per-model LLM Usage Coverage table was added. Sessions are now Observed Sessions (distinct session IDs across spans and logs), user prompts count explicit `user_prompt` events, prompt latency uses `SessionPrompt.prompt` spans, and Top WebFetch Hosts became HTTP Destinations covering all instrumented HTTP client spans. New Telemetry Coverage & Outcomes row with LLM Outcomes Over Time and Telemetry Signals.
- 5/8/2026 Initial version
