# Dashboard Data Ingestion

Several dashboards in this repository — [GitHub Copilot](./github-copilot/README.md), [Claude Code](./claude-code/README.md), [Codex](./codex/README.md), [OpenClaw](./openclaw/README.md), [OpenCode](./opencode/README.md), and [Gemini CLI](./gemini-cli/README.md) — visualize telemetry that flows into **Azure Application Insights** via **OpenTelemetry (OTLP)**.

This guide walks through the end-to-end ingestion pipeline: running an OpenTelemetry Collector with the Azure Monitor Exporter, then pointing each source application at it.

## Architecture

```
┌───────────────┐    OTLP/HTTP    ┌──────────────────┐    Azure Monitor    ┌──────────────────┐              ┌──────────┐
│  Application  │ ──────────────> │  OTel Collector  │ ────── Exporter ──> │    Application   │ <─── KQL ─── │ Grafana  │
│   (source)    │                 │                  │                     │     Insights     │              │ dashboard│
└───────────────┘                 └──────────────────┘                     └──────────────────┘              └──────────┘
```

- Each **source application** (GitHub Copilot / Claude Code / Codex / OpenClaw / OpenCode / Gemini CLI) emits OTLP traces, metrics, and logs to a configured endpoint.
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

### Run with Docker

Save the config above as `otel-collector-config.yaml`, then start the collector with the `contrib` image (which includes the Azure Monitor exporter):

```bash
docker run -d --name otel-collector -p 4318:4318 -p 4317:4317 -v $(pwd)/otel-collector-config.yaml:/etc/otelcol-contrib/config.yaml otel/opentelemetry-collector-contrib:latest
```

Notes:
- Get the `connection_string` from your Application Insights resource (Azure Portal → Application Insights → Overview → **Connection String**).
- The examples below assume the collector is running locally, reachable at `http://localhost:4318`. For a shared/remote collector, substitute your own endpoint.
- The OTLP/HTTP receiver listens on port `4318` by default; all applications documented here use OTLP/HTTP.

## 2. Configure each application

Each application is pointed at the collector's OTLP/HTTP endpoint.

### GitHub Copilot

GitHub Copilot emits OpenTelemetry signals when configured through VS Code settings. See the [official docs](https://code.visualstudio.com/docs/copilot/guides/monitoring-agents).

Add to VS Code `settings.json`:

```json
{
    "github.copilot.chat.otel.enabled": true,
    "github.copilot.chat.otel.exporterType": "otlp-http",
    "github.copilot.chat.otel.otlpEndpoint": "http://localhost:4318",
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
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://localhost:4318",
    "OTEL_LOG_USER_PROMPTS": "1",
    "OTEL_LOG_TOOL_DETAILS": "1",
    "OTEL_METRICS_INCLUDE_VERSION": "true"
  }
}
```

Tip: `OTEL_LOG_USER_PROMPTS` and `OTEL_LOG_TOOL_DETAILS` enrich the dashboard's per-user and tool-usage panels. Omit them if you prefer not to capture prompt text.

### Codex

Codex CLI telemetry should be exported to the collector using OTLP/HTTP and a stable service name.

Use `codex` as the service name and `http://localhost:4318` as the OTLP endpoint. The Codex dashboard reads metrics from the `customMetrics` table and filters by `cloud_RoleName startswith "codex"`.

Important: Codex metric names must follow the `codex.*` naming scheme, including metrics such as `codex.thread.started`, `codex.conversation.turn.count`, `codex.turn.token_usage`, `codex.tool.call`, and `codex.approval.requested`.

### OpenClaw

OpenClaw gateway publishes OpenTelemetry signals via its logging/telemetry config. See the [official docs](https://docs.openclaw.ai/logging#export-to-opentelemetry).

Add to the gateway's telemetry config:

```json
{
  "enabled": true,
  "endpoint": "http://localhost:4318",
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

### OpenCode

OpenCode telemetry should be exported to the collector using OTLP/HTTP and a stable service name.

Use `opencode` as the service name:

```json
{
  "endpoint": "http://localhost:4318",
  "protocol": "http/protobuf",
  "serviceName": "opencode"
}
```

Important: `serviceName` must be `opencode`. The OpenCode dashboard filters by `cloud_RoleName == "opencode"` and groups data by `opencode.*` and AI SDK custom dimensions.

### Gemini CLI

Gemini CLI telemetry should be configured to export OTLP signals to the collector.

Use `gemini-cli` as the service name and `http://localhost:4318` as the OTLP endpoint. The exact configuration surface depends on the Gemini CLI version and telemetry setup, but the resulting telemetry must include Gemini CLI event dimensions such as `gemini_cli.api_response`, `gemini_cli.api_error`, `gemini_cli.tool_call`, and `gemini_cli.user_prompt`.

Important: `serviceName` must be `gemini-cli`. The Gemini CLI dashboard filters by `cloud_RoleName == "gemini-cli"` and groups data by `gemini_cli.*` custom dimensions.

## 3. Verify data in Application Insights

Once the applications and the collector are running, confirm telemetry is arriving:

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
// Codex
customMetrics
| where timestamp > ago(1h)
| where cloud_RoleName startswith "codex"
| where name startswith "codex."
| take 50
```

```kusto
// OpenClaw
dependencies
| where timestamp > ago(1h)
| where cloud_RoleName == "openclaw-gateway"
| take 50
```

```kusto
// OpenCode
dependencies
| where timestamp > ago(1h)
| where cloud_RoleName == "opencode"
| take 50
```

```kusto
// Gemini CLI
traces
| where timestamp > ago(1h)
| where cloud_RoleName == "gemini-cli"
| where tostring(customDimensions["event.name"]) startswith "gemini_cli."
| take 50
```

If rows come back, the pipeline is working. If not, check the collector logs for export errors (typical culprits: wrong connection string, blocked egress, or a firewalled OTLP endpoint).

## 4. Import the dashboards into Grafana

Each dashboard README has its own import and variables reference:

- [Claude Code](./claude-code/README.md)
- [Codex](./codex/README.md)
- [GitHub Copilot](./github-copilot/README.md)
- [OpenClaw](./openclaw/README.md)
- [OpenCode](./opencode/README.md)
- [Gemini CLI](./gemini-cli/README.md)

All require **Grafana 11.6+** with an **Azure Monitor data source** that has access to the subscription containing your Application Insights resource.

## References

- [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/)
- [Azure Monitor Exporter for the OpenTelemetry Collector](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/azuremonitorexporter)
- [Application Insights connection strings](https://learn.microsoft.com/en-us/azure/azure-monitor/app/sdk-connection-string)
- [Monitoring GitHub Copilot agents](https://code.visualstudio.com/docs/copilot/guides/monitoring-agents)
- [Monitoring Claude Code usage](https://code.claude.com/docs/en/monitoring-usage)
- [Codex](https://github.com/openai/codex)
- [OpenClaw — Export to OpenTelemetry](https://docs.openclaw.ai/logging#export-to-opentelemetry)
- [OpenCode](https://opencode.ai)
- [Gemini CLI](https://github.com/google-gemini/gemini-cli)
