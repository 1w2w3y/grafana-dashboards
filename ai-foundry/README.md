# AI Foundry

Visualization for monitoring Azure AI Foundry usage, latency, and estimated cost across model deployments in a single Grafana dashboard. Try this for free inside Azure portal: http://aka.ms/amg/dash/ai-foundry

## Screenshots
![ai-foundry-top-section](https://github.com/1w2w3y/grafana-dashboards/raw/master/ai-foundry/ai-foundry-top-section.png)
![ai-foundry-latency](https://github.com/1w2w3y/grafana-dashboards/raw/master/ai-foundry/ai-foundry-latency.png)

## Issues and feedback
https://github.com/1w2w3y/grafana-dashboards/issues

## Features
- Totals at a glance
  - Estimated cost per model, computed from input/output token usage with built-in per-model price mappings (77 models across OpenAI GPT and o-series, Anthropic Claude, xAI Grok, Moonshot Kimi, DeepSeek, and Mistral)
  - Input and Output token totals per model deployment; deployments with zero traffic in the selected window are hidden automatically
- Request and token trends
  - Model Requests over time, split by deployment
  - Non-200 Model Requests (throttling & errors), split by HTTP status code
  - Input, Output, and Total token time series
- Latency and throughput
  - Average Time to Last Byte for all model families, plus the OpenAI-specific latency metric
  - Tokens per Second (OpenAI models)
  - Token Cache Match Rate (OpenAI models) to see how much of your prompt traffic hits the context cache
- Per-deployment breakdown
  - All counters and time series are split by `ModelDeploymentName` (cost uses `ModelName`), so you can compare individual deployments (for example: `gpt-5.4`, `grok-4.3`, `Kimi-K2.7-Code`, `DeepSeek-V4-Pro`).
- Flexible scoping
  - Dashboard variables let you pick the data source, Subscription, Resource Group, and the target AI Foundry resource. The region is derived automatically.
- Defaults optimized for operations
  - Default time range is the last 7 days, with legends and units configured for quick triage.

## How it works?
- Grafana queries Azure Monitor Metrics for the Azure AI Foundry (Cognitive Services) account using the Grafana Azure Monitor data source.
- The dashboard targets the `Microsoft.CognitiveServices/accounts` resource type and reads the following metrics:
  - `ModelRequests` — model inference requests; also split by `StatusCode` for the throttling/errors panel.
  - `InputTokens` / `OutputTokens` / `TotalTokens` — token consumption.
  - `TimeToLastByte` — average time to last byte across all model families.
  - `AzureOpenAITTLTInMS` — average time to last byte for OpenAI models.
  - `AzureOpenAITokenPerSecond` — output throughput for OpenAI models.
  - `AzureOpenAIContextTokensCacheMatchRate` — context-cache hit rate for OpenAI models.
- Queries group by the `ModelDeploymentName` dimension (the cost panel uses `ModelName`) so you can see usage and performance per model deployment.
- The Estimated Cost panel multiplies per-model input and output token totals by maintained price mappings inside the panel's transformations. The hidden `priceScale` custom variable applies a uniform multiplier to every cost tile (for example `0.8` for a 20% negotiated discount).
- Variables:
  - `am_ds` — the Azure Monitor data source.
  - `sub`, `rg`, `res` — subscription, resource group, and AI Foundry resource, each resolved from the previous selection.
  - Hidden helpers: `region` (derived from the selected resource) and `priceScale`.
- Aggregations:
  - Token and request panels use `Total` over the selected time range.
  - Latency, throughput, and cache-match panels use `Average`.

## Requirements
- Grafana 11.6+ with the Azure Monitor data source configured with access to the subscription that contains your AI Foundry account.

## Change history
- 9/8/2025 Initial version for Grafana 11.6
- 9/30/2025 Update to more suitable metrics and improved layout
- 10/10/2025 add estimated cost panel
- 1/5/2026 update estimated cost with more models like GPT-5.2 and Claude Opus 4.5; add a price scale variable for adjusting cost estimates based on negotiated pricing.
- 1/15/2026 add cost calculation for GPT-5.2-codex; add labels with values for each panel
- 8/10/2026 Layout rework: full-width Input/Output token stat rows with zero-value tiles hidden; new Non-200 Model Requests (throttling & errors) and Tokens per Second panels; estimated-cost price table expanded to 77 models (adds GPT-5.3/5.4/5.5/5.6 tiers, Claude 4.6-5 including Fable 5, Grok 4/4.3 and Fast variants, grok-code-fast-1, Kimi K2.5-K3, DeepSeek V4, gpt-oss-120b, gpt-image-2); refreshed GPT-5.6 Terra/Luna prices.
