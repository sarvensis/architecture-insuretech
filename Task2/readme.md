## Динамическая маршрутизация на основании показателей утилизации памяти

Алгоритм

1. Поднять k8s (я поднимаю на OrbStack)
2. Активировать metrics-server

```
kubectl apply -f metrics-server.yaml
```

Изменения для запуска в orbstack применены

3. Деплой приложения, сервиса и hpa

```
kubectl apply -f deployment.yaml
kubectl apply -f deployment.yaml


4. Установить Prometheus

```
helm -n orbstack repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm -n orbstack repo update

helm -n orbstack install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```

5. В выводе взять пароль от admin в графане, имя пода, подключиться к портам и зайти через интерфейс
```
NAME: prometheus
LAST DEPLOYED: Sun Jun 29 10:29:22 2025
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace monitoring get pods -l "release=prometheus"

Get Grafana 'admin' user password by running:

  kubectl --namespace monitoring get secrets prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

Access Grafana local instance:

  export POD_NAME=$(kubectl --namespace monitoring get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=prometheus" -oname)
  kubectl --namespace monitoring port-forward $POD_NAME 3000
```

6. Установка Prometheus Адаптера

```
helm -n orbstack install prometheus-adapter prometheus-community/prometheus-adapter \
  --namespace monitoring \
  --set prometheus.url=http://prometheus-kube-prometheus-prometheus.monitoring.svc \
  --set rules.default=true

<!-- kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1 -->
```

7. Применить service-monitor.yaml

Зайти в prometheus ui или grafana
```
# prometheus
kubectl port-forward svc/prometheus-kube-prometheus-prometheus -n monitoring 9090:9090
```
