# kubectl Quick Reference

Essential commands for the CKA exam.

## Setup (Pre-configured in Exam)

```bash
alias k=kubectl
export do="--dry-run=client -o yaml"

# Usage
k run nginx --image=nginx $do > pod.yaml
```

## Context & Configuration

```bash
# View contexts
kubectl config get-contexts
kubectl config current-context

# Switch context
kubectl config use-context <context-name>

# Set namespace for context
kubectl config set-context --current --namespace=<namespace>
```

## Resource Creation (Imperative)

```bash
# Pod
kubectl run nginx --image=nginx
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# Deployment
kubectl create deployment nginx --image=nginx --replicas=3
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deploy.yaml

# Service
kubectl expose pod nginx --port=80 --target-port=80 --type=ClusterIP
kubectl expose deployment nginx --port=80 --type=NodePort

# ConfigMap
kubectl create configmap my-config --from-literal=key=value
kubectl create configmap my-config --from-file=config.txt

# Secret
kubectl create secret generic my-secret --from-literal=password=secret

# ServiceAccount
kubectl create serviceaccount my-sa

# Role & RoleBinding
kubectl create role pod-reader --verb=get,list,watch --resource=pods
kubectl create rolebinding read-pods --role=pod-reader --user=jane

# ClusterRole & ClusterRoleBinding
kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods
kubectl create clusterrolebinding read-pods --clusterrole=pod-reader --user=jane
```

## Resource Management

```bash
# Get resources
kubectl get pods -o wide
kubectl get pods --all-namespaces
kubectl get pods -l app=nginx
kubectl get pods --show-labels

# Describe (detailed info + events)
kubectl describe pod nginx
kubectl describe node node01

# Delete
kubectl delete pod nginx
kubectl delete pod nginx --force --grace-period=0

# Edit
kubectl edit deployment nginx

# Apply/Create
kubectl apply -f manifest.yaml
kubectl create -f manifest.yaml

# Scale
kubectl scale deployment nginx --replicas=5

# Update image
kubectl set image deployment/nginx nginx=nginx:1.22
```

## Debugging

```bash
# Logs
kubectl logs nginx
kubectl logs nginx -c container-name    # Multi-container
kubectl logs nginx --previous           # Previous instance
kubectl logs nginx -f                   # Follow

# Execute
kubectl exec -it nginx -- /bin/sh
kubectl exec nginx -- cat /etc/config

# Port forward
kubectl port-forward pod/nginx 8080:80

# Copy files
kubectl cp nginx:/path/file ./local-file
```

## Cluster Info

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes -o wide

# API resources
kubectl api-resources
kubectl api-versions

# Explain (documentation)
kubectl explain pod.spec.containers
kubectl explain deployment.spec.strategy
```

## Labels & Selectors

```bash
# Add label
kubectl label pod nginx env=prod

# Remove label
kubectl label pod nginx env-

# Select by label
kubectl get pods -l env=prod
kubectl get pods -l 'env in (prod,dev)'
```

## Taints & Tolerations

```bash
# Add taint
kubectl taint nodes node01 key=value:NoSchedule

# Remove taint
kubectl taint nodes node01 key=value:NoSchedule-

# View taints
kubectl describe node node01 | grep Taint
```

## Node Operations

```bash
# Cordon (mark unschedulable)
kubectl cordon node01

# Uncordon
kubectl uncordon node01

# Drain (evict pods)
kubectl drain node01 --ignore-daemonsets --force
```

## RBAC

```bash
# Check permissions
kubectl auth can-i create pods
kubectl auth can-i create pods --as jane
kubectl auth can-i create pods --as jane --namespace dev
```

## JSON Path

```bash
# Get specific field
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

# Get node IPs
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
```

## Sorting & Filtering

```bash
# Sort by field
kubectl get pods --sort-by=.metadata.creationTimestamp

# Filter by field
kubectl get pods --field-selector=status.phase=Running
```
