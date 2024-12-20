### `README.md`


# Caddy 2 Load Balancer in AKS with High Availability

This repository contains the necessary Kubernetes manifests and configurations to deploy **Caddy 2** as a high-availability load balancer in an Azure Kubernetes Service (AKS) cluster. The setup includes multiple replicas of Caddy running in a ReplicaSet, with load balancing for backend services deployed within the same AKS cluster.

---

## Features

- **High Availability**: Caddy is deployed as a ReplicaSet with 3 replicas to ensure resilience.
- **Load Balancing**: Caddy distributes traffic across multiple backend services.
- **Health Checks**: Caddy performs health checks on backend services to ensure only healthy instances receive traffic.
- **Internal Networking**: Designed for AKS clusters in a Virtual Network (VNET).

---

## Architecture Overview

1. **Caddy ReplicaSet**: Runs multiple instances of Caddy for high availability.
2. **ConfigMap**: Stores the Caddy configuration (`Caddyfile`), including load balancing rules and health check paths.
3. **Service**: Exposes Caddy to the AKS cluster.
4. **Backend Services**: Applications to be load balanced, exposed via Kubernetes services.

---

## Prerequisites

- An AKS cluster with a VNET configured.
- `kubectl` installed and configured to manage the cluster.
- Backend services deployed in the cluster with proper health check endpoints.

---

## Deployment Steps

### 1. Clone the Repository

```sh
git clone https://github.com/your-repository/caddy-aks-loadbalancer.git
cd caddy-aks-loadbalancer
```

### 2. Create the Namespace

Apply the namespace manifest:

```sh
kubectl apply -f namespace.yaml
```

### 3. Deploy the ConfigMap

Apply the ConfigMap containing the `Caddyfile` configuration:

```sh
kubectl apply -f configmap.yaml
```

### 4. Deploy the ReplicaSet

Apply the ReplicaSet manifest to launch Caddy 2 instances:

```sh
kubectl apply -f replicaset.yaml
```

### 5. Expose the Caddy Service

Apply the Service manifest to expose Caddy within the VNET:

```sh
kubectl apply -f service.yaml
```

---

## Configuration Details

### Caddyfile

The `Caddyfile` defines load balancing and health checks. Below is an example configuration:

```plaintext
{
    auto_https disable
}

:80 {
    reverse_proxy / {
        to backend-service-1.default.svc.cluster.local:8080
           backend-service-2.default.svc.cluster.local:8080
           backend-service-3.default.svc.cluster.local:8080

        health_path /health
        health_interval 10s
        health_timeout 2s
        lb_policy round_robin
    }

    respond /health 200
}
```

- **`to`**: List of backend services to balance traffic.
- **`health_path`**: The path used to perform health checks on backend services.
- **`health_interval`**: Frequency of health checks (default: 10 seconds).
- **`health_timeout`**: Timeout for health checks (default: 2 seconds).
- **`lb_policy`**: Load balancing policy (default: round-robin).

---

## Verifying the Deployment

### Check the Pods

Verify that the Caddy pods are running:

```sh
kubectl get pods -n caddy-namespace
```

### Test the Load Balancer

To test the Caddy service:
- Access `http://caddy-service.caddy-namespace.svc.cluster.local` from within the cluster.
- Use tools like `curl` or a browser to check if traffic is correctly distributed across the backend services.

---

## Customization

1. **Backend Services**: Update the backend service names and namespaces in the `Caddyfile` to match your environment.
2. **Replica Count**: Adjust the `replicas` field in `replicaset.yaml` for your availability requirements.
3. **Exposing the Service Externally**: Modify the service type to `LoadBalancer` if external access is needed.

---

## Cleanup

To remove all resources:

```sh
kubectl delete -f replicaset.yaml
kubectl delete -f configmap.yaml
kubectl delete -f service.yaml
kubectl delete -f namespace.yaml
```

---

## Troubleshooting

- **Pods Not Starting**: Check the ReplicaSet logs and events using:
  ```sh
  kubectl describe replicaset caddy-replicaset -n caddy-namespace
  ```
- **Backend Services Unreachable**: Ensure backend services are running and accessible within the VNET.

---



