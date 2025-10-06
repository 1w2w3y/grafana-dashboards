# Agent Framework - Agent Overview

![Agent Framework Dashboard](https://github.com/1w2w3y/grafana-dashboards/raw/master/agent-framework/agent-framework-grafana.gif)

Comprehensive monitoring dashboard for **Microsoft Agent Framework** performance, tracking operations, token usage, costs, errors, and detailed trace analysis. Monitor your AI agents' health, response times, success rates, and resource consumption in real-time.

## 📊 What is Microsoft Agent Framework?

The [Microsoft Agent Framework](https://github.com/microsoft/agent-framework) is an open-source development kit for building AI agents and multi-agent workflows for .NET and Python. It brings together the best of [Semantic Kernel](https://github.com/microsoft/semantic-kernel) and [AutoGen](https://github.com/microsoft/autogen), combining their strengths while adding new capabilities for enterprise-grade AI applications.

## 🎯 Why This Dashboard?

As you build AI agents that make autonomous decisions, call external tools, and process user requests, you need visibility into:

- **Performance**: Are your agents responding quickly enough?
- **Reliability**: What's your success rate? Where are errors occurring?
- **Cost**: How much are your AI operations costing in token consumption?
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
- **Utilization Heatmap**: Visual representation of agent activity over time
- **Response Time Percentiles**: Compare latency distribution across agents

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

---

**Ready to gain visibility into your AI agents?** Import this dashboard and start monitoring your Agent Framework applications today! 🚀
