# Kubernetes Monitoring Setup with Helm, Prometheus, and Grafana

This guide walks through installing **Prometheus + Grafana** on Kubernetes using Helm, including troubleshooting common installation errors.

---

## Prerequisites

* Kubernetes cluster up and running
* Helm installed
* `kubectl` configured

---

## Step 1: Install Helm 3

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Verify installation:

```bash
helm version
```

---

## Step 2: Add Helm Repositories

We will use the Prometheus community Helm charts and Grafana charts:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

---

## Step 3: Create Namespace for Monitoring

```bash
kubectl create namespace monitoring
```

Isolating monitoring resources in a separate namespace is a good practice.

---

## Step 4: Deploy Prometheus + Node Exporter

Deploy **kube-prometheus-stack**, which includes Prometheus, Node Exporter, Alertmanager, and Grafana.

```bash
helm install kube-prom-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set grafana.enabled=true \
  --set prometheus-nodeExporter.enabled=true
```

---

## Step 5: Check Deployment

```bash
kubectl get pods -n monitoring
```

---

## Step 6: Access Grafana

Since Grafana runs inside the cluster, port-forward to access it:

```bash
kubectl port-forward svc/kube-prom-stack-grafana 3000:80 -n monitoring
```

Open your browser:

```
http://localhost:3000
```

Default login:

* Username: `admin`
* Password: `prom-operator` (or as provided by Helm notes)

---

## Troubleshooting Common Helm Installation Errors

### Error 1: Existing ClusterRole

```
ClusterRole "kube-prom-stack-grafana-clusterrole" in namespace "" exists and cannot be imported into the current release...
```

#### Fix

```bash
kubectl delete clusterrole kube-prom-stack-grafana-clusterrole
kubectl delete clusterrolebinding kube-prom-stack-grafana-clusterrolebinding
```

Then retry Helm install:

```bash
helm install kube-prom-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set grafana.enabled=true \
  --set prometheus-nodeExporter.enabled=true \
  --skip-crds \
  --timeout 20m
```

### Error 2: Existing kube-state-metrics ClusterRole

```
ClusterRole "kube-prom-stack-kube-state-metrics" in namespace "" exists and cannot be imported...
```

#### Fix

```bash
kubectl delete clusterrole kube-prom-stack-kube-state-metrics
kubectl delete clusterrolebinding kube-prom-stack-kube-state-metrics
```

Retry Helm install:

```bash
helm install kube-prom-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.enabled=true \
  --set prometheus-nodeExporter.enabled=true \
  --timeout 20m
```

### Error 3: Context deadline exceeded

* This usually occurs due to network issues or cluster resource constraints.
* Ensure cluster nodes are ready:

```bash
kubectl get nodes
```

* Increase Helm timeout if necessary:

```bash
--timeout 30m
```

---

## Problem Faced:

I am using a Vagrant and Virtual Box based setup with limited resources on my laptop due to which CPU was spiking to its max while installing ```kube-prom-stack```, so I couldn't actually set it up on my cluster. But this can be done, I am aware of the process.