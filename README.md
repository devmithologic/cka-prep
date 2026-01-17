# CKA Exam Preparation Guide

A structured study guide for the **Certified Kubernetes Administrator (CKA)** exam, organized by official exam domains.

## Exam Overview

| Detail | Information |
|--------|-------------|
| **Duration** | 2 hours |
| **Format** | Performance-based (hands-on tasks) |
| **Passing Score** | 66% |
| **Kubernetes Version** | 1.31 (verify at [cncf.io/certification/cka](https://www.cncf.io/certification/cka)) |
| **Resources Allowed** | [kubernetes.io/docs](https://kubernetes.io/docs), [kubernetes.io/blog](https://kubernetes.io/blog), [helm.sh/docs](https://helm.sh/docs) |

## Exam Domains

Navigate directly to the domain you need:

| Domain | Weight | Description |
|--------|--------|-------------|
| [**Troubleshooting**](./01-troubleshooting/) | 30% | Cluster, node, component, and networking troubleshooting |
| [**Cluster Architecture**](./02-cluster-architecture/) | 25% | RBAC, kubeadm, HA control plane, Helm, CRDs |
| [**Services & Networking**](./03-services-networking/) | 20% | Services, Network Policies, Ingress, CoreDNS |
| [**Workloads & Scheduling**](./04-workloads-scheduling/) | 15% | Deployments, ConfigMaps, Secrets, Scheduling |
| [**Storage**](./05-storage/) | 10% | PV, PVC, StorageClasses, Volume types |

## Repository Structure

```
cka-prep/
├── README.md                      # You are here
├── 01-troubleshooting/            # 30% of exam
│   ├── README.md
│   └── labs/
├── 02-cluster-architecture/       # 25% of exam
│   ├── README.md
│   └── labs/
├── 03-services-networking/        # 20% of exam
│   ├── README.md
│   └── labs/
├── 04-workloads-scheduling/       # 15% of exam
│   ├── README.md
│   └── labs/
├── 05-storage/                    # 10% of exam
│   ├── README.md
│   └── labs/
└── cheatsheets/
    ├── kubectl-quick-reference.md
    └── exam-day-tips.md
```

## Quick Reference

### Exam Environment Setup

```bash
# These are typically pre-configured in the exam
alias k=kubectl
export do="--dry-run=client -o yaml"

# Usage
k run nginx --image=nginx $do > pod.yaml
```

### Essential Commands

```bash
# Generate YAML quickly
kubectl run nginx --image=nginx --dry-run=client -o yaml
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
kubectl expose pod nginx --port=80 --type=ClusterIP --dry-run=client -o yaml

# Get documentation during exam
kubectl explain pod.spec.containers
kubectl explain deployment.spec.strategy
```

### ETCD Backup (Memorize This)

```bash
ETCDCTL_API=3 etcdctl snapshot save /tmp/backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

## Resources

- [Kubernetes Official Documentation](https://kubernetes.io/docs/) - Allowed during exam
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Killer.sh Simulator](https://killer.sh/) - Included with exam registration

## Contributing

Found an error or want to improve the content? PRs are welcome.

## License

MIT - Use freely for your CKA preparation.
