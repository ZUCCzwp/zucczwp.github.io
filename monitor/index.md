# 容器监控


## 基本监控
### 单个容器
```
docker ps -a
docker top container_name
```
### weavescope图形化界面
[scope](https://github.com/weaveworks/scope)
### 集群
grafana

## 根据资源占用自动横向伸缩
```
kubectl run php-apache --image=us.gcr.io/k8s-artifacts-prod/hpa-example --requests=cpu=200m --expose --port=80
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
kubectl get horizontalpodautoscaler
```

## 日志
ELK

## Prometheus
[Prometheus](https://github.com/prometheus-operator/prometheus-operator)
