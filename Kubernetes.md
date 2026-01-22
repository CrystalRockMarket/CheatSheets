# Kubernetes

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture](#architecture)
3. [Core Components](#core-components)
4. [Workload Resources](#workload-resources)
5. [Networking](#networking)
6. [Storage](#storage)
7. [Configuration](#configuration)
8. [Security](#security)
9. [Scaling and Autoscaling](#scaling-and-autoscaling)
10. [Scheduling](#scheduling)
11. [Cluster Management](#cluster-management)
12. [Troubleshooting](#troubleshooting)
13. [Quick Reference](#quick-reference)

---

## Introduction

Kubernetes (K8s) is an open-source container orchestration platform that automates deploying, scaling, and managing containerized applications.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Kubernetes Cluster                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                         Control Plane                                │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │    │
│  │  │   kube-     │  │   kube-     │  │   etcd      │  │   cloud-    │ │    │
│  │  │   apiserver │  │  scheduler  │  │ (Database)  │  │ controller  │ │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │    │
│  │         │                │                │               │         │    │
│  │         └────────────────┼────────────────┴───────────────┘         │    │
│  │                          │                                         │    │
│  │                   ┌──────┴──────┐                                 │    │
│  │                   │   etcd      │                                 │    │
│  │                   └─────────────┘                                 │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                    │                                         │
│                                    │                                         │
│         ┌──────────────────────────┼──────────────────────────┐              │
│         ▼                          ▼                          ▼              │
│  ┌─────────────┐           ┌─────────────┐           ┌─────────────┐        │
│  │    Node 1   │           │    Node 2   │           │    Node 3   │        │
│  │  (Worker)   │           │  (Worker)   │           │  (Worker)   │        │
│  ├─────────────┤           ├─────────────┤           ├─────────────┤        │
│  │  kubelet    │           │  kubelet    │           │  kubelet    │        │
│  │  kube-proxy │           │  kube-proxy │           │  kube-proxy │        │
│  │  containerd │           │  containerd │           │  containerd │        │
│  │  ┌───────┐  │           │  ┌───────┐  │           │  ┌───────┐  │        │
│  │  │ Pod A │  │           │  │ Pod C │  │           │  │ Pod E │  │        │
│  │  └───────┘  │           │  └───────┘  │           │  └───────┘  │        │
│  │  ┌───────┐  │           │  ┌───────┐  │           │  ┌───────┐  │        │
│  │  │ Pod B │  │           │  │ Pod D │  │           │  │ Pod F │  │        │
│  │  └───────┘  │           │  └───────┘  │           │  └───────┘  │        │
│  └─────────────┘           └─────────────┘           └─────────────┘        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Features

- **Automated Scheduling**: Schedule containers across nodes
- **Self-Healing**: Restart failed containers, replace nodes
- **Horizontal Scaling**: Scale applications automatically
- **Service Discovery**: Find services automatically
- **Load Balancing**: Distribute traffic
- **Rolling Updates**: Zero-downtime deployments
- **Secret Management**: Store sensitive data
- **Storage Orchestration**: Attach storage automatically

---

## Architecture

### Control Plane Components

| Component | Description | Default Port |
|-----------|-------------|--------------|
| kube-apiserver | API server for all operations | 6443 |
| etcd | Distributed key-value store | 2379-2380 |
| kube-scheduler | Schedules pods to nodes | None |
| kube-controller-manager | Runs controllers | None |
| cloud-controller-manager | Cloud-specific controllers | None |

### Node Components

| Component | Description |
|-----------|-------------|
| kubelet | Agent that ensures containers are running |
| kube-proxy | Network proxy and load balancer |
| container runtime | Runs containers (containerd, CRI-O) |

### Addons

| Addon | Description |
|-------|-------------|
| DNS | CoreDNS for service discovery |
| Web UI | Dashboard |
| Metrics Server | Resource metrics |
| Ingress Controller | HTTP/HTTPS routing |

### API Groups

| API Group | Path | Resources |
|-----------|------|-----------|
| Core | /api | pods, services, configmaps |
| Apps | /apis/apps | deployments, daemonsets, statefulsets |
| Batch | /apis/batch | jobs, cronjobs |
| Networking | /apis/networking.k8s.io | networkpolicies, ingresses |
| Rbac | /apis/rbac.authorization.k8s.io | roles, rolebindings |

---

## Core Components

### Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    version: v1
spec:
  containers:
    - name: nginx
      image: nginx:1.25
      ports:
        - containerPort: 80
          name: http
          protocol: TCP
      env:
        - name: NGINX_PORT
          value: "80"
      resources:
        requests:
          memory: "128Mi"
          cpu: "100m"
        limits:
          memory: "256Mi"
          cpu: "500m"
      livenessProbe:
        httpGet:
          path: /healthz
          port: http
        initialDelaySeconds: 10
        periodSeconds: 10
      readinessProbe:
        httpGet:
          path: /ready
          port: http
        initialDelaySeconds: 5
        periodSeconds: 5
  restartPolicy: Always
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: ClusterIP  # ClusterIP, NodePort, LoadBalancer, ExternalName
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: http
  clusterIP: 10.96.0.1  # Optional
  sessionAffinity: None
  publishNotReadyAddresses: false
```

### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # Key-value pairs
  DATABASE_URL: postgresql://db:5432/app
  CACHE_TYPE: redis
  LOG_LEVEL: info
  # File-like keys
  app.properties: |
    key1=value1
    key2=value2
  nginx.conf: |
    worker_processes auto;
    events {
        worker_connections 1024;
    }
```

### Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque  # Opaque, kubernetes.io/tls, kubernetes.io/basic-auth
data:
  # Base64 encoded values
  username: YWRtaW4=
  password: c2VjcmV0
  # TLS secret
  # type: kubernetes.io/tls
  # data:
  #   tls.crt: BASE64_CERT
  #   tls.key: BASE64_KEY
```

---

## Workload Resources

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  revisionHistoryLimit: 5
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 256Mi
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app: nginx
                topologyKey: kubernetes.io/hostname
```

### ReplicaSet

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
```

### StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: root-password
          volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: fast
        resources:
          requests:
            storage: 10Gi
```

### DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostNetwork: true
      hostPID: true
      containers:
        - name: node-exporter
          image: prom/node-exporter:latest
          ports:
            - containerPort: 9100
              hostPort: 9100
          securityContext:
            privileged: true
          volumeMounts:
            - name: proc
              mountPath: /host/proc
              readOnly: true
            - name: sys
              mountPath: /host/sys
              readOnly: true
      volumes:
        - name: proc
          hostPath:
            path: /proc
        - name: sys
          hostPath:
            path: /sys
```

### Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: process-data
spec:
  backoffLimit: 4
  ttlSecondsAfterFinished: 300
  template:
    spec:
      containers:
        - name: process
          image: myapp:latest
          command: ["python", "process.py"]
      restartPolicy: OnFailure
```

### CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  concurrencyPolicy: Forbid  # Forbid, Replace, Allow
  failedJobsHistoryLimit: 3
  successfulJobsHistoryLimit: 5
  suspend: false
  jobTemplate:
    spec:
      backoffLimit: 2
      template:
        spec:
          containers:
            - name: backup
              image: backup-tool:latest
              command: ["/backup.sh"]
              volumeMounts:
                - name: data
                  mountPath: /data
          restartPolicy: OnFailure
          volumes:
            - name: data
              persistentVolumeClaim:
                claimName: data-pvc
```

---

## Networking

### Service Types

| Type | Description | Use Case |
|------|-------------|----------|
| ClusterIP | Internal IP only | Microservices |
| NodePort | Port on each node | Development |
| LoadBalancer | External LB | Cloud deployments |
| ExternalName | CNAME record | External services |

### NetworkPolicy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-isolation
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
        - namespaceSelector:
            matchLabels:
              name: monitoring
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: database
      ports:
        - protocol: TCP
          port: 5432
    - to:
        - namespaceSelector: {}
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
```

### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - myapp.example.com
      secretName: myapp-tls
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /static
            pathType: Prefix
            backend:
              service:
                name: static-service
                port:
                  number: 80
```

### IngressClass

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
spec:
  controller: k8s.io/ingress-nginx
  parameters:
    scope: Namespace
    namespace: ingress-nginx
    kind: IngressParameters
```

---

## Storage

### PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    server: nfs.example.com
    path: /exports/pv1
```

### PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-webapp
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast
  selector:
    matchLabels:
      type: ssd
```

### StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
mountOptions:
  - debug
```

### Volume Types

| Volume Type | Description |
|-------------|-------------|
| emptyDir | Temporary storage on node |
| hostPath | Node filesystem path |
| configMap | ConfigMap data |
| secret | Secret data |
| downwardAPI | Pod metadata |
| persistentVolumeClaim | PVC reference |
| nfs | NFS share |
| cephfs | CephFS |
| glusterfs | GlusterFS |
| iscsi | iSCSI LUN |
| fc | Fibre Channel |
| azureDisk | Azure Disk |
| gcePersistentDisk | GCE PD |
| awsElasticBlockStore | AWS EBS |

---

## Configuration

### ResourceQuota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: production
spec:
  hard:
    pods: "100"
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    services: "10"
    secrets: "20"
    configmaps: "20"
    persistentvolumeclaims: "10"
```

### LimitRange

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: compute-limits
spec:
  limits:
    - type: Container
      min:
        cpu: 50m
        memory: 8Mi
      max:
        cpu: "2"
        memory: 1Gi
      default:
        cpu: 200m
        memory: 96Mi
      defaultRequest:
        cpu: 100m
        memory: 64Mi
    - type: Pod
      min:
        cpu: 50m
        memory: 8Mi
      max:
        cpu: "4"
        memory: 2Gi
```

### PodDisruptionBudget

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-pdb
spec:
  minAvailable: 2
  # maxUnavailable: 1
  selector:
    matchLabels:
      app: api
```

### HorizontalPodAutoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Percent
          value: 100
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
```

---

## Security

### SecurityContext

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    supplementalGroups: [1000]
  containers:
    - name: app
      image: myapp:latest
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
          add:
            - NET_BIND_SERVICE
        seccompProfile:
          type: RuntimeDefault
```

### PodSecurityStandards

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### PodSecurityPolicy (Deprecated in 1.21+)

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - configMap
    - emptyDir
    - persistentVolumeClaim
    - secret
  hostNetwork: false
  hostPID: false
  hostIPC: false
  runAsUser:
    rule: MustRunAsNonRoot
  runAsGroup:
    rule: MustRunAs
    ranges:
      - min: 1000
        max: 65535
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: MustRunAs
    ranges:
      - min: 1000
        max: 65535
  fsGroup:
    rule: MustRunAs
    ranges:
      - min: 1000
        max: 65535
```

### RBAC

```yaml
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list"]

---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: default
subjects:
  - kind: User
    name: jane
    apiGroup: rbac.authorization.k8s.io
  - kind: Group
    name: developers
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

---
# ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]

---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-reader-binding
subjects:
  - kind: ServiceAccount
    name: metrics-server
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

### ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
  namespace: default
automountServiceAccountToken: true
```

### Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
  ingress: []
  egress: []
```

---

## Scaling and Autoscaling

### Vertical Pod Autoscaler

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: api-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  updatePolicy:
    updateMode: Auto
  resourcePolicy:
    containerPolicies:
      - containerName: api
        minAllowed:
          cpu: 100m
          memory: 128Mi
        maxAllowed:
          cpu: 2000m
          memory: 2Gi
```

### Cluster Autoscaler

```yaml
apiVersion: clusterautoscaler.kubernetes.io/v1alpha1
kind: ClusterAutoscaler
metadata:
  name: default
spec:
  scaleDown:
    enabled: true
    delayAfterAdd: 10m
    delayAfterDelete: 10m
    delayAfterFailure: 3m
    unneededTime: 10m
```

---

## Scheduling

### Node Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values:
                  - us-east-1a
                  - us-east-1b
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          preference:
            matchExpressions:
              - key: instance-type
                operator: In
                values:
                  - t3.large
  containers:
    - name: with-affinity
      image: myapp:latest
```

### Pod Affinity/Anti-Affinity

```yaml
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchLabels:
              app: cache
          topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          podAffinityTerm:
            labelSelector:
              matchExpressions:
                - key: app
                  operator: In
                  values:
                    - api
            topologyKey: kubernetes.io/hostname
```

### Taints and Tolerations

```yaml
# Node with taint
apiVersion: v1
kind: Node
metadata:
  name: node-1
spec:
  taints:
    - key: dedicated
      value: database
      effect: NoSchedule
    - key: gpu
      value: "true"
      effect: NoExecute
      timeAdded: "2024-01-01T00:00:00Z"

# Pod with toleration
apiVersion: v1
kind: Pod
metadata:
  name: database
spec:
  tolerations:
    - key: dedicated
      operator: Equal
      value: database
      effect: NoSchedule
    - key: gpu
      operator: Exists
      effect: NoExecute
  containers:
    - name: database
      image: postgres:15
```

### Priority Classes

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
preemptionPolicy: PreemptLowerPriority
description: "High priority class for production workloads"

---
apiVersion: v1
kind: Pod
metadata:
  name: critical-app
spec:
  priorityClassName: high-priority
  containers:
    - name: critical
      image: myapp:latest
```

---

## Cluster Management

### Node Management

```bash
# Get nodes
kubectl get nodes

# Describe node
kubectl describe node worker-1

# Cordon node (unschedulable)
kubectl cordon worker-1

# Uncordon node
kubectl uncordon worker-1

# Drain node
kubectl drain worker-1 --ignore-daemonsets --delete-emptydir-data

# View node conditions
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{range .status.conditions[*]}{.type}{"="}{.status}{"\t"}{end}{"\n"}{end}'

# Check node resources
kubectl describe node worker-1 | grep -A5 "Allocated resources"
```

### Namespaces

```bash
# Create namespace
kubectl create namespace production

# Get namespaces
kubectl get namespaces

# Set default namespace
kubectl config set-context --current --namespace=production

# Describe namespace
kubectl describe namespace production
```

### Labels and Annotations

```bash
# Add label
kubectl label nodes worker-1 disktype=ssd

# Remove label
kubectl label nodes worker-1 disktype-

# Add annotation
kubectl annotate pod nginx description="Production nginx"

# Remove annotation
kubectl annotate pod nginx description-
```

### Events

```bash
# Get events
kubectl get events --sort-by='.lastTimestamp'

# Get events in namespace
kubectl get events -n production

# Watch events
kubectl get events --watch

# Get specific resource events
kubectl get events --field-selector involvedObject.name=nginx
```

---

## Troubleshooting

### Common Issues and Solutions

**Pod Not Starting**

```bash
# Check pod status
kubectl get pods

# Describe pod
kubectl describe pod <pod-name>

# Check logs
kubectl logs <pod-name>

# Check previous logs
kubectl logs <pod-name> --previous

# Check events
kubectl get events --sort-by='.lastTimestamp' | grep <pod-name>
```

**ImagePullBackOff**

```bash
# Check image name
kubectl describe pod <pod-name> | grep -A5 "ImagePull"

# Verify image exists
docker pull <image-name>

# Check image pull secrets
kubectl get secret -n <namespace>
```

**CrashLoopBackOff**

```bash
# Check logs
kubectl logs <pod-name> --previous

# Check exit code
kubectl describe pod <pod-name> | grep "State:"

# Check resources
kubectl top pod <pod-name>
```

**Pending Pods**

```bash
# Check why pod is pending
kubectl describe pod <pod-name> | grep -A10 "Events:"

# Check node resources
kubectl describe nodes | grep -A5 "Allocated resources"

# Check node conditions
kubectl get nodes
kubectl describe node <node-name>
```

**Service Not Accessible**

```bash
# Check service endpoints
kubectl get endpoints <service-name>

# Check service selector
kubectl describe service <service-name>

# Test service connectivity
kubectl run test --image=busybox:1.36 --rm -it --restart=Never -- wget -qO- http://<service>:<port>

# Check network policies
kubectl get networkpolicies -n <namespace>
```

### Debugging Tools

```bash
# Create debug container
kubectl debug -it <pod-name> --image=busybox:1.36 -- sh

# Copy from pod
kubectl cp <pod-name>:/path/to/file ./file

# Port forward
kubectl port-forward <pod-name> 8080:80

# Apply temporary network policy
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: debug
  namespace: default
spec:
  podSelector: {}
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: debug
EOF
```

### Useful Commands

```bash
# Get all resources
kubectl get all -n <namespace>

# JSONPath examples
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\n"}{end}'

# Top resources
kubectl top nodes
kubectl top pods -n <namespace>

# API resources
kubectl api-resources

# API versions
kubectl api-versions

# Explain resource
kubectl explain pod.spec.containers
```

---

## Quick Reference

### Resource Short Names

| Short Name | Full Name |
|------------|-----------|
| `po` | pods |
| `svc` | services |
| `deploy` | deployments |
| `rs` | replicasets |
| `sts` | statefulsets |
| `ds` | daemonsets |
| `job` | jobs |
| `cj` | cronjobs |
| `cm` | configmaps |
| `secret` | secrets |
| `ns` | namespaces |
| `no` | nodes |
| `pv` | persistentvolumes |
| `pvc` | persistentvolumeclaims |
| `sc` | storageclasses |
| `sa` | serviceaccounts |
| `role` | roles |
| `rb` | rolebindings |
| `cr` | clusterroles |
| `crb` | clusterrolebindings |
| `netpol` | networkpolicies |
| `ing` | ingresses |
| `hpa` | horizontalpodautoscalers |
| `pdb` | poddisruptionbudgets |

### Common Flags

| Flag | Description |
|------|-------------|
| `-n, --namespace` | Namespace scope |
| `-A, --all-namespaces` | All namespaces |
| `-l, --selector` | Label selector |
| `-o, --output` | Output format |
| `--dry-run` | Dry run |
| `-f, --filename` | File to apply |
| `--show-labels` | Show labels |
| `-w, --watch` | Watch changes |
| `--field-selector` | Field selector |
| `--sort-by` | Sort field |

### Output Formats

| Format | Flag |
|--------|------|
| JSON | `-o json` |
| YAML | `-o yaml` |
| Wide | `-o wide` |
| Name | `-o name` |
| Custom | `-o custom-columns` |
| JSONPath | `-o jsonpath` |

### Pod Phases

| Phase | Description |
|-------|-------------|
| `Pending` | Waiting for scheduling |
| `Running` | Running on node |
| `Succeeded` | Completed successfully |
| `Failed` | Container failed |
| `Unknown` | Status unknown |

### Container States

| State | Description |
|-------|-------------|
| `Waiting` | Waiting for container to start |
| `Running` | Container is running |
| `Terminated` | Container stopped |
| `Waiting` | Container waiting |

### Common Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 137 | OOMKilled (128 + 9) |
| 143 | Terminated (128 + 15) |

---

## See Also

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [kubectl Reference](https://kubernetes.io/docs/reference/kubectl/)
- [API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Production Best Practices](https://kubernetes.io/docs/setup/best-practices/)
