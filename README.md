# trivy-operator-demo

## Prerequites

* [Docker](https://www.docker.com/)
* [Helm](https://helm.sh/fr/)
* [OrbStack](https://orbstack.dev/)
* [Trivy](https://trivy.dev/)
* [watch](https://gitlab.com/procps-ng/procps)

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
helm upgrade --install prom oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --values prometheus-values.yaml
helm repo add aqua https://aquasecurity.github.io/helm-charts/
helm repo update
helm install trivy-operator aqua/trivy-operator \
  --namespace trivy-system \
  --create-namespace \
  --version 0.29.0 \
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
watch kubectl tree deploy nginx
kubectl get vulnerabilityreports -o wide
kubectl get configauditreports -o wide
kubectl get exposedsecretreports -o wide
kubectl get sbomreports -o wide
```

```
kubectl set image deployment nginx nginx=nginx:1.17
```

Wait.

```
watch kubectl tree deploy nginx
```

## Demo 03

Commands:

```
kubectl delete deployment nginx
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
