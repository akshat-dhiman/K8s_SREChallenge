# Kubernetes Frontend + Backend Demo

This repository demonstrates a simple **frontend + backend** application running on Kubernetes using **VirtualBox-based nodes** or any local K8s cluster.

The backend is a lightweight Python HTTP server, and the frontend is an Nginx-based webpage that calls the backend.

---

## Architecture Overview

```
Browser → Frontend (Nginx) → Backend (Python HTTP API)
```

Both services communicate inside the cluster using Kubernetes Service DNS.

---

# Step 1: Deploy Backend (Dummy API)

The backend returns a simple text response: **"Hello from backend"**.

## Backend Deployment (`backend-deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: python:3.9-slim
          command: ["python3", "-m", "http.server", "5000"]
          ports:
            - containerPort: 5000
```

## Backend Service (`backend-service.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
  type: NodePort
```

## Apply Backend

```bash
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml
```

## Get Backend NodePort

```bash
kubectl get svc backend-service
```

Example:

```
NodePort: 30080
```

### Backend Access

* **Inside cluster:** `http://backend-service:5000`
* **Outside cluster:** `http://<node-ip>:30080`

---

# Step 2: Deploy Frontend (Nginx Webpage)

The frontend loads an HTML page that uses JavaScript to call the backend.

## Frontend HTML ConfigMap (`frontend-configmap.yaml`)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-html
data:
  index.html: |
    <html>
      <body>
        <h1>Frontend App</h1>
        <p>Backend says:</p>
        <div id="result"></div>
        <script>
          fetch("http://backend-service:5000")
            .then(response => response.text())
            .then(data => document.getElementById("result").innerHTML = data)
            .catch(err => document.getElementById("result").innerHTML = "Error calling backend");
        </script>
      </body>
    </html>
```

## Frontend Deployment (`frontend-deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: nginx:alpine
          volumeMounts:
            - name: html
              mountPath: /usr/share/nginx/html/index.html
              subPath: index.html
      volumes:
        - name: html
          configMap:
            name: frontend-html
```

## Frontend Service (`frontend-service.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
  type: NodePort
```

## Apply Frontend

```bash
kubectl apply -f frontend-configmap.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
```

## Get Frontend NodePort

```bash
kubectl get svc frontend-service
```

Example:

```
NodePort: 30081
```

### Frontend Access (Browser)

```
http://<node-ip>:30081
```

You should see:

```
Frontend App
Backend says: Hello from backend
```

# Result

We now have a working **frontend + backend** Kubernetes setup

