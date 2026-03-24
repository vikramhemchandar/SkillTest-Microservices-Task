# Kubernetes Deployment & Configuration Guide

## 1. Overview
This document outlines the Kubernetes architecture for the microservices project, detailing the manifest configurations and providing a reproducible command list for creating the cluster, configuring Ingress, and deploying the services.

* * *

## 2. Manifest Explanations

### Microservice Deployments & Services
**Files:** `user-service-deployment.yml`, `product-service-deployment.yml`, `order-service-deployment.yml`, `gateway-service-deployment.yml`

Each of the microservices uses a standard Kubernetes Deployment strategy:
- **Replicas:** 2 Pod per service.
- **Liveness & Readiness Probes:** Configured to check the `/health` endpoint to ensure traffic is only routed to healthy Pods.
- **Resources:** Guaranteed `100m` CPU / `128Mi` Memory with limits set to `250m` CPU / `256Mi` Memory to ensure stability.
- **Service:** Exposed via `ClusterIP` to allow seamless internal communication across the cluster on native ports.

### Custom Cluster Configuration
**File:** `kind-cluster-config.yml`

This file defines the custom architecture for our local KIND (Kubernetes IN Docker) cluster. By default, KIND nodes do not map standard web ports to your computer natively.
- **Port Mappings:** It explicitly binds host ports `80` (HTTP) and `443` (HTTPS) to the cluster's control plane, creating a bridge from your browser into the cluster.
- **Node Labels:** It patches the kubelet with the `ingress-ready=true` label. This label is a strict requirement; it acts as a target telling the NGINX Ingress Controller exactly which node to bind itself to.

Without this file mapping the ports and labeling the nodes, adding an Ingress Controller to the cluster would be useless because the external traffic could never enter the cluster on port 80.

### Understanding Ingress and Ingress Controllers
In Kubernetes, an **Ingress** and an **Ingress Controller** work closely together to route external traffic into the cluster.

- **Ingress Controller:** This is the actual running application (like NGINX) that acts as the cluster's reverse proxy and load balancer. By default, Kubernetes does not come with an Ingress Controller; it must be installed manually. Without an active controller, any Ingress rules you define will sit idle and do nothing.
- **Ingress Resource:** This is the configuration file (`ingress.yml`) that tells the Controller *how* to route traffic. It maps specific external URLs directly to internal `ClusterIP` services.

**Our Configuration (`ingress.yml`):**
Our Ingress manages traffic arriving at the `skilltest` domain. We configure it using a simple `Prefix` path matching strategy:
- `/gateway` → `gateway-service:3003`
- `/users` → `user-service:3000`
- `/products` → `product-service:3001`
- `/orders` → `order-service:3002`

Because we use `Prefix` pathing without rewrite rules, NGINX reliably passes the URL exactly as typed to our Node.js Express applications.

* * *

## 3. Master Command List

Here is the exact sequence of commands required to spin up the cluster from scratch, set up the Ingress Controller, explicitly deploy the workloads, and optionally expose the services locally.

### Step 1: Create the KIND Cluster
Create the cluster using the custom configuration file (which elegantly maps ports 80/443 for our ingress rules):
```bash
kind create cluster --config k8s/kind-cluster-config.yml
```

### Step 2: Install the NGINX Ingress Controller for KIND
Before applying your custom ingress rules, you must deploy the official NGINX controller into the cluster so that it can monitor and execute those routes:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```
Wait a moment for the controller to properly initialize and become ready:
```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

### Step 3: Load Docker Images into KIND
Transfer your locally built microservice images directly into the cluster so they are available for deployment without a remote container registry:
```bash
kind load docker-image gateway-service:v1.0 --name dev
kind load docker-image order-service:v1.0 --name dev
kind load docker-image product-service:v1.0 --name dev
kind load docker-image user-service:v1.0 --name dev
```

### Step 4: Apply the Manifests
Deploy the microservices, their corresponding cluster services, and the Ingress routing rules:
```bash
kubectl apply -f k8s/gateway-service-deployment.yml
kubectl apply -f k8s/order-service-deployment.yml
kubectl apply -f k8s/product-service-deployment.yml
kubectl apply -f k8s/user-service-deployment.yml
kubectl apply -f k8s/ingress.yml
```

### Step 5: (Optional) Local Port-Forwarding
If you wish to bypass the Ingress entirely and access the services directly via `localhost` (mimicking a pure `docker-compose` setup), run these port-forward commands in separate terminals:
```bash
kubectl port-forward svc/gateway-service 3003:3003
kubectl port-forward svc/order-service 3002:3002
kubectl port-forward svc/product-service 3001:3001
kubectl port-forward svc/user-service 3000:3000
```

* * *

## 4. Visual Proof & Screenshots

*(Please paste your validation screenshots in the placeholders below)*

**Screenshot 1: Cluster Nodes and Pods Running (`kubectl get pods -A`)**
<img width="637" height="175" alt="image" src="https://github.com/user-attachments/assets/92ccb2d1-6bad-47ce-b4a2-1d8ab72d5221" /><br>
<img width="676" height="115" alt="image" src="https://github.com/user-attachments/assets/6650ecd3-de8e-46fd-a231-799ccf8f37cb" /><br>
<img width="627" height="133" alt="image" src="https://github.com/user-attachments/assets/9f63625d-96ca-4ff6-92bd-b17325b9f473" /><br>


**Screenshot 2: Ingress Resource Successfully Created (`kubectl get ingress`)**
<img width="651" height="85" alt="image" src="https://github.com/user-attachments/assets/b407b90b-6551-48b4-8dbc-8dee4e2100a1" />
<br>

**Screenshot 3: Successful Browser Check for Users (`http://skilltest/users`)** <br>
<img width="389" height="237" alt="image" src="https://github.com/user-attachments/assets/d5509846-9d54-485f-82a5-c395c6c372e0" />
<br>

**Screenshot 4: Successful Browser Check for Products (`http://skilltest/products`)**
<img width="436" height="260" alt="image" src="https://github.com/user-attachments/assets/3d0f2c7b-c474-45f2-b6c0-e6463c4e4ceb" />
<br>

**Screenshot 5: Successful Browser Check for Orders (`http://skilltest/orders`)**
<img width="412" height="209" alt="image" src="https://github.com/user-attachments/assets/a6a28247-11c7-4e5e-8905-71f5c5e449cd" />
<br>

**Screenshot 6: Successful Browser Check for gateway (`http://skilltest/gateway`)**
- users: http://skilltest/gateway/api/users <br>
<img width="436" height="228" alt="image" src="https://github.com/user-attachments/assets/524f3171-b5e3-4de6-aa03-cd48729e8561" /><br>
- products - http://skilltest/gateway/api/products <br>
<img width="472" height="259" alt="image" src="https://github.com/user-attachments/assets/1598c7e0-ad08-4ab3-9070-90d159ffe898" /><br>
