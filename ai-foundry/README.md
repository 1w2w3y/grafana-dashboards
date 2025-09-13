# AI Foundry

Visulization for monitoring Azure AI Foundry usage and latency across model deployments in a single Grafana dashboard.

## Screenshots
![ai-foundry-top-section](https://github.com/1w2w3y/grafana-dashboards/raw/master/ai-foundry/ai-foundry-top-section.png)
![ai-foundry-latency](https://github.com/1w2w3y/grafana-dashboards/raw/master/ai-foundry/ai-foundry-latency.png)

## Issues and feedback
https://github.com/1w2w3y/grafana-dashboards/issues

## Features
- Totals at a glance
  - Inference Token count (Azure Monitor metric `TokenTransaction`)
  - Prompt Token count (metric `ProcessedPromptTokens`)
  - Completion Token count (metric `GeneratedTokens`)
- Latency trend
  - Average Time to Last Byte (metric `AzureOpenAITTLTInMS`) to observe end‑to‑end response latency by model deployment.
- Request and token trends
  - Time series for Requests (metric `AzureOpenAIRequests`)
  - Time series for Inference, Prompt, and Completion tokens over time
- Per‑deployment breakdown
  - All counters and time series are split by `ModelDeploymentName`, so you can compare individual deployments (for example: `gpt-5`, `gpt-4.1`, `o3`, etc.).
- Flexible scoping
  - Dashboard variables let you pick the Subscription and the target AI Foundry resource. The dashboard automatically derives the resource group, name, and region for accurate metric queries.
- Defaults optimized for operations
  - Default time range is the last 7 days, with legends and units configured for quick triage.

## How it works?
- Grafana queries Azure Monitor Metrics for the Azure AI Foundry (Cognitive Services) account using the Grafana Azure Monitor data source.
- The dashboard targets the `Microsoft.CognitiveServices/accounts` resource type and reads the following metrics:
  - `AzureOpenAIRequests` — number of requests received.
  - `TokenTransaction` — processed inference tokens.
  - `ProcessedPromptTokens` — prompt tokens processed.
  - `GeneratedTokens` — completion tokens generated.
  - `AzureOpenAITTLTInMS` — average time to last byte in milliseconds.
- All queries group by the `ModelDeploymentName` dimension so you can see usage and performance per model deployment.
- Variables:
  - `subscriptionId` — selected subscription.
  - `aiFoundryResourceId` — the Azure AI Foundry resource (resolved via Azure Resource Graph).
  - Hidden helper variables (`resourceGroup`, `resourceName`, `region`) are resolved automatically from the selected resource.
- Aggregations:
  - Token and request panels use `Total` over the selected time range.
  - Latency uses `Average`.

## Requirements
- Grafana 11.6+ with the Azure Monitor data source configured with access to the subscription that contains your AI Foundry account.

## Change history
- 9/8/2025 Initial version for Grafana 11.6
