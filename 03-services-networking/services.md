# Services

Services enable communication between components within and outside of a Kubernetes cluster. They provide stable networking endpoints for accessing pods.

## Why Services?

Pods are ephemeral—they can be created, destroyed, and rescheduled at any time. Each pod gets a new IP address, making direct pod-to-pod communication unreliable.

```mermaid
flowchart TB
    subgraph Problem["The Problem"]
        P1["Pod<br/>10.244.0.2"] -->|"dies"| P2["New Pod<br/>10.244.0.7"]
    end
    
    subgraph Solution["The Solution"]
        SVC["Service<br/>Stable IP & Name"]
        PA["Pod A"]
        PB["Pod B"]
        PC["Pod C"]
        
        SVC --> PA
        SVC --> PB
        SVC --> PC
    end
```

**Services provide:**

- Stable IP address and DNS name
- Load balancing across pods
- Service discovery
- Loose coupling between microservices

---

## Service Types

```mermaid
flowchart TB
    subgraph Types["Service Types"]
        NP["NodePort<br/>External access via node port"]
        CIP["ClusterIP<br/>Internal cluster communication"]
        LB["LoadBalancer<br/>Cloud provider load balancer"]
    end
```

| Type | Use Case | Accessible From |
|------|----------|-----------------|
| **ClusterIP** | Internal communication between services | Within cluster only |
| **NodePort** | External access for development/testing | Outside cluster via `<NodeIP>:<NodePort>` |
| **LoadBalancer** | Production external access | External via cloud load balancer |

---

## NodePort

NodePort exposes the service on a static port on each node's IP, allowing external traffic to reach the service.

### Three Ports Involved

```mermaid
flowchart LR
    EXT["External User<br/>192.168.1.10"] -->|"NodePort<br/>30008"| NODE["Node<br/>192.168.1.2"]
    NODE -->|"Port<br/>80"| SVC["Service<br/>ClusterIP"]
    SVC -->|"TargetPort<br/>80"| POD["Pod<br/>10.244.0.2"]
```

| Port | Description | Example |
|------|-------------|---------|
| **NodePort** | Port on the node (external access) | 30008 |
| **Port** | Port on the Service | 80 |
| **TargetPort** | Port on the Pod where app runs | 80 |

> 📝 **NodePort range:** 30000 - 32767

### NodePort Definition

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - targetPort: 80    # Port on Pod (optional, defaults to port)
    port: 80          # Port on Service (required)
    nodePort: 30008   # Port on Node (optional, auto-assigned if omitted)
```

### Port Field Rules

| Field | Required | Default |
|-------|----------|---------|
| `port` | **Yes** | - |
| `targetPort` | No | Same as `port` |
| `nodePort` | No | Auto-assigned (30000-32767) |

### Multi-Pod Load Balancing

When multiple pods match the selector, the service automatically load balances using a **random algorithm**:

```mermaid
flowchart TB
    SVC["Service<br/>myapp-service"]
    
    P1["Pod 1<br/>app: myapp"]
    P2["Pod 2<br/>app: myapp"]
    P3["Pod 3<br/>app: myapp"]
    
    SVC -->|"random"| P1
    SVC -->|"random"| P2
    SVC -->|"random"| P3
```

### Multi-Node Behavior

NodePort service **automatically spans all nodes** in the cluster:

```mermaid
flowchart TB
    subgraph Cluster["Kubernetes Cluster"]
        subgraph N1["Node 1 - 192.168.1.2"]
            NP1["NodePort: 30008"]
            P1["Pod"]
        end
        subgraph N2["Node 2 - 192.168.1.3"]
            NP2["NodePort: 30008"]
            P2["Pod"]
        end
        subgraph N3["Node 3 - 192.168.1.4"]
            NP3["NodePort: 30008"]
        end
    end
    
    USER["User"] -->|"any node IP:30008"| NP1
    USER -->|"any node IP:30008"| NP2
    USER -->|"any node IP:30008"| NP3
