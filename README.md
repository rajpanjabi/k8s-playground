# Kubernetes Playground

This project is a hands-on Kubernetes playground designed to demonstrate and teach key features of Kubernetes through practical examples. It includes a simple web application, a Redis database, and various storage configurations to showcase different Kubernetes concepts.

## 🏗️ Architecture

The playground consists of:
- **API Service**: A simple web application deployed as a Kubernetes Deployment
- **Redis Database**: A stateful database using StatefulSet with persistent storage
- **Storage Examples**: Both local and cloud-based storage configurations

## 🎯 Kubernetes Features Demonstrated

### Core Concepts
- **Namespaces**: Resource isolation and organization (`demo-namespace`)
- **Pods**: Basic unit of deployment (managed by Deployments and StatefulSets)
- **Labels and Selectors**: Resource grouping and selection

### Workloads
- **Deployments**: Managing stateless applications with rolling updates and scaling
- **StatefulSets**: Managing stateful applications with stable network identities and persistent storage
- **Services**: Load balancing and service discovery between pods

### Configuration Management
- **ConfigMaps**: Storing non-sensitive configuration data
- **Secrets**: Managing sensitive information like passwords

### Networking
- **Ingress**: External access to services with host-based routing
- **Services**: Internal communication between pods

### Storage
- **Persistent Volumes (PV)**: Abstracting storage details
- **Persistent Volume Claims (PVC)**: Requesting storage resources
- **Storage Classes**: Dynamic provisioning of storage

## 📁 Project Structure

```
k8s-playground/
├── README.md
├── minikube/
│   ├── namespace.yaml           # Creates demo-namespace
│   ├── configMap.yaml           # Shared config for the app
│   ├── secret.yaml              # Redis password secret
│   ├── ingress.yaml             # External routing for the API
│   ├── api/
│   │   ├── Deployment.yaml      # API Deployment
│   │   └── service.yaml         # API ClusterIP Service
│   ├── redis/
│   │   ├── statefulset.yaml     # Redis StatefulSet
│   │   └── service.yaml         # Headless Redis Service
│   └── storage/
│       ├── cloud/
│       │   ├── storage-class.yaml
│       │   ├── pvc.yaml
│       │   └── pod.yaml
│       └── local/
│           ├── manifest-local.yaml
│           └── local-storage.md
└── kind/
    ├── kcluster.yaml            # kind cluster definition
    ├── kind.md                  # kind notes and examples
    ├── helm.md                  # Helm notes
    └── first_helm_k8s/
        ├── Chart.yaml           # Custom Helm chart metadata
        ├── values.yaml          # Default chart values
        ├── templates/
        │   ├── deployment.yaml
        │   └── service.yaml
        └── wordpress/           # Pulled WordPress chart with dependencies
```

## 🚀 Getting Started

### Prerequisites
- [minikube](https://minikube.sigs.k8s.io/docs/start/) installed and running
- [kubectl](https://kubernetes.io/docs/tasks/tools/) configured
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/) (for ingress functionality)

### Deployment

1. **Start minikube** (if not already running):
   ```bash
   minikube start
   ```

2. **Enable ingress addon**:
   ```bash
   minikube addons enable ingress
   ```

3. **Apply all manifests**:
   ```bash
   kubectl apply -f namespace.yaml
   kubectl apply -f configMap.yaml
   kubectl apply -f secret.yaml
   kubectl apply -f api/
   kubectl apply -f redis/
   kubectl apply -f storage/
   ```

4. **Start the tunnel** for ingress access:
   ```bash
   minikube tunnel
   ```
   This runs in the background and requires sudo/admin privileges.

5. **Access the application**:
   ```bash
   curl -H "Host: myapp.local" http://localhost:8080
   ```

   You should see a welcome message from the hello-kubernetes application.

## 🛠️ Useful Commands

```bash
# Check all resources
kubectl get all -n demo-namespace

# View logs
kubectl logs -l app=api -n demo-namespace
kubectl logs redis-0 -n demo-namespace

# Describe resources for troubleshooting
kubectl describe deployment api -n demo-namespace
kubectl describe statefulset redis -n demo-namespace

# Clean up
kubectl delete namespace demo-namespace
```

## 📚 Key Takeaways

1. **Namespaces** provide isolation and organization
2. **Deployments** handle stateless apps with easy scaling
3. **StatefulSets** maintain identity and storage for stateful apps
4. **ConfigMaps/Secrets** separate configuration from code
5. **Services** enable reliable pod-to-pod communication
6. **Ingress** routes external traffic to internal services
7. **Persistent storage** ensures data survives pod restarts
8. **Storage Classes** abstract underlying storage infrastructure

## 🔧 Customization

- Modify replica counts in Deployments/StatefulSets
- Change environment variables in ConfigMap
- Experiment with different storage backends
- Add health checks and resource limits
- Implement rolling updates and rollbacks

This playground provides a solid foundation for understanding Kubernetes concepts. Experiment freely and explore the official [Kubernetes documentation](https://kubernetes.io/docs/) for deeper learning!</content>
<parameter name="filePath">/Users/rajpanjabi/dev/k8s-playground/README.md
