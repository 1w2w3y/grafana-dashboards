# Dashboard Data Ingestion

Several dashboards in this repository — [GitHub Copilot](./github-copilot/README.md), [Claude Code](./claude-code/README.md), and [OpenClaw](./openclaw/README.md) — visualize telemetry that flows into **Azure Application Insights** via **OpenTelemetry (OTLP)**.

This guide walks through the end-to-end ingestion pipeline: running an OpenTelemetry Collector with the Azure Monitor Exporter, then pointing each source application at it.

## Architecture

```
┌───────────────┐    OTLP/HTTP    ┌──────────────────┐    Azure Monitor    ┌──────────────────┐              ┌──────────┐
│  Application  │ ──────────────> │  OTel Collector  │ ────── Exporter ──> │    Application   │ <─── KQL ─── │ Grafana  │
│   (source)    │                 │                  │                     │     Insights     │              │ dashboard│
└───────────────┘                 └──────────────────┘                     └──────────────────┘              └──────────┘
```

- Each **source application** (GitHub Copilot / Claude Code / OpenClaw) emits OTLP traces, metrics, and logs to a configured endpoint.
- An **OpenTelemetry Collector** terminates OTLP at that endpoint and forwards the data to Application Insights using the Azure Monitor Exporter.
- **Grafana** queries Application Insights via the Azure Monitor data source (Log Analytics / KQL) to render the dashboards.

## 1. Run the OpenTelemetry Collector

Deploy an [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/) (the `contrib` distribution) configured with the [Azure Monitor Exporter](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/azuremonitorexporter). The collector is what bridges OTLP to the Application Insights ingestion API.

### Sample collector config

```yaml
receivers:
  otlp:
    protocols:
      http:
        endpoint: 0.0.0.0:4318
      grpc:
        endpoint: 0.0.0.0:4317

exporters:
  azuremonitor:
    connection_string: "InstrumentationKey=<YOUR-KEY>;IngestionEndpoint=https://<region>.in.applicationinsights.azure.com/;LiveEndpoint=https://<region>.livediagnostics.monitor.azure.com/"

service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [azuremonitor]
    metrics:
      receivers: [otlp]
      exporters: [azuremonitor]
    logs:
      receivers: [otlp]
      exporters: [azuremonitor]
```

Notes:
- Get the `connection_string` from your Application Insights resource (Azure Portal → Application Insights → Overview → **Connection String**).
- Expose the collector at a stable, reachable endpoint — e.g. `https://otel.<region>.<your-domain>`. The examples below assume `https://otel.wus2.sample-dev.azgrafana-test.io`.
- The OTLP/HTTP receiver listens on port `4318` by default; all three applications documented here use OTLP/HTTP.

## 2. Configure each application

Each application is pointed at the collector's OTLP/HTTP endpoint.

### GitHub Copilot

GitHub Copilot emits OpenTelemetry signals when configured through VS Code settings. See the [official docs](https://code.visualstudio.com/docs/copilot/guides/monitoring-agents).

Add to VS Code `settings.json`:

```json
{
    "editor.renderWhitespace": "all",
    "editor.formatOnSave": true,
    "chatgpt.config": {
        "preferred_auth_method": "apikey"
    },
    "chat.agent.maxRequests": 250,
    "github.copilot.chat.otel.enabled": false,
    "github.copilot.chat.otel.exporterType": "otlp-http",
    "github.copilot.chat.otel.otlpEndpoint": "https://otel.wus2.sample-dev.azgrafana-test.io",
    "github.copilot.chat.otel.captureContent": true
}
```

Note: `github.copilot.chat.otel.enabled` is `false` in the sample because Copilot OTel export is gated at the machine level. To turn it on, set the machine environment variable `COPILOT_OTEL_ENABLED=true` rather than flipping the setting.

### Claude Code

Claude Code reads telemetry configuration from environment variables. See the [official docs](https://code.claude.com/docs/en/monitoring-usage).

Add to the Claude Code `settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp",
    "OTEL_EXPORTER_OTLP_PROTOCOL": "http/protobuf",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "https://otel.wus2.sample-dev.azgrafana-test.io",
    "OTEL_LOG_USER_PROMPTS": "1",
    "OTEL_LOG_TOOL_DETAILS": "1",
    "OTEL_METRICS_INCLUDE_VERSION": "true"
  }
}
```

Tip: `OTEL_LOG_USER_PROMPTS` and `OTEL_LOG_TOOL_DETAILS` enrich the dashboard's per-user and tool-usage panels. Omit them if you prefer not to capture prompt text.

### OpenClaw

OpenClaw gateway publishes OpenTelemetry signals via its logging/telemetry config. See the [official docs](https://docs.openclaw.ai/logging#export-to-opentelemetry).

Add to the gateway's telemetry config:

```json
{
  "enabled": true,
  "endpoint": "https://otel.wus2.sample-dev.azgrafana-test.io",
  "protocol": "http/protobuf",
  "serviceName": "openclaw-gateway",
  "traces": true,
  "metrics": true,
  "logs": true,
  "sampleRate": 1,
  "flushIntervalMs": 5000
}
```

Important: `serviceName` must be `openclaw-gateway`. The OpenClaw dashboard filters by `cloud_RoleName == "openclaw-gateway"`, which is derived from this field.

## 3. Verify data in Application Insights

Once both the applications and the collector are running, confirm telemetry is arriving:

1. Azure Portal → your Application Insights resource → **Logs**
2. Run a KQL check for each source:

```kusto
// GitHub Copilot
dependencies
| where timestamp > ago(1h)
| where cloud_RoleName == "copilot-chat"
| take 50
```

```kusto
// Claude Code
customMetrics
| where timestamp > ago(1h)
| where name startswith "claude_code"
| take 50
```

```kusto
// OpenClaw
dependencies
| where timestamp > ago(1h)
| where cloud_RoleName == "openclaw-gateway"
| take 50
```

If rows come back, the pipeline is working. If not, check the collector logs for export errors (typical culprits: wrong connection string, blocked egress, or a firewalled OTLP endpoint).

## 4. Import the dashboards into Grafana

Each dashboard README has its own import and variables reference:

- [Claude Code](./claude-code/README.md)
- [GitHub Copilot](./github-copilot/README.md)
- [OpenClaw](./openclaw/README.md)

All three require **Grafana 11.6+** with an **Azure Monitor data source** that has access to the subscription containing your Application Insights resource.

## References

- [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/)
- [Azure Monitor Exporter for the OpenTelemetry Collector](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/azuremonitorexporter)
- [Application Insights connection strings](https://learn.microsoft.com/en-us/azure/azure-monitor/app/sdk-connection-string)
- [Monitoring GitHub Copilot agents](https://code.visualstudio.com/docs/copilot/guides/monitoring-agents)
- [Monitoring Claude Code usage](https://code.claude.com/docs/en/monitoring-usage)
- [OpenClaw — Export to OpenTelemetry](https://docs.openclaw.ai/logging#export-to-opentelemetry)
