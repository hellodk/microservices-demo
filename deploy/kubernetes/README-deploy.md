# Sock Shop — Kubernetes Deployment

Weaveworks Sock Shop microservices demo, modernised for **Kubernetes 1.34**, with an Nginx reverse-proxy and traffic-generator monitoring stack.

## Prerequisites

- Kubernetes ≥ 1.34 cluster
- `kubectl` configured with cluster access

## Quick Start

### 1. Deploy the Sock Shop application

```bash
kubectl apply -f deploy/kubernetes/complete-demo.yaml
```

This creates the `sock-shop` namespace and deploys all 14 microservices (carts, catalogue, front-end, orders, payment, queue-master, rabbitmq, session-db, shipping, user, and their databases).

### 2. Deploy the Nginx monitoring stack

```bash
kubectl apply -f deploy/kubernetes/manifests/30-nginx-monitoring-stack.yaml
```

This deploys into `sock-shop`:

| Resource | Purpose |
|---|---|
| `nginx-config` ConfigMap | Nginx config with structured JSON telemetry logging |
| `backend` Deployment + Service | Simple HTTP echo backend on port 8080 |
| `nginx` Deployment (3 replicas) + Service | Reverse proxy, NodePort 32766, metrics on port 8081 |
| `traffic-generator` Deployment | Continuous GET/POST/PUT traffic to nginx |
| `global-traffic-generator` Deployment | Multi-method, multi-path traffic with spoofed IPs |

### 3. Verify deployment

```bash
# Check all pods
kubectl get pods -n sock-shop

# Check all services
kubectl get svc -n sock-shop

# Wait for all pods to be ready
kubectl wait --for=condition=Ready pods --all -n sock-shop --timeout=300s
```

## Accessing the Application

| Service | URL |
|---|---|
| Sock Shop Front-End | `http://<NODE_IP>:30001` |
| Nginx (reverse proxy) | `http://<NODE_IP>:32766` |
| Nginx stub_status | `http://<NODE_IP>:32766` (port 8081 inside the pod) |

Get a node IP:

```bash
kubectl get nodes -o wide
```

## Monitoring Nginx Telemetry

View the structured JSON access logs from the nginx pods:

```bash
# Stream logs from all nginx pods
kubectl logs -n sock-shop -l app=nginx -f

# Pretty-print with jq
kubectl logs -n sock-shop -l app=nginx -f | jq .
```

## Tear Down

```bash
# Remove the nginx monitoring stack
kubectl delete -f deploy/kubernetes/manifests/30-nginx-monitoring-stack.yaml

# Remove the entire sock-shop deployment
kubectl delete -f deploy/kubernetes/complete-demo.yaml
```

## Constraints

See [CONSTRAINTS.md](CONSTRAINTS.md) for deployment rules:

1. All resources use the `sock-shop` namespace exclusively
2. No modifications beyond what is explicitly requested
