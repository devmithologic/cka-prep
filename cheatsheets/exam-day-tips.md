# CKA Exam Day Tips

Practical advice for maximizing your score.

## Before the Exam

### Technical Setup
- [ ] Stable internet connection
- [ ] Clean desk, no papers/notes
- [ ] Government ID ready
- [ ] Browser check completed
- [ ] Camera and microphone working

### Mental Preparation
- [ ] Good night's sleep
- [ ] Light meal before exam
- [ ] Bathroom break before starting

## Exam Strategy

### Time Management (2 hours)

| Question Weight | Suggested Time |
|-----------------|----------------|
| 2-4% | 2-3 minutes |
| 5-7% | 5-7 minutes |
| 8%+ | 8-10 minutes |

**Rule of thumb:** If stuck for more than 5 minutes, flag and move on.

### Question Approach

1. **Read carefully** - Understand exactly what's asked
2. **Note the namespace** - Many questions specify a namespace
3. **Note the context** - Switch context if required
4. **Verify your work** - Quick `kubectl get` to confirm

## First Things to Do

```bash
# Set up aliases (usually pre-configured)
alias k=kubectl
export do="--dry-run=client -o yaml"

# Check current context
kubectl config get-contexts

# Verify cluster access
kubectl get nodes
```

## Time-Saving Techniques

### Use Imperative Commands

```bash
# Fast pod creation
k run nginx --image=nginx $do > pod.yaml

# Fast deployment
k create deploy nginx --image=nginx --replicas=3 $do > deploy.yaml

# Fast service
k expose pod nginx --port=80 --type=ClusterIP
```

### Use kubectl explain

```bash
# Don't memorize YAML - look it up
k explain pod.spec.containers.resources
k explain deployment.spec.strategy
```

### Bookmark These Docs Pages

- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Pod Specification](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

## Common Mistakes to Avoid

### 1. Wrong Namespace
```bash
# Always check/set namespace
kubectl config set-context --current --namespace=<namespace>

# Or use -n flag
kubectl get pods -n kube-system
```

### 2. Wrong Context
```bash
# Verify before each question
kubectl config current-context

# Switch if needed
kubectl config use-context <context>
```

### 3. Not Verifying Work
```bash
# Always verify after creating resources
kubectl get <resource> -o wide
kubectl describe <resource>
```

### 4. Forgetting to Save YAML Before Editing
```bash
# Backup before editing
kubectl get deployment nginx -o yaml > backup.yaml
```

## Difficult Question Types

### ETCD Backup/Restore
- Know the certificate paths
- Use `ETCDCTL_API=3`
- Remember to update data-dir after restore

### Cluster Upgrades
- Drain node first
- Upgrade kubeadm, then kubelet
- Uncordon after

### Network Policies
- Empty selector `{}` means all pods
- Missing `policyTypes` defaults based on rules
- Test with a temporary pod

### Troubleshooting
- Check events: `kubectl describe`
- Check logs: `kubectl logs` and `journalctl`
- Check component status in `kube-system`

## Quick Validation Commands

```bash
# Pod running?
kubectl get pods -o wide

# Service has endpoints?
kubectl get endpoints <service>

# Network connectivity?
kubectl run test --image=busybox:1.28 --rm -it -- wget -qO- <url>

# DNS working?
kubectl run test --image=busybox:1.28 --rm -it -- nslookup <service>
```

## If Things Go Wrong

### Pod Won't Start
```bash
kubectl describe pod <name>    # Check events
kubectl logs <name>            # Check logs
kubectl logs <name> --previous # Previous instance
```

### Can't Connect to Cluster
```bash
# Check kubeconfig
echo $KUBECONFIG
cat ~/.kube/config

# Check current context
kubectl config current-context
```

### Command Not Working
```bash
# Check syntax
kubectl explain <resource>

# Check API version
kubectl api-resources | grep <resource>
```

## Last 10 Minutes

- Review flagged questions
- Verify all answers are complete
- Don't leave questions blank - partial credit exists

## Remember

- **66% to pass** - You can miss some questions
- **Kubernetes docs are allowed** - Use them
- **Performance-based** - Working solutions matter, not perfect YAML
- **Partial credit** - Attempt everything
