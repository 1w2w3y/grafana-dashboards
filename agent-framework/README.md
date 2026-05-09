# Agent Framework - Agent Overview

![Agent Framework Dashboard](https://github.com/1w2w3y/grafana-dashboards/raw/master/agent-framework/agent-framework-grafana.gif)

Comprehensive monitoring dashboard for **Microsoft Agent Framework** performance, tracking operations, token usage, costs, LLM cache behavior, tool calls, errors, and detailed trace analysis. Monitor your AI agents' health, response times, success rates, model latency, and resource consumption in real-time.

## 📊 What is Microsoft Agent Framework?

The [Microsoft Agent Framework](https://github.com/microsoft/agent-framework) is an open-source development kit for building AI agents and multi-agent workflows for .NET and Python. It brings together the best of [Semantic Kernel](https://github.com/microsoft/semantic-kernel) and [AutoGen](https://github.com/microsoft/autogen), combining their strengths while adding new capabilities for enterprise-grade AI applications.

## 🎯 Why This Dashboard?

As you build AI agents that make autonomous decisions, call external tools, and process user requests, you need visibility into:

- **Performance**: Are your agents responding quickly enough?
- **Reliability**: What's your success rate? Where are errors occurring?
- **Cost**: How much are your AI operations costing in token consumption?
- **LLM behavior**: Are prompts benefiting from cache? Which models are slow?
- **Tool health**: Which tools are called most often, and where are failures or latency concentrated?
- **Usage**: Which agents are most active? When are peak usage times?
- **Debugging**: What's happening inside failed operations?

This dashboard answers all these questions and more—in real-time.

## ✨ Dashboard Highlights

### 📈 Performance Monitoring
- **Response Time Trends**: Track average, 95th, and 99th percentile latencies
- **Success Rate Gauge**: Instant health check with color-coded thresholds
- **Throughput Analysis**: Monitor operations per minute to identify peak loads
- **Agent-Level Metrics**: Compare performance across different agents

### 💰 Token Usage & Cost Analysis
- **Token Consumption Over Time**: Stacked visualization of input/output tokens
- **Cost Estimation**: Daily spending estimates based on model pricing
- **Per-Agent Breakdown**: Identify which agents consume the most resources
- **Trend Analysis**: Spot usage patterns and optimize accordingly

### 🧠 LLM & Tool Insights
- **Cache Hit Rate**: Track the percentage of chat input tokens served from prompt cache
- **Tokens From Cache**: See total cached input tokens and estimate cost savings
- **Chat Finish Reasons**: Review `tool_calls`, `stop`, `length`, and content-filter outcomes
- **Model Latency**: Compare p95 chat latency by model to catch routing or performance drift
- **Tool Usage Leaderboard**: Rank tools by calls, success rate, average duration, and p95 latency

### 🔍 Error Analysis & Debugging
- **Error Rate Monitoring**: Overall system health at a glance
- **Error Timeline**: Identify when problems occur
- **Recent Errors Table**: Quick access to the latest failures
- **Root Cause Analysis**: Drill down with agent name, error type, and duration

### 🔬 Detailed Trace Analysis
- **Interactive Trace Selection**: Click any operation to see full execution details
- **Trace Visualization**: Complete execution flow with timing and dependencies
- **Span Analysis**: Understand the hierarchical relationships in your agent operations
- **Performance Deep-Dive**: Identify bottlenecks in complex agent workflows

### 📊 Agent Activity Insights
- **Agent Performance Summary**: Comprehensive table with operations, duration, and success rates
- **Agent Filter Drilldown**: Click an agent row to filter the dashboard to that agent
- **Utilization Heatmap**: Visual representation of agent activity over time
- **Response Time Percentiles**: Compare latency distribution across agents

## How it works

- Grafana queries Application Insights through the Azure Monitor data source using Log Analytics (KQL) against `dependencies`, `requests`, and `traces`.
- Agent operation panels filter `dependencies` where `name contains "invoke_agent"` and use OpenTelemetry GenAI dimensions such as `gen_ai.agent.name`, `gen_ai.usage.input_tokens`, and `gen_ai.usage.output_tokens`.
- LLM panels use `gen_ai.operation.name == "chat"` with dimensions such as `gen_ai.response.model`, `gen_ai.response.finish_reasons`, and `gen_ai.usage.cache_read.input_tokens`.
- Tool panels use `gen_ai.operation.name == "execute_tool"` and `gen_ai.tool.name`.
- Variables:
  - `am_ds` — Azure Monitor data source
  - `sub` — Azure subscription
  - `rg` — Resource group
  - `res` — Application Insights resource
  - `agentName` — Agent filter
  - `agentTraceId` — Hidden trace selector for drill-down
- Default time range is 7 days; default refresh is 30 minutes.

## Requirements

- Grafana 11.6+ with the Azure Monitor data source configured with access to the subscription containing your Application Insights resource.
- Microsoft Agent Framework telemetry flowing into Application Insights with dependency spans for agent invocation, LLM chat calls, tool execution, and trace correlation.
- GenAI custom dimensions should include agent names, token usage, model names, finish reasons, cached token counts, and tool names for all dashboard panels to populate.

## 🔧 Customization

### Adjust Cost Estimation
The dashboard uses example pricing. Update the cost calculation query to match your model pricing:

```kql
// In the "Daily Cost Estimation" panel query
input_cost = input_tokens * 0.00000025,   // Your input token rate
output_cost = output_tokens * 0.000002    // Your output token rate
```

## 🤝 Contributing

Found a bug or have a feature request? 

- [Open an issue](https://github.com/1w2w3y/grafana-dashboards/issues)
- Submit a pull request
- Share your customizations!

## 📚 Resources

- [Microsoft Agent Framework Documentation](https://learn.microsoft.com/en-us/agent-framework/overview/agent-framework-overview)
- [Agent Framework GitHub Repository](https://github.com/microsoft/agent-framework)
- [Azure Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
- [Grafana Documentation](https://grafana.com/docs/)

## Change history

- 5/8/2026 Add LLM cache, finish reason, model latency, and tool usage panels.
- 10/5/2025 Initial version.

---

**Ready to gain visibility into your AI agents?** Import this dashboard and start monitoring your Agent Framework applications today! 🚀
