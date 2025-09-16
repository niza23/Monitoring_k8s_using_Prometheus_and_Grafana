
# Monitoring AWS EKS using Prometheus and Grafana

This guide explains how to set up **Prometheus** and **Grafana** to monitor an **Amazon Elastic Kubernetes Service (EKS)** cluster.
We will go step by step, installing all required tools (`kubectl`, `eksctl`, `helm`), creating an EKS cluster, deploying monitoring stacks, and finally exposing Grafana dashboards.

---

## Prerequisites

Before you begin, make sure you have:

* An **Ubuntu Server** (18.04 or later recommended).
* AWS CLI installed and configured with your **AWS credentials** (`aws configure`).
* Permissions to create EKS clusters (Admin or equivalent IAM role).

---

## Step 1: Install and Configure `kubectl`

`kubectl` is the **command-line interface** for interacting with Kubernetes clusters. It allows you to deploy applications, check cluster resources, and manage workloads.

```bash
# Download kubectl binary from AWS EKS S3 bucket
sudo curl --silent --location -o /usr/local/bin/kubectl \
  https://s3.us-west-2.amazonaws.com/amazon-eks/1.22.6/2022-03-09/bin/linux/amd64/kubectl

# Make kubectl executable
sudo chmod +x /usr/local/bin/kubectl 

# Verify kubectl installation
kubectl version --short --client
```

* `curl` downloads the binary.
* `chmod +x` makes it executable.
* `kubectl version --short --client` confirms the installation.

---

## Step 2: Install and Configure `eksctl`

`eksctl` is a **simple command-line tool** for creating and managing EKS clusters. It abstracts a lot of AWS complexity and allows us to spin up clusters quickly.

```bash
# Download latest eksctl release and extract it to /tmp
curl --silent --location \
  "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
  | tar xz -C /tmp

# Move eksctl binary to /usr/local/bin for global access
sudo mv /tmp/eksctl /usr/local/bin

# Verify installation
eksctl version
```

* The binary is fetched directly from GitHub.
* We place it in `/usr/local/bin` so it’s available system-wide.
* `eksctl version` checks if it’s working.

---

## Step 3: Install Helm

[Helm](https://helm.sh/) is the **package manager for Kubernetes**. Think of it like `apt` or `yum` but for Kubernetes resources. We’ll use it to install Prometheus and Grafana.

```bash
# Download Helm installation script
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

# Give execution permission
chmod 700 get_helm.sh

# Run the script to install Helm
./get_helm.sh

# Alternatively, install a specific version (e.g., v3.8.2)
DESIRED_VERSION=v3.8.2 bash get_helm.sh
```

Verify installation:

```bash
helm version
```

---

## Step 4: Create an Amazon EKS Cluster

Now that we have `kubectl` and `eksctl`, let’s create an EKS cluster.

```bash
eksctl create cluster \
  --name=eks-cluster \
  --region=ap-south-1 \
  --version=1.29 \
  --nodegroup-name=my-nodes \
  --node-type=t3.medium \
  --managed \
  --nodes=2 \
  --nodes-min=2 \
  --nodes-max=3
```

Explanation:

* `--name`: Cluster name.
* `--region`: AWS region (e.g., `ap-south-1` for Mumbai).
* `--version`: Kubernetes version.
* `--node-type`: EC2 instance type for worker nodes.
* `--nodes`: Desired number of worker nodes.
* `--nodes-min` / `--nodes-max`: Auto-scaling limits.

⚠️ This process takes **15–20 minutes**. Once done, you can check it in the AWS Console.

Verify cluster:

```bash
eksctl get cluster --name eks-cluster --region ap-south-1
```

Update kubeconfig to connect with cluster:

```bash
aws eks update-kubeconfig --name eks-cluster --region ap-south-1
```

Check connectivity:

```bash
kubectl get nodes
kubectl get ns
```

---

## Step 5: Add Helm Stable Charts

```bash
helm repo add stable https://charts.helm.sh/stable
```

This adds the stable Helm repository (contains widely used charts).

---

## Step 6: Add Prometheus Helm Repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

This gives us access to official **Prometheus charts**.

---

## Step 7: Create Prometheus Namespace

Namespaces help organize Kubernetes resources.

```bash
kubectl create namespace prometheus
kubectl get ns
```

---

## Step 8: Install Prometheus with Grafana

```bash
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
```

This installs:

* **Prometheus** (for metrics collection).
* **Grafana** (for visualization).
* Alertmanager and exporters (for monitoring cluster components).

Verify installation:

```bash
kubectl get pods -n prometheus
kubectl get svc -n prometheus
```

---

## Step 9: Expose Prometheus and Grafana

By default, services are exposed internally using **ClusterIP**.
We need to expose them via **LoadBalancer** for external access.

### Expose Prometheus

```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
```

* Change `type: ClusterIP` → `type: LoadBalancer`
* Save and exit.

Check service:

```bash
kubectl get svc -n prometheus
```

You’ll now see an **external LoadBalancer endpoint** (accessible on port `9090`).

### Expose Grafana

```bash
kubectl edit svc stable-grafana -n prometheus
```

* Again, change `ClusterIP` → `LoadBalancer`.

Check Grafana endpoint:

```bash
kubectl get svc -n prometheus
```

Now you can access Grafana in your browser.

### Get Grafana Login Credentials

```bash
kubectl get secret --namespace prometheus stable-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

* Username: `admin`
* Password: (output of the above command)

---

## Step 10: Explore Grafana Dashboards

Grafana provides pre-configured dashboards for:

* **CPU & RAM usage**
* **Pods by namespace**
* **Pod lifecycle history**
* **Horizontal Pod Autoscaler (HPA) metrics**
* **Resource usage by container**
* **Network bandwidth & packet rates**

---

## Step 11: Clean Up (Delete EKS Cluster)

When you are done, delete the cluster to avoid AWS costs:

```bash
eksctl delete cluster --name eks-cluster
```

---

## 🎯 Summary

You have now:

1. Installed `kubectl`, `eksctl`, and `helm`.
2. Created an **EKS cluster**.
3. Deployed **Prometheus and Grafana** using Helm.
4. Exposed services using **LoadBalancer**.
5. Accessed Grafana dashboards for real-time monitoring.
6. Deprovisioned the cluster after use.

---


