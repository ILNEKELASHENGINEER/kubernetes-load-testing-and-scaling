# Kubernetes Auto-Scaling & Monitoring (Minikube)

This project demonstrates how to deploy a containerized application on Kubernetes, enable auto-scaling based on CPU usage, simulate traffic, and observe scaling behavior in real time.

---

## Overview

In this project, I:

* Set up a local Kubernetes cluster using Minikube
* Deployed an application using a Deployment
* Exposed it using a Service
* Configured Horizontal Pod Autoscaler (HPA)
* Simulated traffic using k6
* Monitored scaling behavior using kubectl metrics

---

## Architecture (Simple workflow)

User Traffic → Service → Pods → CPU Increase → HPA → Scale Pods → Monitor Metrics

---

## Setup & Installation

### 1. Install Minikube

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

Start cluster:

```bash
minikube start --driver=docker
```

---

### 2. Install kubectl

```bash
sudo apt update
sudo apt install kubectl -y
```

Verify:

```bash
kubectl get nodes
```

---

### 3. Enable Metrics Server (Required for HPA)

```bash
minikube addons enable metrics-server
```

---

## Deployment

### 4. Create Deployment

```bash
touch deployment.yaml
nano deployment.yaml
```

Example config (nginx):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "200m"
        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

---

### 5. Expose Service

```bash
kubectl expose deployment nginx-deployment --type=NodePort --port=80
```

Get URL:

```bash
minikube service nginx-deployment --url
```

---

## 📈🚀 Auto Scaling (HPA)

```bash
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=2 --max=8
```

Check:

```bash
kubectl get hpa
```

---

## Load Testing

### Install k6

```bash
sudo apt-get update
sudo apt-get install k6
```

### Create test script

```bash
nano load.js
```

Example:

```javascript
import http from 'k6/http';

export default function () {
  http.get('http://<MINIKUBE-IP>:<NODEPORT>');
}
```

### Run load

```bash
k6 run --vus 60 --duration 45s load.js
k6 run --vus 80 --duration 60s load.js
```

---

## 📊 Monitoring

### View metrics

```bash
kubectl top nodes
kubectl top pods
```

### Watch live scaling

```bash
watch kubectl top pods
kubectl get hpa -w
```

### Check pods

```bash
kubectl get pods
```

---

## Screenshots (Add Here)

* Deployment running (pods list)
* HPA scaling (2 → 8 pods)
* kubectl top output
* k6 load test output

---

## Cleanup

### Delete resources

```bash
kubectl delete hpa nginx-deployment
kubectl delete service nginx-deployment
kubectl delete deployment nginx-deployment
```

Or:

```bash
kubectl delete -f deployment.yaml
```

### Verify cleanup

```bash
kubectl get all
```

---

### Stop / Delete cluster

```bash
minikube stop
minikube delete
```

---

## Outcomes

* Kubernetes Deployment and Service basics
  <br><br>
   <img width="846" height="288" alt="Screenshot 2026-03-18 193057" src="https://github.com/user-attachments/assets/769b4144-ae85-4838-8dfa-90171ccf6004" />
<br><br>
  <img width="795" height="305" alt="Screenshot 2026-03-18 193708" src="https://github.com/user-attachments/assets/275bde2b-88e0-4545-830d-6c15e1b963f8" />
<br><br>
  <img width="923" height="365" alt="Screenshot 2026-03-18 200059" src="https://github.com/user-attachments/assets/d276e982-ada0-4d9c-bb41-494dd11f6aa4" />
 <br><br>
* HTTP traffic is initated using k6 tool
<br><br>
  <img width="1919" height="646" alt="Screenshot 2026-03-18 195044" src="https://github.com/user-attachments/assets/2fbff3a5-cfdf-4397-8784-676a6ee2686f" />
<br><br>
* Application is live with huge traffic - Horizontal Pod Autoscaler (HPA) - pods getting multiplied (replica)
<br><br>
  <img width="707" height="205" alt="Screenshot 2026-03-18 195134" src="https://github.com/user-attachments/assets/45922dd7-b572-4ce8-8444-9bc9bb036bf9" />
<br><br>
* Monitoring using kubectl metrics
<br><br>
  <img width="929" height="178" alt="image" src="https://github.com/user-attachments/assets/f629f93d-7c9a-438d-9e57-a807428a80c1" />

---

## ⚠️ Debug Tips 🚀

* If metrics not available:
  disable and enable the addon again!!!
  ```bash
  minikube addons disable metrics-server
  minikube addons enable metrics-server
  ```

* If pods not scaling:

  * Ensure CPU requests/limits are set
  * Check:

    ```bash
    kubectl get hpa
    ```

* If service not accessible:

  ```bash
  minikube service nginx-deployment --url
  ```

* If kubectl not working:

  ```bash
  kubectl get nodes
  ```

---

## 📌 Commands Summary

### Setup

```bash
minikube start --driver=docker
kubectl get nodes
minikube addons enable metrics-server
```

### Deployment

```bash
kubectl apply -f deployment.yaml
kubectl expose deployment nginx-deployment --type=NodePort --port=80
```

### Scaling

```bash
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=2 --max=8
kubectl get hpa
```

### Testing

```bash
k6 run --vus 60 --duration 45s load.js
k6 run --vus 80 --duration 60s load.js
```

### Monitoring

```bash
kubectl top pods
watch kubectl top pods
kubectl get hpa -w
```

### Cleanup

```bash
kubectl delete hpa nginx-deployment
kubectl delete service nginx-deployment
kubectl delete deployment nginx-deployment
minikube stop
minikube delete
```

---

## 🏁 Final Note

This project demonstrates how traffic-driven load increases CPU usage, which triggers Kubernetes autoscaling. It gives a practical understanding of how real-world systems handle dynamic workloads.

---
