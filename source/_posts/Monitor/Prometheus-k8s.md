## 官方部署

由于我司服务器环境需要替换镜像，这里列出了需要调整的镜像和文件位置，方便参考

quay.io/prometheus/alertmanager
quay.io/prometheus/node-exporter:v0.18.1
quay.io/prometheus/prometheus
grafana/grafana:6.4.3
quay.io/coreos/kube-rbac-proxy:v0.4.1
quay.io/coreos/kube-state-metrics:v1.8.0
quay.io/coreos/k8s-prometheus-adapter-amd64:v0.5.0
quay.io/coreos/prometheus-operator:v0.34.0
quay.io/coreos/prometheus-config-reloader:v0.34.0
quay.io/coreos/configmap-reload:v0.0.1

my-register/quay.io/prometheus/alertmanager  `alertmanager-alertmanager.yaml`

my-register/grafana/grafana:6.4.3  `grafana-deployment.yaml`

my-register/quay.io/coreos/kube-rbac-proxy:v0.4.1  `kube-state-metrics-deployment.yaml`
my-register/quay.io/coreos/kube-state-metrics:v1.8.0

my-register/quay.io/prometheus/node-exporter:v0.18.1  `node-exporter-daemonset.yaml`
my-register/quay.io/coreos/kube-rbac-proxy:v0.4.1

my-register/quay.io/coreos/k8s-prometheus-adapter-amd64:v0.5.0  `prometheus-adapter-deployment.yaml`

my-register/quay.io/prometheus/prometheus  `prometheus-prometheus.yaml`
 
my-register/quay.io/coreos/prometheus-operator:v0.34.0  `prometheus-operator-deployment.yaml`
my-register/quay.io/coreos/prometheus-config-reloader:v0.34.0
my-register/quay.io/coreos/configmap-reload:v0.0.1

## 流程总结

部署 promethues，grafana，alert manager

关于数据存储

- "--storage.tsdb.path=/prometheus"
- "--storage.tsdb.retention=24h"

## 关于版本

我们通过github搜索 prometheus，发现有好多的版本

详细大家比较熟知的还是Prometheus仓库下的版本，当时去网上一搜，找到的也是用这个仓库的来部署的，但是后来发现，这个部署的有问题，虽然说监控k8s，但是你会发现很多指标都没有，基本就是一些cpu和内存的指标，如果你像知道容器状态等指标都是没有的

一开始我一度以为是版本太低，后来我升级的版本后，发现指标是多了，但是依然没有关键的数据

直到后来我发现，官方还有一个prometheus-operator的仓库，这名字一看，不就是为了监控kubernetes

仓库分为两个版本 Prometheus Operator和kube-prometheus

这是原文比较

```
Prometheus Operator vs. kube-prometheus vs. community helm chart
The Prometheus Operator uses Kubernetes custom resources to simplifiy the deployment and configuration of Prometheus, Alertmanager, and related monitoring components.

kube-prometheus provides example configurations for a complete cluster monitoring stack based on Prometheus and the Prometheus Operator. This includes deployment of multiple Prometheus and Alertmanager instances, metrics exporters such as the node_exporter for gathering node metrics, scrape target configuration linking Prometheus to various metrics endpoints, and example alerting rules for notification of potential issues in the cluster.

The stable/prometheus-operator helm chart provides a similar feature set to kube-prometheus. This chart is maintained by the Helm community. For more information, please see the chart's readme
```

那么我们应该采用kube-prometheus来部署