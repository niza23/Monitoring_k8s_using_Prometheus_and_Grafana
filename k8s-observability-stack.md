
# 🚀 Kubernetes Monitoring & Observability Stack

![Kubernetes](https://img.shields.io/badge/Kubernetes-Observability-blue?logo=kubernetes&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-orange?logo=prometheus)
![Grafana](https://img.shields.io/badge/Grafana-Dashboards-yellow?logo=grafana)
![Loki](https://img.shields.io/badge/Loki-Logs-green?logo=grafana)
![Thanos](https://img.shields.io/badge/Thanos-Long--term%20Metrics-purple)
![Alertmanager](https://img.shields.io/badge/Alertmanager-Alerts-red)
![Instana](https://img.shields.io/badge/Instana-APM%20%26%20Tracing-lightgrey)

---

## 📚 Table of Contents
1. [Overview](#-overview)
2. [Stack Components](#-stack-components)
3. [Architecture](#-architecture)
4. [Directory Structure](#-directory-structure)
5. [Configuration Examples](#-configuration-examples)
   - [Prometheus](#prometheus)
   - [Alertmanager](#alertmanager)
   - [Grafana](#grafana)
   - [Loki](#loki)
   - [Thanos](#thanos)
6. [Best Practices](#-best-practices)
7. [Interview Essentials](#-interview-essentials)
8. [References](#-references)

---

## 🔍 Overview

This repository provides a **production-ready Kubernetes Observability Stack** combining:

- **Prometheus** → Metrics Collection  
- **Grafana** → Visualization  
- **Loki** → Log Aggregation  
- **Alertmanager** → Alert Routing  
- **Thanos** → Long-term Metric Storage  
- **Instana** → Tracing & APM  

---

## 🧠 Stack Components

| Layer | Tool | Description |
|-------|------|--------------|
| Metrics | **Prometheus** | Scrapes & stores metrics |
| Alerts | **Alertmanager** | Routes alerts (Slack, PagerDuty, Email) |
| Visualization | **Grafana** | Dashboards & alerting |
| Logs | **Loki** | Centralized log collection |
| Exporters | **Node Exporter**, **kube-state-metrics**, **Pushgateway** | Metric sources |
| Tracing | **Instana** | Application tracing & profiling |
| Storage | **Thanos** | Long-term metrics storage |

---

## 🏗️ Architecture

```text
[ Node Exporter ]   \
[ kube-state-metrics ] → Prometheus → Thanos (Object Storage)
[ Pushgateway ]     /
          ↓
     Alertmanager → Slack / PagerDuty
          ↓
       Grafana → Dashboards (Metrics + Logs + Traces)
          ↓
          Loki → Centralized Logs
          ↓
        Instana → Application Performance Monitoring
````

---

## 📂 Directory Structure

```bash
k8s-observability-stack/
├── prometheus/
│   ├── prometheus-deploy.yaml
│   ├── prometheus-config.yaml
│   └── rules/
│       └── alerts.yaml
├── alertmanager/
│   ├── alertmanager-config.yaml
│   └── templates/
├── grafana/
│   ├── dashboards/
│   │   ├── cluster-overview.json
│   │   ├── node-metrics.json
│   │   └── app-performance.json
│   └── grafana-deploy.yaml
├── loki/
│   ├── loki-config.yaml
│   └── promtail-config.yaml
├── thanos/
│   ├── thanos-sidecar.yaml
│   ├── thanos-store.yaml
│   └── thanos-query.yaml
└── README.md
```

---

## ⚙️ Configuration Examples

### 🟠 Prometheus

**`prometheus/prometheus-config.yaml`**

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)

  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod

  - job_name: 'pushgateway'
    static_configs:
      - targets: ['pushgateway:9091']

rule_files:
  - "/etc/prometheus/rules/*.yaml"
```

---

### 🔔 Alertmanager

**`alertmanager/alertmanager-config.yaml`**

```yaml
global:
  resolve_timeout: 5m

route:
  receiver: 'slack-notifications'
  group_by: ['alertname', 'severity']

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - send_resolved: true
        channel: '#alerts'
        username: 'Alertmanager'
        api_url: 'https://hooks.slack.com/services/XXXX/XXXX/XXXX'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname']
```

---

### 📊 Grafana

**`grafana/grafana-deploy.yaml`**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-datasources
data:
  datasources.yaml: |
    apiVersion: 1
    datasources:
      - name: Prometheus
        type: prometheus
        url: http://prometheus:9090
        access: proxy
      - name: Loki
        type: loki
        url: http://loki:3100
```

**Example Dashboard:**
[`grafana/dashboards/cluster-overview.json`](./grafana/dashboards/cluster-overview.json)

---

### 🪵 Loki

**`loki/loki-config.yaml`**

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory
  chunk_idle_period: 5m

schema_config:
  configs:
    - from: 2022-01-01
      store: boltdb-shipper
      object_store: s3
      bucket_names: my-loki-logs
      schema: v11
      index:
        prefix: index_
        period: 24h
```

---

### 🟣 Thanos

**`thanos/thanos-sidecar.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: thanos-sidecar
spec:
  template:
    spec:
      containers:
      - name: thanos-sidecar
        image: quay.io/thanos/thanos:v0.33.0
        args:
          - sidecar
          - --tsdb.path=/prometheus
          - --objstore.config-file=/etc/thanos/object-store.yaml
          - --prometheus.url=http://prometheus:9090
```

**`thanos/object-store.yaml`**

```yaml
type: S3
config:
  bucket: "thanos-metrics"
  endpoint: "s3.amazonaws.com"
  access_key: "${AWS_ACCESS_KEY_ID}"
  secret_key: "${AWS_SECRET_ACCESS_KEY}"
  insecure: false
```

---

## 🧩 Best Practices

| Component        | Recommendation                                                          |
| ---------------- | ----------------------------------------------------------------------- |
| **Prometheus**   | Use HA pairs; keep retention short; use Thanos for long-term            |
| **Alertmanager** | Group alerts, deduplicate, use Slack/PagerDuty integrations             |
| **Grafana**      | Version dashboards with GitOps; enable SSO; combine metrics/logs/traces |
| **Loki**         | Store logs in S3/GCS; avoid high-cardinality labels                     |
| **Thanos**       | Enable compactor and store-gateway for scalability                      |
| **Instana**      | Trace microservices; correlate with Prometheus metrics                  |

---

## 🎯 Interview Essentials

**Golden Signals:** Latency | Traffic | Errors | Saturation
**Rule of Thumb:**
➡️ Alerts → Grafana → Loki logs → Root cause in Instana

---

## 🔗 References

* [Prometheus Docs](https://prometheus.io/docs/)
* [Grafana Labs](https://grafana.com/)
* [Loki Documentation](https://grafana.com/docs/loki/latest/)
* [Thanos Project](https://thanos.io/)
* [Alertmanager Docs](https://prometheus.io/docs/alerting/latest/alertmanager/)
* [Instana APM](https://www.ibm.com/instana/)

---

