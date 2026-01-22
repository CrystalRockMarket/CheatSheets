# kubectl

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Context and Configuration](#context-and-configuration)
4. [Pod Management](#pod-management)
5. [Deployment Management](#deployment-management)
6. [Service and Ingress](#service-and-ingress)
7. [ConfigMap and Secret](#configmap-and-secret)
8. [Namespace Management](#namespace-management)
9. [Resource Management](#resource-management)
10. [Debugging and Troubleshooting](#debugging-and-troubleshooting)
11. [Advanced Commands](#advanced-commands)
12. [Quick Reference](#quick-reference)

---

## Introduction

kubectl is the command-line tool for interacting with Kubernetes clusters. It allows you to deploy applications, inspect and manage cluster resources, and view logs.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        kubectl Client                            │
│                         (Local CLI)                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    ┌─────────────────────────────────────────────────────────┐  │
│    │              kubeconfig (~/.kube/config)                 │  │
│    │  ┌─────────────────────────────────────────────────┐    │  │
│    │  │  clusters:                                       │    │  │
│    │  │    - name: production                            │    │  │
│    │  │      server: https://k8s-api.example.com         │    │  │
│    │  │      certificate-authority-data: BASE64_CERT     │    │  │
│    │  │                                                  │    │  │
│    │  │  contexts:                                       │    │  │
│    │  │    - name: prod-context                          │    │  │
│    │  │      cluster: production                         │    │  │
│    │  │      user: admin-user                            │    │  │
│    │  │                                                  │    │  │
│    │  │  users:                                          │    │  │
│    │  │    - name: admin-user                            │    │  │
│    │  │      token: OAUTH_TOKEN                          │    │  │
│    │  └─────────────────────────────────────────────────┘    │  │
│    └─────────────────────────────────────────────────────────┘  │
│                            │                                     │
│                            ▼                                     │
│              ┌───────────────────────────┐                      │
│              │   Kubernetes API Server   │                      │
│              │    (Control Plane)        │                      │
│              └─────────────┬─────────────┘                      │
│                            │                                     │
│         ┌──────────────────┼──────────────────┐                  │
│         ▼                  ▼                  ▼                  │
│   ┌──────────┐      ┌──────────┐      ┌──────────┐             │
│   │   etcd   │      │  Scheduler│     │Controller│             │
│   │ (Storage)│      │           │     │ Manager  │             │
│   └──────────┘      └──────────┘      └──────────┘             │
│                            │                                     │
│                            ▼                                     │
│              ┌───────────────────────────┐                      │
│              │    kubelet on Workers     │                      │
│              │  (Node Agents)            │                      │
│              └───────────────────────────┘                      │
│                            │                                     │
│         ┌──────────────────┼──────────────────┐                  │
│         ▼                  ▼                  ▼                  │
│   ┌──────────┐      ┌──────────┐      ┌──────────┐             │
│   │  Pod-1   │      │  Pod-2   │      │  Pod-3   │             │
│   └──────────┘      └──────────┘      └──────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

### Common Operations

- Deploy applications
- Inspect cluster resources
- Debug applications
- Manage resources (pods, services, deployments)
- Scale applications
- Update applications

---

## Installation

### Linux

```bash
# Direct download
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Install binary
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify
kubectl version --client
```

### macOS

```bash
# Via Homebrew
brew install kubectl

# Or direct download
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x kubectl
```

### Windows

```bash
# Via Chocolatey
choco install kubernetes-cli

# Or download directly
curl -LO https://dl.k8s.io/release/v1.28.0/bin/windows/amd64/kubectl.exe
```

### Shell Autocompletion

```bash
# Bash
echo 'source <(kubectl completion bash)' >> ~/.bashrc

# Zsh
echo 'source <(kubectl completion zsh)' >> ~/.zshrc

# Fish
kubectl completion fish > ~/.config/fish/completions/kubectl.fish
```

### Install kubectl as kubectl plugin (krew)

```bash
# Install krew
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/arm.*/arm64/')" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew.tar.gz" &&
  tar zxvf krew.tar.gz &&
  ./krew-"${OS}_${ARCH}" install krew
)

# Install plugins
kubectl krew install ctx
kubectl krew install ns
kubectl krew install neat
```

---

## Context and Configuration

### kubeconfig Structure

```yaml
# ~/.kube/config
apiVersion: v1
kind: Config
preferences: {}

clusters:
  - name: development
    cluster:
      server: https://127.0.0.1:6443
      certificate-authority-data: BASE64_ENCODED_CA
  - name: production
    cluster:
      server: https://prod-api.example.com
      certificate-authority: /path/to/ca.crt

contexts:
  - name: dev-context
    context:
      cluster: development
      user: dev-user
      namespace: default
  - name: prod-context
    context:
      cluster: production
      user: prod-user
      namespace: production

current-context: dev-context

users:
  - name: dev-user
    user:
      token: <YOUR_TOKEN_HERE>
  - name: prod-user
    user:
      username: admin
      password: secret
      client-certificate: /path/to/client.crt
      client-key: /path/to/client.key
```

### Context Commands

```bash
# List contexts
kubectl config get-contexts

# Show current context
kubectl config current-context

# Switch context
kubectl config use-context production

# Set current context namespace
kubectl config set-context --current --namespace=my-namespace

# Create new context
kubectl config set-context my-context \
  --cluster=production \
  --user=admin \
  --namespace=default

# Delete context
kubectl config delete-context my-context
```

### Cluster Commands

```bash
# List clusters
kubectl config get-clusters

# Add cluster
kubectl config set-cluster production \
  --server=https://api.example.com \
  --certificate-authority=/path/to/ca.crt

# View cluster info
kubectl cluster-info

# View cluster dump
kubectl cluster-info dump

# Dump to file
kubectl cluster-info dump --output-directory=/tmp/cluster-dump
```

### User Commands

```bash
# Set credentials
kubectl config set-credentials admin \
  --username=admin \
  --password=secret

# Set token
kubectl config set-credentials admin \
  --token=<Bearer Token>

# Set certificate
kubectl config set-credentials admin \
  --client-certificate=/path/to/client.crt \
  --client-key=/path/to/client.key

# View user config
kubectl config view --minify --output jsonpath='{.users[0]}'
```

### View and Merge Configs

```bash
# View full config
kubectl config view

# View minified (current context only)
kubectl config view --minify

# View as JSON
kubectl config view -o json

# View specific user
kubectl config view -o jsonpath='{.users[?(@.name == "admin")]}'

# Use specific kubeconfig file
kubectl --kubeconfig=/path/to/config get pods

# Set KUBECONFIG environment variable
export KUBECONFIG=/path/to/config:/path/to/other-config
```

---

## Pod Management

### Create Pods

```bash
# Create from YAML file
kubectl apply -f pod.yaml

# Create from YAML inline
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.25
EOF

# Generate YAML without applying
kubectl run nginx --image=nginx:1.25 --dry-run=client -o yaml

# Create and run a pod
kubectl run mypod --image=nginx --restart=Never

# Create with environment variables
kubectl run web --image=nginx:alpine \
  --env="NGINX_PORT=8080" \
  --env="HOSTNAME=$HOSTNAME"
```

### Get Pods

```bash
# List all pods in default namespace
kubectl get pods

# List pods in all namespaces
kubectl get pods -A
kubectl get pods --all-namespaces

# List pods with more details
kubectl get pods -o wide

# List pods with labels
kubectl get pods --show-labels

# List pods with specific labels
kubectl get pods -l app=nginx

# List pods in JSON format
kubectl get pods -o json

# List pods in YAML format
kubectl get pods -o yaml

# List pods with custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName

# Wide output with specific fields
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
```

### Describe Pods

```bash
# Detailed pod information
kubectl describe pod nginx

# Describe pod in specific namespace
kubectl describe pod nginx -n production

# Describe pods with label selector
kubectl describe pod -l app=web
```

### Delete Pods

```bash
# Delete pod
kubectl delete pod nginx

# Delete pod with grace period
kubectl delete pod nginx --grace-period=0 --force

# Delete pods from file
kubectl delete -f pod.yaml

# Delete pods by label
kubectl delete pod -l app=nginx

# Delete all pods in namespace
kubectl delete pods --all

# Delete all pods with wait
kubectl delete pods --all --wait
```

### Pod Logs

```bash
# Get logs
kubectl logs nginx

# Get logs from specific container
kubectl logs nginx -c nginx

# Follow logs
kubectl logs -f nginx

# Follow logs from specific container
kubectl logs -f nginx -c init-container

# Get last N lines
kubectl logs --tail=100 nginx

# Get logs since duration
kubectl logs --since=1h nginx

# Get logs since timestamp
kubectl logs --since-time=2024-01-01T00:00:00Z nginx

# Get previous container logs (after restart)
kubectl logs nginx --previous

# Get logs from all containers
kubectl logs nginx --all-containers=true

# Get logs from containers matching selector
kubectl logs -l app=nginx --all-containers=true
```

### Execute Commands

```bash
# Execute command in pod
kubectl exec nginx -- ls /usr/share/nginx/html

# Interactive shell
kubectl exec -it nginx -- /bin/sh

# Execute in specific container
kubectl exec nginx -c nginx -- /bin/sh

# Execute with specific user
kubectl exec nginx -- su - www-data

# Run single command
kubectl exec nginx -- printenv | grep NODE
```

### Copy Files

```bash
# Copy from pod to local
kubectl cp nginx:/var/log/nginx/access.log ./access.log

# Copy from local to pod
kubectl cp ./index.html nginx:/usr/share/nginx/html/

# Copy from specific container
kubectl cp nginx:/var/log/nginx/access.log ./access.log -c nginx

# Copy directory
kubectl cp nginx:/var/log ./logs
```

### Port Forward

```bash
# Forward local port to pod
kubectl port-forward nginx 8080:80

# Forward to random local port
kubectl port-forward nginx :80

# Forward to specific address
kubectl port-forward --address 0.0.0.0 nginx 8080:80

# Forward to multiple pods
kubectl port-forward pod/nginx 8080:80 &
kubectl port-forward pod/redis 6379:6379 &
```

### Attach to Running Pod

```bash
# Attach to pod
kubectl attach nginx -it

# Attach to specific container
kubectl attach nginx -c nginx -it
```

---

## Deployment Management

### Create Deployments

```bash
# Create deployment from file
kubectl apply -f deployment.yaml

# Generate deployment YAML
kubectl create deployment nginx --image=nginx:1.25 --dry-run=client -o yaml

# Create with replicas
kubectl create deployment nginx --image=nginx:1.25 --replicas=3

# Create with custom labels
kubectl create deployment web --image=nginx:alpine \
  --labels="app=web,env=production"

# Create with environment variables
kubectl create deployment api --image=node:20 \
  --env="NODE_ENV=production" \
  --env="API_KEY=secret"

# Create with port exposure
kubectl create deployment web --image=nginx:alpine --port=80

# Create with resource limits
kubectl create deployment api --image=node:20 \
  --requests=cpu=500m,memory=512Mi \
  --limits=cpu=1000m,memory=1Gi

# Create with volume mounts
kubectl create deployment db --image=postgres:15 \
  --env="POSTGRES_PASSWORD=secret" \
  --volume="pgdata:/var/lib/postgresql/data"
```

### Get Deployments

```bash
# List deployments
kubectl get deployments

# List deployments in all namespaces
kubectl get deployments -A

# Get deployment with status
kubectl get deployment nginx

# Wide output
kubectl get deployment -o wide

# Get deployment YAML
kubectl get deployment nginx -o yaml

# Get deployment JSON
kubectl get deployment nginx -o json

# Get deployment with custom columns
kubectl get deployment -o custom-columns=NAME:.metadata.name,READY:.status.readyReplicas,UP-TO-DATE:.status.updatedReplicas,AVAILABLE:.status.availableReplicas
```

### Describe Deployment

```bash
# Detailed deployment info
kubectl describe deployment nginx

# Show replica sets
kubectl describe deployment nginx | grep -A5 "ReplicaSets"
```

### Scale Deployments

```bash
# Scale deployment
kubectl scale deployment nginx --replicas=5

# Scale from file
kubectl scale -f deployment.yaml --replicas=3

# Scale by label selector
kubectl scale deployment -l app=web --replicas=10

# Scale with current replica count
kubectl scale deployment nginx --current-replicas=2 --replicas=5

# Autoscale deployment (Horizontal Pod Autoscaler)
kubectl autoscale deployment nginx --min=3 --max=10 --cpu-percent=80
kubectl create hpa nginx --min=3 --max=10 --cpu-percent=80
```

### Update Deployments

```bash
# Set image update
kubectl set image deployment/nginx nginx=nginx:1.26

# Set image for specific container
kubectl set image deployment/web web=nginx:alpine api=node:20

# Set environment variables
kubectl set env deployment nginx NGINX_PORT=8080

# Set resources
kubectl set resources deployment nginx \
  --requests=cpu=500m,memory=512Mi \
  --limits=cpu=1000m,memory=1Gi

# Set labels
kubectl set labels deployment nginx version=v2

# Set annotations
kubectl set annotations deployment nginx description="Updated deployment"
```

### Rollout Management

```bash
# Check rollout status
kubectl rollout status deployment/nginx

# View rollout history
kubectl rollout history deployment/nginx

# View specific revision
kubectl rollout history deployment/nginx --revision=3

# Undo last rollout
kubectl rollout undo deployment/nginx

# Undo to specific revision
kubectl rollout undo deployment/nginx --to-revision=2

# Pause rollout
kubectl rollout pause deployment/nginx

# Resume rollout
kubectl rollout resume deployment/nginx

# Restart deployment (creates new pods)
kubectl rollout restart deployment/nginx

# Check rollout status with timeout
kubectl rollout status deployment/nginx --timeout=5m
```

### Delete Deployments

```bash
# Delete deployment
kubectl delete deployment nginx

# Delete from file
kubectl delete -f deployment.yaml

# Delete by label
kubectl delete deployment -l app=nginx

# Delete all in namespace
kubectl delete deployment --all
```

---

## Service and Ingress

### Create Services

```bash
# Create service from file
kubectl apply -f service.yaml

# Create service exposing deployment
kubectl expose deployment nginx --port=80 --target-port=80

# Create service with type
kubectl expose deployment nginx --port=80 --type=ClusterIP
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Create service with specific node port
kubectl expose deployment nginx --port=80 --type=NodePort --node-port=30080

# Create service with external IP
kubectl expose deployment nginx --port=80 --type=LoadBalancer \
  --external-ip=203.0.113.10

# Generate service YAML
kubectl create service clusterip nginx --tcp=80:80 --dry-run=client -o yaml
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

### Get Services

```bash
# List services
kubectl get services

# List services in all namespaces
kubectl get services -A

# Get service details
kubectl get service nginx -o wide

# Get service endpoints
kubectl get endpoints nginx

# Get service as YAML
kubectl get service nginx -o yaml

# Describe service
kubectl describe service nginx
```

### Delete Services

```bash
# Delete service
kubectl delete service nginx

# Delete from file
kubectl delete -f service.yaml
```

### Ingress

```bash
# Create ingress from file
kubectl apply -f ingress.yaml

# Get ingress
kubectl get ingress

# Describe ingress
kubectl describe ingress nginx

# Delete ingress
kubectl delete ingress nginx
```

### Endpoint Operations

```bash
# Get endpoints
kubectl get endpoints

# Describe endpoint
kubectl describe endpoint nginx
```

---

## ConfigMap and Secret

### ConfigMap

```bash
# Create from file
kubectl create configmap app-config --from-file=config.properties

# Create from literal values
kubectl create configmap app-config \
  --from-literal=DEBUG=1 \
  --from-literal=LOG_LEVEL=info

# Create from env file
kubectl create configmap app-config --from-env-file=.env

# Create from directory
kubectl create configmap app-config --from-file=/path/to/config/

# Generate ConfigMap YAML
kubectl create configmap app-config \
  --from-literal=DEBUG=1 \
  --dry-run=client -o yaml

# Apply ConfigMap
kubectl apply -f configmap.yaml

# Describe ConfigMap
kubectl describe configmap app-config

# Get ConfigMap
kubectl get configmap app-config -o yaml

# Delete ConfigMap
kubectl delete configmap app-config
```

### Secret

```bash
# Create generic secret from file
kubectl create secret generic db-credentials \
  --from-file=username=./username.txt \
  --from-file=password=./password.txt

# Create generic secret from literals
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=secret123

# Create docker-registry secret
kubectl create secret docker-registry my-registry \
  --docker-server=https://registry.example.com \
  --docker-username=admin \
  --docker-password=secret \
  --docker-email=admin@example.com

# Create TLS secret
kubectl create secret tls tls-cert \
  --cert=tls.crt \
  --key=tls.key

# Generate secret YAML
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=secret \
  --dry-run=client -o yaml

# Describe secret
kubectl describe secret db-credentials

# Get secret (base64 encoded)
kubectl get secret db-credentials -o yaml

# Decode secret
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 -d

# Delete secret
kubectl delete secret db-credentials
```

### Use ConfigMap/Secret in Pod

```bash
# Create pod with env vars from ConfigMap
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: node:20
    envFrom:
    - configMapRef:
        name: app-config
    - secretRef:
        name: db-credentials
EOF

# Create pod with specific env vars
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: node:20
    env:
    - name: DEBUG
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DEBUG
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
EOF

# Create pod with ConfigMap volume
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: node:20
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
EOF
```

---

## Namespace Management

### Create Namespace

```bash
# Create namespace
kubectl create namespace production

# Create from file
kubectl apply -f namespace.yaml

# Generate namespace YAML
kubectl create namespace production --dry-run=client -o yaml
```

### Get Namespaces

```bash
# List namespaces
kubectl get namespaces

# Get namespace details
kubectl get namespace production

# Describe namespace
kubectl describe namespace production

# Get namespace as YAML
kubectl get namespace production -o yaml
```

### Set Default Namespace

```bash
# Using context
kubectl config set-context --current --namespace=production

# Using kubens (krew plugin)
kubectl ns production

# Using environment variable
export NAMESPACE=production
```

### Delete Namespace

```bash
# Delete namespace (deletes all resources in namespace)
kubectl delete namespace production

# Delete with wait
kubectl delete namespace production --wait
```

---

## Resource Management

### Get All Resources

```bash
# Get all pods in namespace
kubectl get all

# Get all resources in namespace
kubectl get all -n production

# Get resources by type
kubectl get pods,svc,deployments

# Get all with label selector
kubectl get all -l app=web

# Wide output
kubectl get all -o wide
```

### Label and Annotation Operations

```bash
# Set label
kubectl label pods nginx version=v2

# Overwrite label
kubectl label pods nginx version=v3 --overwrite

# Remove label
kubectl label pods nginx version-

# Set annotation
kubectl annotate pods nginx description="Production pod"

# Overwrite annotation
kubectl annotate pods nginx description="Updated description" --overwrite

# Remove annotation
kubectl annotate pods nginx description-

# Show labels
kubectl get pods --show-labels

# Filter by label
kubectl get pods -l version=v2

# JSONPath with labels
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.version}{"\n"}{end}'
```

### Delete Resources

```bash
# Delete by file
kubectl delete -f deployment.yaml

# Delete by type and name
kubectl delete deployment,service nginx

# Delete by label
kubectl delete pods -l app=nginx

# Delete by field selector
kubectl delete pods --field-selector status.phase=Running

# Delete all in namespace
kubectl delete all --all

# Delete with grace period
kubectl delete pod nginx --grace-period=30

# Force delete (stuck terminating)
kubectl delete pod nginx --grace-period=0 --force
```

### Apply and Patch

```bash
# Apply configuration
kubectl apply -f deployment.yaml

# Apply with dry-run
kubectl apply -f deployment.yaml --dry-run=client
kubectl apply -f deployment.yaml --dry-run=server

# Apply from stdin
kubectl apply -f -

# Apply with label selector
kubectl apply -f deployment.yaml -l app=web

# View diff before apply
kubectl apply -f deployment.yaml --dry-run=server -f -o yaml | kubectl diff -f -

# Patch resource
kubectl patch deployment nginx --type=json -p='[{"op": "replace", "path": "/spec/replicas", "value": 5}]'

# Strategic merge patch
kubectl patch deployment nginx --patch='{"spec": {"replicas": 5}}'

# Update container image
kubectl set image deployment/nginx nginx=nginx:1.26

# Update env var
kubectl set env deployment nginx DEBUG=0
```

### Edit Resource

```bash
# Edit resource in editor
kubectl edit deployment nginx

# Edit in specific namespace
kubectl edit deployment nginx -n production

# Edit as JSON
kubectl edit deployment nginx -o json
```

### Explain Resources

```bash
# Explain resource fields
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.spec.containers

# Explain with examples
kubectl explain pod --recursive
```

### Resource Quotas

```bash
# Get resource quotas
kubectl get resourcequotas

# Describe quota
kubectl describe resourcequota mem-limit

# Get quota usage
kubectl get resourcequota mem-limit -o yaml
```

### Limit Ranges

```bash
# Get limit ranges
kubectl get limitranges

# Describe limit range
kubectl describe limitrange default
```

---

## Debugging and Troubleshooting

### Pod Status Troubleshooting

```bash
# Check pod status
kubectl get pods

# Get pod events
kubectl describe pod nginx

# Check pod logs
kubectl logs nginx --all-containers=true

# Check previous pod logs
kubectl logs nginx --previous

# Execute into pod
kubectl exec -it nginx -- /bin/sh

# Port forward for debugging
kubectl port-forward nginx 8080:80
```

### Common Pod States

| State | Meaning | Action |
|-------|---------|--------|
| `Pending` | Waiting for scheduling | Check resources, node capacity |
| `Running` | Running | Check logs, health |
| `Succeeded` | Completed successfully | Normal for jobs |
| `Failed` | Container failed | Check exit code, logs |
| `Unknown` | Cannot communicate with pod | Check node, network |
| `CrashLoopBackOff` | Restarting repeatedly | Check logs, resources |

### Debug Commands

```bash
# Get events
kubectl get events --sort-by='.lastTimestamp'

# Get events in namespace
kubectl get events -n production

# Watch events
kubectl get events --watch

# Describe node
kubectl describe node worker-1

# Check node conditions
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{range .status.conditions[*]}{.type}{"="}{.status}{"\t"}{end}{"\n"}{end}'

# Check node resources
kubectl describe nodes | grep -A5 "Allocated resources"

# View controller logs
kubectl logs -n kube-system -l app=kube-controller-manager

# Check DNS
kubectl exec -it nginx -- nslookup kubernetes.default
kubectl exec -it nginx -- cat /etc/resolv.conf
```

### Profiling and Performance

```bash
# Top nodes
kubectl top nodes

# Top pods
kubectl top pods

# Top pods in namespace
kubectl top pods -n production

# Top pods with labels
kubectl top pods -l app=web
```

### Service Troubleshooting

```bash
# Check service endpoints
kubectl get endpoints nginx

# Describe service
kubectl describe service nginx

# Test service connectivity
kubectl run test --image=busybox:1.36 --rm -it --restart=Never -- wget -qO- http://nginx:80

# Check kube-proxy logs
kubectl logs -n kube-system -l k8s-app=kube-proxy

# Check service DNS
kubectl exec -it nginx -- nslookup service-name
```

### Network Policy

```bash
# List network policies
kubectl get networkpolicies

# Describe network policy
kubectl describe networkpolicy nginx-policy

# Check if policy allows traffic
kubectl auth can-i create pods --as=system:serviceaccount:default:default
```

### RBAC Troubleshooting

```bash
# Check permissions
kubectl auth can-i create pods
kubectl auth can-i create pods --as=system:serviceaccount:default:nginx

# List API groups
kubectl api-resources

# Check RBAC bindings
kubectl get rolebindings
kubectl get clusterrolebindings

# Describe role binding
kubectl describe rolebinding admin

# Check subjects in role binding
kubectl get rolebinding admin -o jsonpath='{.subjects}'
```

### Evict and Drain

```bash
# Drain node for maintenance
kubectl drain worker-1 --ignore-daemonsets --delete-emptydir-data

# Force drain
kubectl drain worker-1 --ignore-daemonsets --delete-emptydir-data --force

# Cordon node (unschedulable)
kubectl cordon worker-1

# Uncordon node
kubectl uncordon worker-1
```

### Certificates

```bash
# Check certificate expiration
kubectl get csr
kubectl certificate approve <csr-name>
kubectl certificate deny <csr-name>
```

---

## Advanced Commands

### kubectl Plugins

```bash
# List plugins
kubectl plugin list

# Use plugin
kubectl plugin context    # kubectx equivalent
kubectl plugin ns         # kubens equivalent
kubectl plugin neat       # Clean up YAML output

# Install via krew
kubectl krew install ctx
kubectl krew install ns
kubectl krew install neat
kubectl krew install deprecations
```

### Generate Resources

```bash
# Generate pod YAML
kubectl run nginx --image=nginx --dry-run=client -o yaml

# Generate deployment YAML
kubectl create deployment nginx --image=nginx --replicas=3 --dry-run=client -o yaml

# Generate service YAML
kubectl create service clusterip nginx --tcp=80:80 --dry-run=client -o yaml

# Generate configmap YAML
kubectl create configmap app-config --from-literal=DEBUG=1 --dry-run=client -o yaml

# Generate secret YAML
kubectl create secret generic creds --from-literal=user=admin --dry-run=client -o yaml

# Generate all for deployment
kubectl create deployment nginx --image=nginx --replicas=3 \
  --dry-run=client -o yaml > deployment.yaml
```

### Custom Columns

```bash
# Custom columns for pods
kubectl get pods -o custom-columns=NAME:metadata.name,STATUS:status.phase,IP:status.podIP

# With JSONPath
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\n"}{end}'

# Wide with specific fields
kubectl get pods -o 'custom-columns=POD:metadata.name,NODE:spec.nodeName,IMG:image'
```

### Sort and Filter

```bash
# Sort by name
kubectl get pods --sort-by=.metadata.name

# Sort by creation timestamp
kubectl get pods --sort-by=.metadata.creationTimestamp

# Sort by restart count
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# Filter by field
kubectl get pods --field-selector=status.phase=Running

# Filter by label
kubectl get pods -l version=v2
```

### JSONPath Examples

```bash
# Get all pod names
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

# Get pod names in namespace
kubectl get pods -n production -o jsonpath='{.items[*].metadata.name}'

# Get all node names
kubectl get nodes -o jsonpath='{.items[*].metadata.name}'

# Get node external IPs
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.addresses[?(@.type=="ExternalIP")].address}{"\n"}{end}'

# Get container images
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{": "}{range .spec.containers[*]}{.image}{", "}{end}{"\n"}{end}'
```

### kubectl Diff

```bash
# Show diff before apply
kubectl diff -f deployment.yaml

# Diff between current and file
kubectl diff

# Diff specific resource
kubectl diff deployment/nginx
```

### Wait for Resources

```bash
# Wait for pod to be ready
kubectl wait --for=condition=ready pod/nginx --timeout=60s

# Wait for pod to be deleted
kubectl wait --for=delete pod/nginx --timeout=60s

# Wait for any pod with label
kubectl wait --for=condition=ready pod -l app=web --timeout=120s

# Wait for multiple conditions
kubectl wait --for=jsonpath='{.status.phase}'=Running pod/nginx --timeout=60s
```

### Proxy and API Access

```bash
# Start API proxy
kubectl proxy

# Access API directly
curl http://localhost:8001/api/v1/namespaces/default/pods

# Port forward to API
kubectl port-forward svc/kubernetes 8443:443

# API versions available
kubectl api-versions

# API resources available
kubectl api-resources
```

---

## Quick Reference

### Common Commands

| Command | Description |
|---------|-------------|
| `kubectl get pods` | List pods |
| `kubectl get pods -o wide` | List pods with node info |
| `kubectl describe pod <name>` | Pod details |
| `kubectl logs <name>` | Get logs |
| `kubectl logs -f <name>` | Follow logs |
| `kubectl exec -it <name> -- sh` | Shell into pod |
| `kubectl apply -f <file>` | Apply config |
| `kubectl delete -f <file>` | Delete config |
| `kubectl get all` | Get all resources |
| `kubectl get events` | Get cluster events |
| `kubectl cluster-info` | Cluster information |
| `kubectl config view` | View kubeconfig |

### Short Names for Resources

| Short Name | Resource |
|------------|----------|
| `po` | pods |
| `deploy` | deployments |
| `rs` | replicasets |
| `svc` | services |
| `cm` | configmaps |
| `sec` | secrets |
| `ns` | namespaces |
| `node` | nodes |
| `ing` | ingresses |
| `hpa` | horizontalpodautoscalers |
| `pdb` | poddisruptionbudgets |
| `psp` | podsecuritypolicies |
| `sa` | serviceaccounts |
| `role` | roles |
| `clusterrole` | clusterroles |
| `rb` | rolebindings |
| `crb` | clusterrolebindings |

### Output Formats

| Format | Description |
|--------|-------------|
| `-o json` | Output as JSON |
| `-o yaml` | Output as YAML |
| `-o wide` | Additional columns |
| `-o name` | Resource names only |
| `-o jsonpath` | Custom output using JSONPath |
| `-o custom-columns` | Custom columns |

### Global Flags

| Flag | Description |
|------|-------------|
| `-n, --namespace` | Namespace scope |
| `-A, --all-namespaces` | All namespaces |
| `-l, --label` | Label selector |
| `-f, --filename` | File to apply |
| `--dry-run` | Dry run (client/server) |
| `-o, --output` | Output format |
| `-v, --v` | Verbosity level |

### Resource Status Conditions

| Condition | Meaning |
|-----------|---------|
| `Available` | Resource is available |
| `Ready` | Resource is ready to serve |
| `Scheduled` | Pod has been scheduled |
| `Initialized` | Containers initialized |
| `Ready` | Containers ready to serve |

### Common Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 124 | Command timed out |

---

## See Also

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [kubectl Reference](https://kubernetes.io/docs/reference/kubectl/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
