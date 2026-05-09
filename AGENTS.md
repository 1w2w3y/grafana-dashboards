# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Repository Purpose

Curated collection of ready-to-use Grafana dashboards for Azure services. Each dashboard is a standalone JSON file importable into Grafana, published to the Grafana.com gallery. There is no build system, test suite, or linting — the artifacts are the JSON dashboards themselves.

## Repository Structure

Each dashboard lives in its own directory with a consistent layout:
- `<dashboard-dir>/dashboards/<name>.json` — the importable Grafana dashboard JSON
- `<dashboard-dir>/README.md` — setup instructions, features, metrics reference, and change history
- `<dashboard-dir>/*.png` or `*.gif` — preview screenshots

Current dashboards:
- **ai-foundry/** — Azure AI Foundry (Cognitive Services) usage, latency, and cost estimation
- **agent-framework/** — Agent Framework agent overview
- **af-workflow/** — Agent Framework workflow overview
- **aks-pods-az-monitor/** — AKS Container Insights via Azure Monitor
- **litellm-azmon/** — LiteLLM usage and latency via Azure Monitor
- **litellm-trace/** — LiteLLM trace details

The `doc/` directory contains supplementary documentation on GenAI monitoring concepts.

## Dashboard JSON Conventions

- Dashboards use the Grafana Azure Monitor data source (`grafana-azure-monitor-datasource`). The datasource input is typically named `am_ds` with label `Azure Monitor`.
- Dashboard variables provide flexible scoping (subscription, resource ID) with hidden helper variables auto-resolved from the selected resource.
- Metrics are grouped by dimensions like `ModelDeploymentName` for per-deployment breakdowns.
- The `__inputs`, `__elements`, and `__requires` top-level keys define datasource bindings and Grafana version requirements.

## Working with Dashboards

- To add a new dashboard: create a new directory, add the JSON under `dashboards/`, add a README with setup/features/change history, add preview images, and update the root `README.md`.
- When modifying a dashboard JSON, preserve the `__inputs`/`__requires` structure so the import wizard works correctly.
- Update the change history section in the dashboard's README when making modifications.
- The root `README.md` lists all dashboards with gallery IDs, links, and preview images.
