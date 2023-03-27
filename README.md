# Monitoring Stack using Helm Operators
## Prometheus, Alert-manager, Node-Exporter, Grafana, Promtail, Loki, and Komodor included

# Installation

Create a new namespaces
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

Install Promtail
```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm upgrade --install promtail grafana/promtail -f monitoring/promtail-values.yaml -n monitoring
kubbectl get pods  -n monitoring
```

Install Loki
```
helm upgrade --install loki grafana/loki-distributed -n monitoring
kubectl get pods  -n monitoring
```

Install Komodor
```
helm repo add komodorio https://helm-charts.komodor.io
helm repo update
```
* Register with komodor.com and recieve your cluster integration bash script. it will automaticall deploy komodor in our cluster

Install Demo application resources
```
kubectl apply -n demo -f ./application
kubectl apply -n demo -f pinger/.
```

Test pinger app latency by komodor
```
k -n demo set image deployment/app app=anaisulrichs/ping-pong:slow
```
