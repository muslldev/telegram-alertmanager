# ![logo](https://skillicons.dev/icons?i=prometheus) Alerts with Prometheus and Alertmanager

## 1. Let's describe the alert rules

```
groups:
  - name: node-alerts
    rules:
      - alert: HostHighCPU
        expr: 100 - (
          avg(
          rate(
          node_cpu_seconds_total{mode="idle"}[5m]
          )
          ) * 100
          ) > 80
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage"
          description: "CPU above 80% for more than 2 minutes"

  - name: node-instance
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Server {{ $labels.instance }} is unavailable."
          description: "Server has been unavailable for more than 1 minute."
```

## 2. Specify Telegram in the Alertmanager config for displaying notifications.

```
route:
  receiver: telegram

receivers:
  - name: telegram
    webhook_configs:
      - url: http://prom-telegrambot:8080
```

## 3. Enable alert rules, Alertmanager, and Node Exporter in the Prometheus config.

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "node"
    static_configs:
      - targets: ["prom-node-exp:9100"]

rule_files:
  - alerts.yml

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - prom-alertmanager:9093
```

## 4. Set up our monitoring stack

```
docker compose up -d
```

## 5. Ensure Prometheus receives alert rules

![prometheus_alerts](assets/prometheus_alerts.png)

## 6. Initiate a server downtime scenario and ensure Alertmanager processes it

![alertmanager_alerts](assets/alertmanager_alerts.png)

## 7. Ensure Telegram sends notifications

![telegram_notifications](assets/telegram_notifications.png)
