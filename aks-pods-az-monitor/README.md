# AKS Container Insights
Global view for listing all the pods and the containers from **multiple AKS clusters**.

## Screenshots
![aks-az-mon-2509](https://github.com/1w2w3y/grafana-dashboards/raw/master/aks-pods-az-monitor/aks-az-mon-2509.png)
![aks-az-mon](https://github.com/1w2w3y/grafana-dashboards/raw/master/aks-pods-az-monitor/aks-az-mon.PNG)

## Features
* List all the pods and their important health status.
* List all the containers, container image version, container health.

## How it works?
* Each AKS cluster need to [enable the Azure Monitor Container Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/container-insights-enable-existing-clusters).
* Azure Monitor will periodically gather the pods and containers status from the Kubernetes API server.
* The telemetry data collected from multiple AKS clusters will be sent to one [Log Analytics workspace](https://docs.microsoft.com/en-us/azure/azure-monitor/learn/quick-create-workspace)
* Grafana will query the Log Analytics workspace for the pods and containers status.

## Contact
wuweng@microsoft.com

## Change history
* 9/6/2025 Updated dashboard for Grafana 11.6
* 9/22/2020 Fixed the broken Kusto query for listing the clusters
* 7/9/2020 Fixed the broken container panel and updated the Dashboard to based on Grafana 7

## Gallery link
https://grafana.com/grafana/dashboards/12180-aks-container-insights/