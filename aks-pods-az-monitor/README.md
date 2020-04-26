# AKS Container Insights
Global view for listing all the pods and the containers from **multiple AKS clusters**.

## Screenshots
![aks-az-mon](./aks-az-mon.PNG)

## Features
* List all the pods and their important health status.
* List all the containers, container image version, container health.

## How it works?
* Each AKS cluster need to [enable the Azure Monitor Container Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/container-insights-enable-existing-clusters).
* Azure Monitor will periodically gather the pods and containers status from the Kubernetes API server.
* The telemetry data collected from multiple AKS clusters will be sent to one [Log Analytics workspace](https://docs.microsoft.com/en-us/azure/azure-monitor/learn/quick-create-workspace)
* Grafana will query the Log Analytics workspace for the pods and containers status.