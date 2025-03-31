# API Gateway on a Local Kubernetes Instance with Multipass

## Introduction
This guide provides a step-by-step process for developers to set up a local API Gateway using Multipass and MicroK8s on an Ubuntu virtual machine. For this project, I used a Mac Pro with an Apple M2 Pro processor and 32GB of RAM, ensuring smooth performance for running the VM and Kubernetes cluster locally. By leveraging Multipass to create a lightweight VM and MicroK8s for a minimal Kubernetes cluster, you can quickly deploy the APISIX API Gateway and a demo application for testing. This setup is ideal for development, experimentation, or learning purposes, offering a self-contained environment that mirrors cloud-based Kubernetes deployments. The process includes VM creation, Kubernetes configuration, APISIX installation, dashboard access, and deployment of a sample application, all tailored for a local workflow.

---
## Architecture Overview

The following network block diagram illustrates the architecture of the local API Gateway setup, showing the key components and their network interactions. This includes the host machine (Mac Pro), the Ubuntu VM managed by Multipass, the MicroK8s Kubernetes cluster, the APISIX API Gateway components, and the demo application, all connected via specific IPs and ports.

```mermaidgraph TD
    A[<br> Host Machine <br>] -->|Multipass| B[Ubuntu VM: apigateway <br> IP: 192.168.64.15]

    B -->|Hosts| C[MicroK8s Cluster <br> Single Node]

    subgraph MicroK8s Cluster
        C --> D[Kube-System Namespace]
        C --> E[Ingress-APISIX Namespace]
        C --> F[App Namespace]

        D -->|Service: kubernetes-dashboard <br> Port: 443| G[Kubernetes Dashboard <br> Exposed via Port Forward <br> 192.168.64.15:10443]
        
        E --> H[APISIX Gateway <br> NodePort: 32178 <br> ClusterIP: 10.152.183.220]
        E --> I[APISIX Admin <br> ClusterIP: 10.152.183.23 <br> Port: 9180]
        E --> J[APISIX Ingress Controller <br> ClusterIP: 10.152.183.253 <br> Port: 80]
        E --> K[ETCD Cluster <br> ClusterIP: 10.152.183.21 <br> Ports: 2379, 2380]

        F --> L[App Service <br> ClusterIP: 10.152.183.204 <br> Port: 8080]
        F --> M[App Deployment <br> Pod IP: Dynamic]
    end

    A -->|HTTP Request <br> localhost:32178| H
    H -->|Routes Traffic| L
    G -->|HTTPS Access <br> Token Auth| A
    J -->|Manages| H
    K -->|Stores Config| H
    M -->|Serves| L
```
---

## Prerequisites

- **Multipass**: A tool to quickly generate cloud-style Ubuntu VMs on Linux, macOS, or Windows.
- **Supported OS**: Linux, macOS, or Windows.
- **Hardware**: At least 4GB RAM, 10GB disk space, and 2 CPU cores available for the VM.

---

## Step 1: Install Multipass

Install Multipass to create and manage VMs:

```bash
# Visit the official site for installation instructions
https://canonical.com/multipass/install
```

---

## Step 2: Create a Virtual Machine

Launch an Ubuntu VM with specified resources:

```bash
multipass launch -c 2 -d 10g -m 4g -n apigateway
```

- `-c 2`: 2 CPU cores
- `-d 10g`: 10GB disk space
- `-m 4g`: 4GB memory
- `-n apigateway`: VM name

Verify the VM is running:

```bash
multipass list
```

**Example Output:**
```
Name         State    IPv4          Image
apigateway   Running  192.168.64.15 Ubuntu 24.04 LTS
```

Access the VM shell:

```bash
multipass shell apigateway
```

---

## Step 3: Set Up Kubernetes with MicroK8s

Install MicroK8s, a lightweight Kubernetes distribution:

```bash
sudo snap install microk8s --classic
```

**Example Output:**
```
2025-03-30T17:24:17-04:00 INFO Waiting for automatic snapd restart...
microk8s (1.32/stable) v1.32.2 from Canonical✓ installed
```

Configure permissions and directories:

```bash
mkdir ~/.kube
sudo usermod -a -G microk8s ubuntu
sudo chown -R ubuntu ~/.kube
exit
```

Restart the VM and reconnect:

```bash
multipass restart apigateway
multipass shell apigateway
```

Verify MicroK8s is ready:

```bash
microk8s status --wait-ready
```

**Example Output (partial):**
```
microk8s is running
high-availability: no
  datastore master nodes: 127.0.0.1:19001
  datastore standby nodes: none
addons:
  enabled:
    dns       # CoreDNS
    ha-cluster # High availability configuration
    helm      # Helm package manager
    helm3     # Helm 3 package manager
  disabled:
    dashboard # Kubernetes dashboard
    ingress   # Ingress controller
    ...
```

