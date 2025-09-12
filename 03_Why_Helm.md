
# ⎈ Why Use Helm for Prometheus & Grafana in EKS?

## 🔹 What is Helm?
Helm is often called the **"package manager for Kubernetes"**.  
It allows you to install, upgrade, and manage complex Kubernetes applications easily using **Helm Charts** (pre-packaged sets of YAML files).

---

## 🔹 Why Helm for Prometheus & Grafana?
Prometheus and Grafana are **not just single pods**.  
They require multiple Kubernetes resources to run properly, for example:
- Deployments  
- Services  
- ConfigMaps  
- Secrets  
- ServiceAccounts  
- Roles & RoleBindings  

👉 Manually writing and managing all these YAML files would be **long, repetitive, and error-prone**.  

Helm simplifies this by:
- Bundling everything into a **Helm Chart**  
- Allowing you to install Prometheus or Grafana with **a single command**  
- Managing **upgrades, rollbacks, and configurations** easily  
- Providing **community-maintained charts** (`prometheus-community/kube-prometheus-stack`) that include **best practices**  

---

## 🛠 What happens when you run `helm install`

1. **Find the chart**

   * A Helm **chart** = package of YAML files (like an app installer).
   * Helm looks in a repo (like “prometheus-community”) and downloads it.

2. **Mix your settings**

   * The chart comes with default settings (`values.yaml`).
   * You can override with your own file (`-f myvalues.yaml`) or `--set`.
   * Helm mixes them → final configuration.

3. **Prepare YAMLs (rendering)**

   * Helm takes the templates inside the chart (they’re like blueprints).
   * It replaces placeholders (`{{ }}`) with your actual values.
   * End result = plain Kubernetes YAMLs (Deployments, Services, ConfigMaps, etc.).

   👉 You can preview this with:

   ```bash
   helm template myrel chart
   ```

4. **Send YAMLs to Kubernetes**

   * Helm calls the Kubernetes API (same as `kubectl apply`).
   * Kubernetes creates the resources (pods, services, CRDs).

5. **Store release info**

   * Helm saves a record of what it installed inside the cluster (in a Secret).
   * This lets you later **upgrade**, **rollback**, or **delete** the release safely.

6. **Check status (optional)**

   * If you use `--wait`, Helm waits until pods are running.
   * If pods fail, `--atomic` can roll everything back.

---

## 🌀 What happens later

* **Upgrade**: You change values, run `helm upgrade`. Helm renders again, compares with old release, and applies only changes.
* **Rollback**: Helm can roll back to a previous version because it saved all past YAMLs as history.
* **Uninstall**: Helm deletes the resources and removes its history record.

---

## ⚡ Why this is useful for Prometheus & Grafana

* Without Helm → you’d need to apply **dozens of YAML files** (Deployments, Services, ConfigMaps, CRDs, Roles, etc.) by hand.
* With Helm → `helm install prometheus ...` handles everything in the right order.
* Helm also makes **upgrades easy** (e.g., new Prometheus version).

---

## 🔍 Simple visual

```text
You (helm install) 
    ↓
Helm chart (templates + default values)
    ↓ (merge with your values)
Rendered YAML (Deployment, Service, CRDs, etc.)
    ↓
Kubernetes API (creates pods, services, etc.)
    ↓
Cluster is running Prometheus + Grafana
    ↓
Helm stores release info (so you can upgrade/rollback later)
```

---

👉 In short:
**Helm = package manager for Kubernetes apps.**
It saves you from manually writing and applying 50+ YAMLs.
It also gives you version control for your deployments.

---

## 🔹 Key Benefits

* **One command setup** → Instead of 20+ YAML files.
* **Reusability** → Charts are parameterized; you can customize them without rewriting.
* **Consistency** → Everyone uses the same standardized chart.
* **Easier upgrades** → Just run `helm upgrade`.
* **Rollback support** → If something fails, run `helm rollback`.

---

## ✅ Summary

We use **Helm for Prometheus and Grafana in EKS** because:

* They are complex apps with many components.
* Helm provides **automation, consistency, and flexibility**.
* It saves time, reduces errors, and makes cluster management much easier.
