# Grafana dashboards for AI Foundry

AI Foundry model deployments will emit useful Azure Monitor metrics. You can leverage this AI Foundry Grafana dashboard to view cost, usage and performance of your model deployments.

![AI Foundry dashboard](https://raw.githubusercontent.com/1w2w3y/grafana-dashboards/master/ai-foundry/ai-foundry-top-section.png)

## Access the dashboard in Azure Portal

- [AI Foundry dashboard](https://aka.ms/amg/dash/ai-foundry)

# Grafana dashboards for Agent Framework

Azure Monitor Dashboards with Grafana provides prebuilt monitoring visualizations for Agent Framework. These dashboards are available directly within the Azure portal, with no extra setup or costs.


## Available dashboards

Agent Framework offers two prebuilt Grafana dashboards:

| Dashboard | Description |
|---|---|
| **Agent Overview dashboard** | Comprehensive monitoring dashboard for AI agent framework performance, tracking operations, token usage, costs, errors, and detailed trace analysis. Monitor agent health, response times, success rates, and resource consumption in real-time. |
| **Workflow Overview dashboard** | Comprehensive monitoring dashboard for Agent Framework workflows, tracking execution metrics, performance analysis, and trace visualization. |

![Agent Overview dashboard](https://github.com/Azure/azure-managed-grafana/raw/main/samples/assets/grafana-af-agent.gif)
![Workflow Overview dashboard](https://github.com/Azure/azure-managed-grafana/raw/main/samples/assets/grafana-af-workflow.gif)

## Instrument agent framework with Application Insights - python
Details here:
https://github.com/microsoft/agent-framework/blob/main/python/samples/getting_started/observability/README.md

### Setting up exporters and providers using `setup_observability()`

To make it easier for developers to get started, the `agent_framework.observability` module provides a `setup_observability()` function that will setup exporters and providers for traces, logs, and metrics based on environment variables. You can call this function at the start of your application to enable telemetry.

```python
from agent_framework.observability import setup_observability

setup_observability()
```

### Environment variables for `setup_observability()`

The `setup_observability()` function will look for the following environment variables to determine how to setup the exporters and providers:

- APPLICATIONINSIGHTS_CONNECTION_STRING="..."

## Instrument agent framework with Application Insights - .NET
Details here:
https://github.com/microsoft/agent-framework/blob/main/dotnet/samples/GettingStarted/Workflows/Observability/ApplicationInsights/Program.cs

## Access the dashboards in Azure Portal

- [Agent Framework - Agent Overview dashboard](https://aka.ms/amg/dash/af-agent)
- [Agent Framework - Workflow Overview dashboard](https://aka.ms/amg/dash/af-workflow)