```

Access your app via **any node's IP**, even if the pod isn't running on that node:

```bash
curl http://192.168.1.2:30008
curl http://192.168.1.3:30008
curl http://192.168.1.4:30008   # Works even without a pod on this node
```

---

## ClusterIP

ClusterIP is the **default** service type. It creates a virtual IP inside the cluster for internal communication.

### Use Case: Microservices

```mermaid
flowchart LR
    subgraph Frontend
        FE1["Frontend Pod"]
        FE2["Frontend Pod"]
    end
    
    subgraph Backend
        BE_SVC["backend-service<br/>ClusterIP"]
        BE1["Backend Pod"]
        BE2["Backend Pod"]
        BE3["Backend Pod"]
    end
    
    subgraph Database
        DB_SVC["db-service<br/>ClusterIP"]
        DB1["MySQL Pod"]
    end
    
    FE1 --> BE_SVC
    FE2 --> BE_SVC
    BE_SVC --> BE1
    BE_SVC --> BE2
    BE_SVC --> BE3
    
    BE1 --> DB_SVC
    BE2 --> DB_SVC
    BE3 --> DB_SVC
    DB_SVC --> DB1
```

### ClusterIP Definition

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP    # Optional - ClusterIP is the default
  selector:
    app: backend
  ports:
  - targetPort: 80
    port: 80
```

### Accessing ClusterIP Services

Other pods can access the service by:

```bash
# By ClusterIP
curl http://10.96.0.10:80

# By service name (same namespace)
curl http://backend-service:80

# By FQDN (cross-namespace)
curl http://backend-service.default.svc.cluster.local:80
```

---

## LoadBalancer

LoadBalancer provisions an external load balancer from the cloud provider (AWS, GCP, Azure).

### The Problem with NodePort

```mermaid
flowchart TB
    subgraph NodePort["NodePort: Multiple URLs"]
        U1["http://192.168.1.2:30008"]
        U2["http://192.168.1.3:30008"]
        U3["http://192.168.1.4:30008"]
        U4["http://192.168.1.5:30008"]
    end
    
    USER["User: Which URL do I use?"]
    USER -.-> U1
    USER -.-> U2
    USER -.-> U3
    USER -.-> U4
```

Users want a **single URL** like `https://myapp.example.com`

### The Solution

```mermaid
flowchart TB
    USER["User"] -->|"myapp.example.com"| LB["Cloud Load Balancer"]
    
    LB --> N1["Node 1:30008"]
    LB --> N2["Node 2:30008"]
    LB --> N3["Node 3:30008"]
    
    N1 --> P1["Pod"]
    N2 --> P2["Pod"]
    N3 --> P3["Pod"]
```

### LoadBalancer Definition

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - targetPort: 80
    port: 80
    nodePort: 30008    # Optional
```

### Important Notes

| Environment | Behavior |
|-------------|----------|
| **Cloud (AWS, GCP, Azure)** | Provisions real load balancer |
| **On-premises / VirtualBox** | Falls back to NodePort behavior |

> ⚠️ LoadBalancer only works with supported cloud providers. In unsupported environments, it behaves like NodePort.

---

## Labels and Selectors

Services find pods using **labels and selectors**:

```yaml
# Pod definition
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp        # Label
spec:
  containers:
  - name: nginx
    image: nginx

---
# Service definition
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp        # Selector matches pod label
  ports:
  - port: 80
    targetPort: 80
```

---

## Essential Commands

### Create Service

```bash
# From YAML
kubectl create -f service.yaml
kubectl apply -f service.yaml

# Imperative - expose a pod
kubectl expose pod nginx --port=80 --target-port=80 --type=NodePort

# Imperative - expose a deployment
kubectl expose deployment nginx --port=80 --type=ClusterIP
```

### View Services

```bash
# List services
kubectl get services
kubectl get svc              # short form

# Detailed info
kubectl describe svc myapp-service

# Check endpoints (pods backing the service)
kubectl get endpoints myapp-service
```

### Test Connectivity

```bash
# From within cluster
kubectl run test --image=busybox:1.28 --rm -it -- wget -qO- http://myapp-service:80

# From node (NodePort)
curl http://<node-ip>:30008
```

---

## Quick Reference

| Service Type | Port Range | Use Case |
|--------------|------------|----------|
| ClusterIP | Any | Internal communication |
| NodePort | 30000-32767 | External access (dev/test) |
| LoadBalancer | Any | Production external access |

---

## Key Takeaways

1. **Services provide stable endpoints** for ephemeral pods
2. **ClusterIP** is default - for internal cluster communication
3. **NodePort** exposes on all nodes, even without pods on that node
4. **LoadBalancer** requires cloud provider support
5. **Labels and selectors** link services to pods
6. Services automatically **load balance** across matching pods
7. **No additional config needed** when pods are added/removed

---

[Back to 03-services-networking](README.md)

[Back to root directory](../README.md)
