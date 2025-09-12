# LiteLLM Trace Details

Detailed diagnostics for a single LiteLLM request trace using Azure Monitor (Application Insights) in Grafana. Inspect the end-to-end distributed trace, model and token usage, completion output, and the full prompt conversation captured by LiteLLM.

## Screenshots
![litellm-trace-overview](https://github.com/1w2w3y/grafana-dashboards/raw/master/litellm-trace/litellm-trace-overwiew-2509.png)
![litellm-trace-llm-output](https://github.com/1w2w3y/grafana-dashboards/raw/master/litellm-trace/litellm-trace-llm-output-2509.png)
![litellm-trace-only](https://github.com/1w2w3y/grafana-dashboards/raw/master/litellm-trace/litellm-trace-only-2509.png)
![litellm-trace-llm-prompts](https://github.com/1w2w3y/grafana-dashboards/raw/master/litellm-trace/litellm-trace-llm-prompts-2509.png)

## Features
- End-to-end distributed trace
  - Visualizes the span timeline for the selected request (`operation_Id`), including requests, dependencies, exceptions, custom events, and traces.
  - Helpful for diagnosing latency issues and downstream call behavior.
- LLM output and token summary
  - Shows model (`gen_ai.request.model`), total tokens (`llm.usage.total_tokens`), and the first completion content with its finish reason.
- Full prompt conversation
  - Reconstructs the entire conversation turns from `customDimensions` keys `gen_ai.prompt.N.role` and `gen_ai.prompt.N.content`, ordered by turn index.
  - Displays an easy-to-read conversation with the total number of turns.
- Flexible scoping
  - Dashboard variables let you pick the Subscription, the target Application Insights resource, and the Trace Id (`operation_Id`) to inspect.
- Defaults optimized for investigations
  - Default time range is the last 1 hour; suitable for ad-hoc deep dives on recent requests.

## How it works?
- Data source: Grafana Azure Monitor data source.
  - Azure Log Analytics queries against your Application Insights resource.
  - Azure Traces query to render the distributed trace for the selected `operation_Id`.
- Trace and details logic:
  - The details panels read from the `dependencies` table and filter LiteLLM items:
    - `where name == "litellm_request"`.
    - Filter by the selected trace: `where operation_Id == "$litellmTraceId"`.
  - Fields extracted from `customDimensions`:
    - Model: `customDimensions["gen_ai.request.model"]`
    - System prompt: `customDimensions["gen_ai.system"]`
    - Tokens: `customDimensions["gen_ai.usage.prompt_tokens"]`, `customDimensions["gen_ai.usage.completion_tokens"]`, `customDimensions["llm.usage.total_tokens"]`
    - Completion: `customDimensions["gen_ai.completion.0.role"]`, `customDimensions["gen_ai.completion.0.content"]`, `customDimensions["gen_ai.completion.0.finish_reason"]`
  - Prompt reconstruction:
    - `mv-expand bag_keys(customDimensions)` and regex match `^gen_ai\.prompt\.(\d+)\.(role|content)$` to capture each turn.
    - Build per-index objects, order by `index`, and generate readable markdown:
      - Header includes total turns, followed by each turn as role/content, separated by `---`.
- Variables:
  - `subscriptionId` — selected subscription.
  - `applicationInsightsResourceId` — the Application Insights resource ID (discovered via Azure Resource Graph scoped by the selected subscription).
  - `litellmTraceId` — the trace identifier (`operation_Id`) sourced from `dependencies | distinct operation_Id` for the selected resource and dashboard time.

## Requirements
- Grafana 11.6+ with the Azure Monitor data source (11.6.x).
- Business Text panel plugin: `marcusolsson-dynamictext-panel` (used to render model/tokens/output and the prompt conversation).
- Read access to the target subscription and Application Insights component.
- LiteLLM must emit telemetry to Application Insights:
  - OpenTelemetry dependency with `name == "litellm_request"`.
  - Custom dimensions as listed above for model, tokens, completion, and prompt turns.

## Contact
wuweng@microsoft.com

## Change history
- 9/12/2025 Initial version for Grafana 11.6
