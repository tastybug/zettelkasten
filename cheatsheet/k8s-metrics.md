# K8S Metrics

Normalized CPU saturation (percent)

```
sum (rate(container_cpu_usage_seconds_total{namespace="prod-seller-app",image!="", pod=~"seller-listing-e.*", container="seller-listing-enrichment"}[1m])) by (pod) / max(kube_pod_container_resource_limits_cpu_cores{pod=~"seller-listing-e.*", namespace="prod-seller-app", container="seller-listing-enrichment"}) by (pod)
```
