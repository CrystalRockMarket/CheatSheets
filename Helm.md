# Helm

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Helm Basics](#helm-basics)
4. [Chart Structure](#chart-structure)
5. [Managing Repositories](#managing-repositories)
6. [Installing and Upgrading](#installing-and-upgrading)
7. [Release Management](#release-management)
8. [Customizing Charts](#customizing-charts)
9. [Creating Charts](#creating-charts)
10. [Helm Plugins](#helm-plugins)
11. [Security and Best Practices](#security-and-best-practices)
12. [Troubleshooting](#troubleshooting)
13. [Quick Reference](#quick-reference)

---

## Introduction

Helm is the Kubernetes package manager. It helps you define, install, and upgrade complex Kubernetes applications using packages called "charts".

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         Helm Client                              │
│                     (helm CLI Commands)                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    ┌─────────────────────────────────────────────────────────┐  │
│    │                     Chart (.tgz)                         │  │
│    │  ┌─────────────────────────────────────────────────┐    │  │
│    │  │  Chart.yaml         # Package metadata          │    │  │
│    │  │  values.yaml        # Default config values     │    │  │
│    │  │  templates/         # Kubernetes manifests      │    │  │
│    │  │  charts/            # Sub-charts               │    │  │
│    │  │  README.md          # Documentation             │    │  │
│    │  │  LICENSE            # License                   │    │  │
│    │  └─────────────────────────────────────────────────┘    │  │
│    └─────────────────────────────────────────────────────────┘  │
│                            │                                     │
│                            ▼                                     │
│              ┌───────────────────────────┐                      │
│              │   Helm Release (Tiller)   │                      │
│              │   (In-Cluster)            │                      │
│              │   (Helm 2 only)           │                      │
│              └─────────────┬─────────────┘                      │
│                            │                                     │
│              Helm 3+:      ▼                                     │
│              ┌───────────────────────────┐                      │
│              │   Kubernetes API Server   │                      │
│              │    (Direct Integration)   │                      │
│              └─────────────┬─────────────┘                      │
│                            │                                     │
│         ┌──────────────────┼──────────────────┐                  │
│         ▼                  ▼                  ▼                  │
│   ┌──────────┐      ┌──────────┐      ┌──────────┐             │
│   │  ConfigMap│      │  Secret  │      │ Deployment│            │
│   │ (Release) │      │          │      │          │             │
│   └──────────┘      └──────────┘      └──────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

### Why Use Helm

- **Package Management**: Bundle Kubernetes manifests
- **Versioning**: Track releases and rollbacks
- **Templating**: DRY principle for manifests
- **Reusability**: Share charts across teams
- **Dependency Management**: Handle chart dependencies
- **Rollbacks**: Easy recovery from failures

### Helm 2 vs Helm 3

| Feature | Helm 2 | Helm 3 |
|---------|--------|--------|
| Tiller | Yes (in-cluster) | No |
| Release Storage | ConfigMaps | Secrets |
| Library Charts | Limited | Native Support |
| OCI Support | No | Yes |
| Default Namespace | Default | Current Context |

---

## Installation

### Linux

```bash
# Download binary
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Or download manually
curl -LO https://get.helm.sh/helm-v3.14.0-linux-amd64.tar.gz
tar -xzf helm-v3.14.0-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
rm -rf linux-amd64
```

### macOS

```bash
# Via Homebrew
brew install helm

# Download manually
curl -LO https://get.helm.sh/helm-v3.14.0-darwin-amd64.tar.gz
tar -xzf helm-v3.14.0-darwin-amd64.tar.gz
sudo mv darwin-amd64/helm /usr/local/bin/helm
```

### Windows

```bash
# Via Chocolatey
choco install kubernetes-helm

# Via Scoop
scoop install helm

# Download manually
curl -LO https://get.helm.sh/helm-v3.14.0-windows-amd64.zip
unzip helm-v3.14.0-windows-amd64.zip
```

### Verify Installation

```bash
helm version
helm version --short
```

### Shell Autocompletion

```bash
# Bash
echo 'source <(helm completion bash)' >> ~/.bashrc

# Zsh
echo 'source <(helm completion zsh)' >> ~/.zshrc

# Fish
helm completion fish > ~/.config/fish/completions/helm.fish
```

---

## Helm Basics

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Chart** | Package of pre-configured Kubernetes resources |
| **Release** | Instance of a chart running in a cluster |
| **Repository** | Collection of charts |
| **Config** | Configuration values for a release |
| **Template** | Go template files for manifests |

### First Commands

```bash
# View Helm version
helm version

# Get help
helm --help
helm <command> --help

# Search for charts
helm search repo nginx
helm search hub nginx

# List releases
helm list
helm list --all

# List releases in namespace
helm list -n production
```

### Release Lifecycle

```
helm install          →  Creates a new release
     ↓
helm upgrade          →  Updates release to new chart/version
     ↓
helm rollback         →  Reverts to previous version
     ↓
helm uninstall        →  Removes release from cluster
```

---

## Chart Structure

### Directory Layout

```
mychart/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration values
├── values.schema.json  # JSON schema for validation (optional)
├── templates/          # Template files
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── _helpers.tpl    # Template helpers
│   ├── NOTES.txt       # Post-install instructions
│   └── configmap.yaml
├── charts/             # Sub-charts (dependencies)
│   └── subchart/
├── crds/               # Custom Resource Definitions
│   └── crd.yaml
├── README.md           # Chart documentation
├── LICENSE             # License file
└── .helmignore         # Files to ignore
```

### Chart.yaml

```yaml
apiVersion: v2
name: myapp
description: A Helm chart for My Application
type: application
version: 1.0.0
appVersion: "1.0.0"
kubeVersion: ">=1.19.0"

keywords:
  - web
  - api

home: https://example.com/myapp
icon: https://example.com/logo.png

sources:
  - https://github.com/example/myapp

maintainers:
  - name: John Doe
    email: john@example.com
    url: https://john.example.com

dependencies:
  - name: postgresql
    version: "12.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
    tags:
      - database
    import-values:
      - child: ""
        parent: ""
    alias: db

  - name: redis
    version: "18.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
    alias: cache

annotations:
  category: Application
  licenses: Apache-2.0
```

### values.yaml

```yaml
# Default values for mychart
replicaCount: 2

image:
  repository: myapp
  tag: "1.0.0"
  pullPolicy: IfNotPresent
  pullSecrets:
    - name: registry-secret

imagePullSecrets:
  - name: regcred

nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  name: ""
  annotations: {}

podAnnotations: {}
podLabels: {}

podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000

securityContext:
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false

service:
  type: ClusterIP
  port: 80
  targetPort: http

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: myapp-tls
      hosts:
        - myapp.example.com

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80

nodeSelector: {}
tolerations: []
affinity: {}

topologySpreadConstraints: []

livenessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 10
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3

startupProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 0
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 30

postgresql:
  enabled: true
  postgresqlPassword: secret
  persistence:
    enabled: true

redis:
  enabled: true
  password: redis-secret
```

### .helmignore

```
.git
.gitignore
*.md
*.tmp
*.orig
*.rej
*.swp
.DS_Store
.editorconfig
.env
.env.*
.git/
.bowerrc
.eslintrc
.gulpfile.js
node_modules/
npm-debug.log
yarn-error.log
```

### Template Functions and Pipelines

```yaml
# values.yaml
fullname: my-app
environment: production
replicaCount: 3

# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          {{- with .Values.image.pullSecrets }}
          imagePullSecrets:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
          env:
            - name: ENV
              value: {{ .Values.environment | quote }}
            - name: REPLICAS
              value: {{ .Values.replicaCount | quote }}
```

---

## Managing Repositories

### Add Repository

```bash
# Add stable charts
helm repo add bitnami https://charts.bitnami.com/bitnami

# Add ingress-nginx
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Add prometheus-community
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Add custom/private repository
helm repo add myrepo https://charts.example.com
```

### Manage Repositories

```bash
# List added repositories
helm repo list

# Update repository index
helm repo update

# Update specific repo
helm repo update bitnami

# Search in repository
helm search repo bitnami/nginx

# Search with version constraint
helm search repo bitnami/nginx --versions

# Remove repository
helm repo remove bitnami

# Index local directory
helm repo index /path/to/charts --url https://charts.example.com

# Update index file
helm repo update
```

### Search Charts

```bash
# Search in local repos
helm search repo nginx
helm search repo bitnami/postgresql

# Search with version
helm search repo nginx --versions | head -10

# Search in Artifact Hub
helm search hub nginx

# Search Artifact Hub by publisher
helm search hub prometheus --hub-url https://artifacthub.io

# Search with regex
helm search repo "^nginx.*ingress$"
```

### OCI Registry Support

```bash
# Login to OCI registry
helm registry login registry.example.com

# Pull chart from OCI
helm pull oci://registry.example.com/charts/myapp --version 1.0.0

# Install directly from OCI
helm install myapp oci://registry.example.com/charts/myapp --version 1.0.0

# Push chart to OCI registry
helm chart save mychart/ registry.example.com/charts/mychart:1.0.0
helm chart push registry.example.com/charts/mychart:1.0.0
```

---

## Installing and Upgrading

### Install Charts

```bash
# Install from repository
helm install my-release bitnami/nginx

# Install with version
helm install my-release bitnami/nginx --version 15.0.0

# Install from specific repo
helm install my-release bitnami/nginx -n mynamespace --create-namespace

# Install from local chart
helm install my-release ./mychart

# Install from compressed chart
helm install my-release ./mychart-1.0.0.tgz

# Install with custom values
helm install my-release ./mychart -f values-production.yaml

# Install with set values
helm install my-release ./mychart \
  --set replicaCount=3 \
  --set image.tag=v2.0.0

# Dry-run install
helm install my-release ./mychart --dry-run --debug

# Install in debug mode (render templates only)
helm install my-release ./mychart --debug --dry-run
```

### Upgrade Charts

```bash
# Upgrade release
helm upgrade my-release bitnami/nginx

# Upgrade with values
helm upgrade my-release ./mychart -f values-prod.yaml

# Upgrade with set values
helm upgrade my-release ./mychart \
  --set replicaCount=5 \
  --set image.tag=v2.0.0

# Upgrade and reset values if failed
helm upgrade my-release ./mychart --reset-values

# Upgrade with wait for resources
helm upgrade my-release ./mychart --wait --timeout 5m

# Upgrade atomic (rollback on failure)
helm upgrade my-release ./mychart --atomic --timeout 5m

# Upgrade with timeout
helm upgrade my-release ./mychart --timeout 10m

# Upgrade to specific version
helm upgrade my-release bitnami/nginx --version 15.1.0
```

### Rollback

```bash
# Rollback to previous release
helm rollback my-release 1

# Rollback to specific revision
helm rollback my-release 3

# Rollback with timeout
helm rollback my-release 1 --timeout 5m

# Show rollback history
helm history my-release

# Get previous release values
helm get values my-release --revision 2
```

### Uninstall

```bash
# Uninstall release
helm uninstall my-release

# Uninstall with timeout
helm uninstall my-release --wait --timeout 5m

# Keep history after uninstall
helm uninstall my-release --keep-history

# Uninstall from specific namespace
helm uninstall my-release -n production
```

### Release Information

```bash
# List all releases
helm list

# List all releases in namespace
helm list -n production

# List releases with status
helm list --all -n production

# Filter by status
helm list --pending
helm list --failed
helm list --deployed
helm list --uninstalled

# Get release status
helm status my-release

# Get release history
helm history my-release

# Get release values
helm get values my-release

# Get all values (including computed)
helm get values my-release --all

# Get values for specific revision
helm get values my-release --revision 3

# Get release manifest
helm get manifest my-release

# Get hooks
helm get hooks my-release

# Get notes
helm get notes my-release
```

---

## Release Management

### Testing Releases

```bash
# Test release (runs test pods)
helm test my-release

# Test with cleanup
helm test my-release --cleanup

# Test specific pod
helm test my-release <test-pod-name>
```

### Pulling Charts

```bash
# Pull chart to local directory
helm pull bitnami/nginx

# Pull specific version
helm pull bitnami/nginx --version 15.0.0

# Pull and untar
helm pull bitnami/nginx --untar

# Pull to specific directory
helm pull bitnami/nginx --destination /tmp/charts
```

### Exporting Charts

```bash
# Export release to YAML
helm template my-release ./mychart > rendered.yaml

# Export with values
helm template my-release ./mychart -f values.yaml > rendered.yaml

# Template in namespace
helm template my-release ./mychart -n production > rendered.yaml
```

### Plugin Commands

```# List plugins
helm plugin list

# Install plugin
helm plugin install https://github.com/databus23/helm-diff

# Use diff plugin
helm diff upgrade my-release ./mychart

# Uninstall plugin
helm plugin uninstall helm-diff
```

---

## Customizing Charts

### Value Files

```bash
# Create custom values file
cat > values-prod.yaml <<EOF
replicaCount: 5
image:
  tag: "1.0.0-prod"
resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: 2000m
    memory: 2Gi
ingress:
  enabled: true
  hosts:
    - host: api.example.com
      paths:
        - path: /
          pathType: Prefix
postgresql:
  enabled: true
  postgresqlPassword: ${DB_PASSWORD}
EOF

# Install with custom values
helm install myapp ./mychart -f values-prod.yaml
```

### Set Values

```bash
# Set single value
helm install myapp ./mychart --set replicaCount=3

# Set multiple values
helm install myapp ./mychart \
  --set replicaCount=3 \
  --set image.tag=v2.0.0 \
  --set resources.limits.cpu=1000m

# Set nested value
helm install myapp ./mychart \
  --set "ingress.hosts[0].host=api.example.com"

# Set value with JSON path
helm install myapp ./mychart \
  --set-json 'ingress.hosts=[{"host": "api.example.com", "paths": [{"path": "/", "pathType": "Prefix"}]}]'

# Set value from file
helm install myapp ./mychart --set-file config.path=/path/to/config.yaml
```

### show Commands

```bash
# Show chart information
helm show chart bitnami/nginx

# Show values
helm show values bitnami/nginx

# Show readme
helm show readme bitnami/nginx

# Show all
helm show all bitnami/nginx

# Show crds
helm show crds bitnami/nginx
```

### lint Command

```bash
# Lint chart
helm lint ./mychart

# Lint with values
helm lint ./mychart -f values-prod.yaml

# Lint with strict mode
helm lint ./mychart --strict
```

---

## Creating Charts

### Create New Chart

```bash
# Create chart
helm create mychart

# Create chart in specific directory
helm create charts/myapp

# Create library chart
helm create mylibchart --type library

# Create chart from scratch
mkdir mychart
cd mychart
helm init --starter
```

### Chart.yaml Creation

```yaml
apiVersion: v2
name: mychart
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.0.0"
dependencies: []
```

### Template Helpers

```yaml
# templates/_helpers.tpl
{{/*
Expand the name of the chart.
*/}}
{{- define "mychart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "mychart.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Create chart name and version as used by the chart label.
*/}}
{{- define "mychart.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "mychart.labels" -}}
helm.sh/chart: {{ include "mychart.chart" . }}
{{ include "mychart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "mychart.selectorLabels" -}}
app.kubernetes.io/name: {{ include "mychart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Create the name of the service account to use
*/}}
{{- define "mychart.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "mychart.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}
```

### NOTES.txt

```txt
Thank you for installing {{ .Chart.Name }}.

Your release is named {{ .Release.Name }}.

To get the application URL, run these commands:

{{- if .Values.ingress.enabled }}
{{- range $host := .Values.ingress.hosts }}
  http{{ if $.Values.ingress.tls }}s{{ end }}://{{ $host.host }}
{{- end }}
{{- else if contains "NodePort" .Values.service.type }}
  export NODE_PORT=$(kubectl get --namespace {{ .Release.Namespace }} -o jsonpath="{.spec.ports[0].nodePort}" services {{ include "mychart.fullname" . }})
  echo http://$NODE_PORT
{{- else if contains "LoadBalancer" .Values.service.type }}
     NOTE: It may take a few minutes for the LoadBalancer IP to be available.
           You can watch the status by running 'kubectl get svc -w {{ include "mychart.fullname" . }}'
{{- end }}
```

### Package Chart

```bash
# Package chart
helm package ./mychart

# Package with version
helm package ./mychart --version 1.0.0

# Package and sign
helm package ./mychart --sign --key mykey --keyring ~/.gnupg/secring.gpg

# Verify package
helm verify mychart-1.0.0.tgz

# Sign package with cosign (recommended)
cosign sign-blob mychart-1.0.0.tgz --output-signature mychart-1.0.0.tgz.sig
cosign verify --key cosign.pub mychart-1.0.0.tgz
```

### Dependency Management

```bash
# Build dependencies
helm dependency build ./mychart

# Update dependencies
helm dependency update ./mychart

# List dependencies
helm dependency list ./mychart

# Build with lock file
helm dependency build ./mychart --verify
```

---

## Helm Plugins

### Popular Plugins

```bash
# helm-diff: Show differences
helm plugin install https://github.com/databus23/helm-diff

# helm-secrets: Handle secrets
helm plugin install https://github.com/jkroepke/helm-secrets

# helm-last: Get last release values
helm plugin install https://github.com/mstruebing/helm-last

# helm-pass: Password management
helm plugin install https://github.com/adamreese/helm-pass

# helm-unittest: Unit testing for charts
helm plugin install https://github.com/helm-unittest/helm-unittest

# helm-push: Push to registries
helm plugin install https://github.com/chartmuseum/helm-push

# helm-gcs: Google Cloud Storage
helm plugin install https://github.com/hayorov/helm-gcs

# helm-s3: AWS S3 storage
helm plugin install https://github.com/hypnoglow/helm-s3
```

### Plugin Commands

```bash
# List plugins
helm plugin list

# Update all plugins
helm plugin update

# Update specific plugin
helm plugin update diff

# Uninstall plugin
helm plugin uninstall diff
```

### Using helm-diff

```bash
# Show differences before upgrade
helm diff upgrade my-release ./mychart

# Show differences for specific values file
helm diff upgrade my-release ./mychart -f values-prod.yaml

# Show differences for previous revision
helm diff revision my-release 1 2

# Suppress confirmed values
helm diff upgrade my-release ./mychart --suppress-secrets
```

---

## Security and Best Practices

### Security Best Practices

```bash
# Use signed charts
helm verify mychart-1.0.0.tgz

# Verify with public key
helm verify --keyring path/to/pubring.gpg mychart-1.0.0.tgz

# Install with verification
helm install myapp mychart-1.0.0.tgz --verify

# Use secrets for sensitive data
# values.yaml
secret:
  apiKey: ${API_KEY}  # Set from environment

# templates/deployment.yaml
env:
  - name: API_KEY
    valueFrom:
      secretKeyRef:
        name: {{ include "mychart.fullname" . }}-secrets
        key: apiKey
```

### RBAC

```yaml
# ServiceAccount for Helm
apiVersion: v1
kind: ServiceAccount
metadata:
  name: helm-deployer
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: helm-deployer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: helm-deployer
    namespace: kube-system
```

### Install with ServiceAccount

```bash
# Create namespace
kubectl create namespace production

# Create service account
kubectl create serviceaccount helm -n production

# Create RBAC
kubectl create clusterrolebinding helm-binding --clusterrole=cluster-admin --serviceaccount=production:helm

# Install with service account
helm install myapp ./mychart -n production --service-account helm
```

### Best Practices Checklist

- Use semantic versioning for charts
- Pin image versions (avoid `latest`)
- Use resource requests and limits
- Implement health checks (liveness/readiness)
- Use security contexts
- Enable pod disruption budgets
- Use network policies
- Use secrets for sensitive data
- Test charts with `helm test`
- Sign and verify charts
- Use `.helmignore` properly
- Document chart in README.md
- Use library charts for shared templates

---

## Troubleshooting

### Debug Commands

```bash
# Dry-run to see rendered templates
helm install my-release ./mychart --dry-run --debug
helm template my-release ./mychart --debug

# Check release status
helm status my-release

# Get release history
helm history my-release

# Get release values
helm get values my-release --all

# Get manifest
helm get manifest my-release

# Check hooks
helm get hooks my-release

# Validate chart
helm lint ./mychart

# Check dependencies
helm dependency list ./mychart

# Update dependencies
helm dependency update ./mychart
```

### Common Issues

**Permission Denied**

```bash
# Check kubeconfig
kubectl config view

# Verify cluster access
kubectl cluster-info

# Check context
kubectl config current-context

# Switch context
kubectl config use-context my-cluster
```

**Release Name Already Exists**

```bash
# List releases
helm list

# Uninstall existing release
helm uninstall existing-release

# Use different release name
helm install my-release-new ./mychart
```

**Timeout During Install**

```bash
# Increase timeout
helm install my-release ./mychart --timeout 10m

# Use atomic mode
helm install my-release ./mychart --atomic

# Check resource status
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

**Chart Not Found**

```bash
# Update repositories
helm repo update

# Search for chart
helm search repo <chart-name>

# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
```

**Template Rendering Errors**

```bash
# Debug with dry-run
helm install my-release ./mychart --dry-run --debug

# Check syntax
helm template my-release ./mychart --validate

# Lint chart
helm lint ./mychart --strict
```

**Hook Failures**

```bash
# Check hook status
helm get hooks my-release

# View hook logs
kubectl get jobs -n <namespace>
kubectl logs -n <namespace> <job-name>

# Skip hooks on upgrade
helm upgrade my-release ./mychart --no-hooks
```

---

## Quick Reference

### Common Commands

| Command | Description |
|---------|-------------|
| `helm install <name> <chart>` | Install a chart |
| `helm upgrade <name> <chart>` | Upgrade a release |
| `helm rollback <name> <rev>` | Rollback release |
| `helm uninstall <name>` | Uninstall release |
| `helm list` | List releases |
| `helm status <name>` | Release status |
| `helm get values <name>` | Get release values |
| `helm get manifest <name>` | Get manifest |
| `helm history <name>` | Release history |
| `helm test <name>` | Run tests |

### Repository Commands

| Command | Description |
|---------|-------------|
| `helm repo add <name> <url>` | Add repository |
| `helm repo list` | List repositories |
| `helm repo update` | Update repositories |
| `helm repo remove <name>` | Remove repository |
| `helm search repo <term>` | Search repository |
| `helm search hub <term>` | Search Artifact Hub |

### Chart Commands

| Command | Description |
|---------|-------------|
| `helm create <name>` | Create chart |
| `helm package <chart>` | Package chart |
| `helm lint <chart>` | Lint chart |
| `helm template <name> <chart>` | Render templates |
| `helm pull <chart>` | Pull chart |
| `helm show <type> <chart>` | Show chart info |

### Flags

| Flag | Description |
|------|-------------|
| `-n, --namespace` | Namespace scope |
| `-f, --values` | Values file |
| `--set` | Set values |
| `--dry-run` | Dry run |
| `--wait` | Wait for resources |
| `--timeout` | Timeout duration |
| `--atomic` | Atomic install/upgrade |
| `--verify` | Verify chart |
| `--version` | Chart version |
| `--create-namespace` | Create namespace |

### Output Formats

| Format | Flag |
|--------|------|
| JSON | `-o json` |
| YAML | `-o yaml` |
| Wide | `-o wide` |
| JSONPath | `-o jsonpath='...'` |

### Template Functions

| Function | Description |
|----------|-------------|
| `{{ .Values.x }}` | Access values |
| `{{ .Release.Name }}` | Release name |
| `{{ .Chart.Name }}` | Chart name |
| `{{ .Files.Get }}` | Get file content |
| `{{ include "tmpl" . }}` | Include template |
| `{{ toYaml .Values.x }}` | Convert to YAML |
| `{{ tpl .Values.x . }}` | Template string |
| `{{ indent 2 "text" }}` | Indent text |
| `{{ nindent 2 "text" }}` | Indent with newline |

### State Values

| Value | Description |
|-------|-------------|
| `failed` | Release failed |
| `pending-install` | Installing |
| `pending-upgrade` | Upgrading |
| `pending-rollback` | Rolling back |
| `deployed` | Successfully deployed |
| `uninstalling` | Uninstalling |
| `superseded` | Old version |
| `uninstalled` | Uninstalled |
| `failed` | Failed |

---

## See Also

- [Helm Documentation](https://helm.sh/docs/)
- [Helm Chart Template Guide](https://helm.sh/docs/chart_template_guide/)
- [Helm Hub](https://artifacthub.io/)
- [Bitnami Charts](https://github.com/bitnami/charts)
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)
