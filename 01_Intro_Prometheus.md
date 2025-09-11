
# 📊 Prometheus in Amazon EKS

## 🔹 What is Prometheus?
Prometheus is an **open-source monitoring and alerting toolkit** designed for modern, cloud-native applications.  
It collects and stores **time-series metrics** (CPU usage, memory, requests, errors, etc.) and allows you to query them for analysis or alerts.

---

## 🔹 Why Prometheus in EKS?
When you run workloads in **Amazon EKS (Elastic Kubernetes Service)**, you need to keep track of:
- How many pods are running?  
- Are nodes under heavy CPU or memory load?  
- Are services responding quickly?  
- Is the cluster healthy?  

Prometheus helps answer these questions by:
- **Scraping metrics** from Kubernetes components (API Server, Kubelets, etc.) and application pods.  
- **Storing metrics** in a time-series database.  
- **Triggering alerts** when something goes wrong.  
- Acting as a **data source** for visualization tools like Grafana.  

---

## 🔹 Prometheus Workflow in EKS

```text
        +------------------------+
        |   Kubernetes Cluster   |
        |  (Nodes + Pods + Apps) |
        +-----------+------------+
                    |
                    | Metrics (via exporters, kubelet, API server)
                    v
        +------------------------+
        |      Prometheus        |
        |  Collects & Stores TS  |
        +-----------+------------+
                    |
            +-------+-------+
            |               |
        Queries         Alerts
            |               |
            v               v
  +----------------+   +-----------------+
  | Grafana (Dash) |   | Alertmanager    |
  | Visualization  |   | Notifications   |
  +----------------+   +-----------------+
````

---

## 🔹 Key Points

* Prometheus **runs inside the EKS cluster** as pods.
* It **pulls metrics** (scraping model) from various sources.
* Data is **stored locally in time-series format**.
* Other tools like **Grafana** connect to Prometheus to visualize the data.

---

## ✅ Summary

Prometheus = **eyes & ears of your EKS cluster**.
It ensures you can observe cluster health, workloads, and performance, and react to issues in real-time.

```

---