Enable additional addons:

```bash
microk8s enable dashboard
microk8s enable hostpath-storage
```

Install `kubectl` for interacting with Kubernetes:

```bash
sudo snap install kubectl --classic
```

Generate a kubeconfig file:

```bash
microk8s config > ~/.kube/config
```

---

## Step 4: Install APISIX API Gateway

Install Helm to manage Kubernetes packages:

```bash
sudo snap install helm --classic
```

Add required Helm repositories:

```bash
helm repo add apisix https://charts.apiseven.com
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Set the APISIX admin API version (v3 in this example):

```bash
ADMIN_API_VERSION=v3
```

Install APISIX with Helm:

```bash
helm install apisix apisix/apisix \
  --set service.type=NodePort \
  --set ingress-controller.enabled=true \
  --create-namespace \
  --namespace ingress-apisix \
  --set ingress-controller.config.apisix.serviceNamespace=ingress-apisix \
  --set ingress-controller.config.apisix.adminAPIVersion=$ADMIN_API_VERSION
```

Verify services in the `ingress-apisix` namespace:

```bash
kubectl get service -n ingress-apisix
```

Get the APISIX Gateway URL:

```bash
export NODE_PORT=$(kubectl get -n ingress-apisix -o jsonpath="{.spec.ports[0].nodePort}" services apisix-gateway)
export NODE_IP=$(kubectl get nodes -n ingress-apisix -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT
```

---

## Step 5: Access the Kubernetes Dashboard

Forward the dashboard port (run this in a second terminal):

```bash
multipass exec apigateway -- sudo /snap/bin/microk8s kubectl port-forward -n kube-system service/kubernetes-dashboard 10443:443 --address 0.0.0.0
```

**Output:**
```
Forwarding from 0.0.0.0:10443 -> 8443
```

Retrieve the dashboard token (run this in a third terminal):

```bash
multipass exec apigateway -- sudo /snap/bin/microk8s kubectl -n kube-system describe secret $(multipass exec apigateway -- sudo /snap/bin/microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
```

**Example Output (partial):**
```
Data
====
token: eyJhbGciOiJSUzI1NiIsImtpZCI6IjM5WW5zbVlYbHZWeVhIcmQ3dGJtaVlZb3dDOFJGZHR4ZXBoUnhsNUdfcEEifQ...
```

Access the dashboard at:

```
https://192.168.64.15:10443/#/login
```

Log in using the token from the previous step.

<img src="./kube.jpg">

<img src="./kube2.jpg">

---

## Step 6: Deploy a Demo Application

Check APISIX endpoints and services:

```bash
kubectl get endpoints,service -n ingress-apisix
```

Test the gateway:

```bash
curl localhost:32178
```

**Expected Output:**
```
{"error_msg":"404 Route Not Found"}
```

Apply the demo application manifests:

```bash
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

Verify the demo resources:

```bash
kubectl get pod,service -n demo
```

**Example Output:**
```
NAME                                  READY  STATUS   RESTARTS  AGE
pod/echo-deployment-6b57bd58b-r8nhh   1/1    Running  0         2m39s

NAME                   TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)    AGE
service/echo-service   ClusterIP  10.152.183.204  <none>       8080/TCP   6s
```

Test the demo application:

```bash
curl localhost:32178
```

**Example Output:**
```json
{
  "path": "/",
  "headers": {
    "host": "localhost:32178",
    "x-real-ip": "192.168.64.15",
    "x-forwarded-for": "192.168.64.15",
    "user-agent": "curl/8.5.0",
    "accept": "*/*"
  },
  "method": "GET",
  "body": "",
  "hostname": "localhost",
  "ip": "192.168.64.15",
  "protocol": "http",
  "query": {},
  "os": {
    "hostname": "echo-deployment-6b57bd58b-r8nhh"
  }
}
```

---

## Notes

- Replace `192.168.64.15` with your VM’s IP if it differs.
- For APISIX v2.x, adjust the `ADMIN_API_VERSION` to `v2` in Step 4.

## References

1. **APISIX Ingress Controller GitHub**: [https://github.com/apache/apisix-ingress-controller](https://github.com/apache/apisix-ingress-controller) - Official GitHub repository for the APISIX Ingress Controller.
2. **APISIX Minikube Deployment Docs**: [https://apisix.apache.org/docs/ingress-controller/deployments/minikube/](https://apisix.apache.org/docs/ingress-controller/deployments/minikube/) - Documentation for deploying APISIX on Minikube (related reference).
3. **Docker HTTP/HTTPS Echo**: [https://code.mendhak.com/docker-http-https-echo/](https://code.mendhak.com/docker-http-https-echo/) - Source for the echo server used in the demo application.
