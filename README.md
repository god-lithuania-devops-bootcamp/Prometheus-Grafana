# Monitoring Stack using Helm Operators
## Prometheus, Alert-manager, Node-Exporter, Grafana, Promtail, Loki, and Komodor included

# Installation

Create a new namespace
```
kubectl create ns demo
kubectl create ns monitoring
```
Install monitoring stack

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update 
helm install prom prometheus-community/kube-prometheus-stack -n monitoring --values monitoring/values.yaml
```
Install NodePort Services for external access

```
kubectl apply -f monitoring/nodeport.yaml
```

Install Demo application resources
```
kubectl apply -n demo -f ./application
```
