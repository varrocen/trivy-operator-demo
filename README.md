# trivy-operator-demo

## Cleanup

Commands:

```
orb delete k8s -f
```

## Warmup

Commands:

```
docker pull python:3.4-alpine
trivy image --download-db-only
orb start k8s
helm repo add aqua https://aquasecurity.github.io/helm-charts/
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install trivy-operator aqua/trivy-operator \
  --namespace trivy-system \
  --create-namespace \
  --version 0.20.0 \
  --values trivy-values.yaml
```

## Demo 01

Commands:

```
trivy image --scanners vuln,misconfig,secret,license python:3.4-alpine
trivy k8s --report=summary cluster
```

## Demo 02

Commands:

```
kubectl create deployment nginx --image nginx:1.16
```

Wait.

```
kubectl get vulnerabilityreports -o wide
kubectl get configauditreports -o wide
kubectl tree deploy nginx
```

```
kubectl set image deployment nginx nginx=nginx:1.17
```

Wait.

```
kubectl tree deploy nginx
```

## Demo 03

Commands:

```
helm upgrade --install prom prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --values prom-values.yaml

helm install trivy-operator aqua/trivy-operator \
  --namespace trivy-system \
  --create-namespace \
  --version 0.20.0 \
  --values trivy-values.yaml

kubectl create deployment nginx --image nginx:1.16
```

URLs:

* http://prom-kube-prometheus-stack-prometheus.monitoring.svc.cluster.local:9090/
* http://prom-grafana.monitoring.svc.cluster.local/
* https://grafana.com/grafana/dashboards/17813-trivy-operator-dashboard/

Prom queries:

* sum(trivy_image_vulnerabilities)
* sum(trivy_resource_configaudits)
* sum(trivy_image_exposedsecrets)

Get trivy operator pod IP:

```
k -n trivy-system get pod -o wide
```
