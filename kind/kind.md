## Basic Setup

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane    # Master node (API server, scheduler, etcd)
  - role: worker           # Worker 1 (runs your pods)
  - role: worker           # Worker 2
  - role: worker           # Worker 3
```

```
kind create cluster
       ↓
1. Pulls kindest/node Docker image (contains K8s + containerd)
       ↓
2. Spins up 4 Docker containers (1 control-plane + 3 workers)
       ↓
3. Bootstraps control plane (kubeadm init)
       ↓
4. Joins workers to cluster (kubeadm join)
       ↓
5. Configures kubectl context automatically
```

## kind commands

```bash 
# List clusters
kind get clusters

# Delete cluster
kind delete cluster --name prod-sim

# Create again later
kind create cluster --name prod-sim --config=cluster.yaml

## Scheduling nodes to pods
# Here we have 3 worker nodes, lets say they are different in size(capacity, cpu, ram). We can spin up pods based on requiremnets to any of these nodes. 
# - First, we label the nodes 
kubectl label nodes node_name key=value
kubectl label nodes prod-sim-worker env=production
```

- Now when defining the pod config file, we mention nodeSelector in the spec section with this key-value pair.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: prod-only
spec:
  nodeSelector:
    env: production
  containers:
    - name: app
      image: nginx
```
