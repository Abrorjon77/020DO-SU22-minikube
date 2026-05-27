# 020DO-SU22 — Minikube Labs

A hands-on lab repository for learning Kubernetes using [minikube](https://github.com/kubernetes/minikube). Each lab deploys a real-world tool and teaches core Kubernetes concepts.

**What you will learn:**
- [Kubernetes](https://kubernetes.io/) objects: [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Services](https://kubernetes.io/docs/concepts/services-networking/service/), [Ingresses](https://kubernetes.io/docs/concepts/services-networking/ingress/), [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/), [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/), [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
- [MongoDB](https://www.mongodb.com/) — database deployment with secrets and config
- [Redis](https://redis.io/) — volume-mounted ConfigMap configuration
- [Prometheus](https://prometheus.io/) + [Grafana](https://grafana.com/) — monitoring and dashboards via [Helm](https://helm.sh/)
- [cert-manager](https://cert-manager.io/) — TLS certificate management

---

## Directory Structure

```
.
├── mongo-webapp/
│   ├── mongo-cm.yaml       # ConfigMap for MongoDB
│   ├── mongo-secret.yaml   # Secret (credentials)
│   ├── mongo.yaml          # MongoDB Deployment + Service
│   └── webapp.yaml         # Web app Deployment + Service
├── redis-server/
│   ├── redis-cm.yaml       # ConfigMap with Redis config
│   ├── redis-deploy.yaml   # Redis Deployment
│   └── redis-pod.yaml      # Standalone Redis Pod
├── prometheus-grafana/
│   └── helm-tutorial/      # Helm-based Prometheus + Grafana setup
├── cert-manager/           # TLS certificate management lab
└── README.md
```

---

## Prerequisites

Install the required tools before starting any lab:

| Tool | Install |
|------|---------|
| [minikube](https://minikube.sigs.k8s.io/docs/start/) | `brew install minikube` (macOS) |
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | `brew install kubectl` (macOS) |
| [Helm](https://helm.sh/docs/intro/install/) | `brew install helm` (macOS) |

### Minikube Memory Requirements

No memory limits are defined in the lab manifests, but each lab has real-world resource needs:

| Lab | Memory needed |
|-----|--------------|
| MongoDB + webapp | ~512 MB |
| Redis | ~128 MB |
| Prometheus + Grafana | ~1.5 GB (heaviest) |
| cert-manager | ~256 MB |

Start minikube with enough memory for what you plan to run:

```bash
# Minimum — individual labs only
minikube start --memory=2048 --cpus=2

# Recommended — running Prometheus + Grafana or multiple labs at once
minikube start --memory=4096 --cpus=2
```

Enable the metrics-server addon to monitor resource usage:

```bash
minikube addons enable metrics-server

# Then check usage
kubectl top node
kubectl top pods
```

Verify minikube is running:

---

## Lab 1 — MongoDB Web App

**Concepts:** Deployments, Services, ConfigMaps, Secrets, Ingress

Enable the ingress addon:

```bash
minikube addons enable ingress
```

Deploy:

```bash
cd mongo-webapp/

# 1. Apply ConfigMap and Secret
kubectl apply -f mongo-cm.yaml -f mongo-secret.yaml

# 2. Deploy MongoDB
kubectl apply -f mongo.yaml

# 3. Deploy the web application
kubectl apply -f webapp.yaml

# 4. Check everything is running
kubectl get pods,services
```

Clean up:

```bash
kubectl delete deployment.apps/mongo deployment.apps/webapp \
  service/mongo-service service/webapp-service
```

---

## Lab 2 — Redis with ConfigMap

**Concepts:** Volumes, ConfigMaps, rolling restarts

Deploy:

```bash
cd redis-server/

kubectl apply -f redis-cm.yaml
kubectl apply -f redis-deploy.yaml

# Wait for the pod to be ready
kubectl get pod -l app=redis
```

Verify the default Redis config:

```bash
# Get the pod name
kubectl get pod -l app=redis

# Open a Redis shell
kubectl exec -it <pod-name> -- redis-cli

127.0.0.1:6379> CONFIG GET maxmemory
# Returns: 0 (default)

127.0.0.1:6379> CONFIG GET maxmemory-policy
# Returns: noeviction (default)
```

Update the ConfigMap (edit `redis-cm.yaml` to set values):

```yaml
data:
  redis-config: |
    maxmemory 2mb
    maxmemory-policy allkeys-lru
```

Apply the updated config and restart the pod to pick up changes:

```bash
kubectl apply -f redis-cm.yaml
kubectl rollout restart deploy redis

# Verify the new config is active
kubectl exec -it <pod-name> -- redis-cli
127.0.0.1:6379> CONFIG GET maxmemory
# Returns: 2097152
```

Clean up:

```bash
kubectl delete deployment/redis configmap/example-redis-config
```

---

## Lab 3 — Prometheus + Grafana (Helm)

**Concepts:** Helm charts, monitoring, dashboards

### Install Prometheus

```bash
# Add the Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus
helm install prometheus prometheus-community/prometheus

# Expose Prometheus and open in browser
kubectl expose service prometheus-server \
  --type=NodePort --target-port=9090 --name=prometheus-server-np

minikube service prometheus-server-np
```

### Install Grafana

```bash
# Add the Helm repo
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install Grafana
helm install grafana grafana/grafana

# Get the admin password
kubectl get secret grafana -o jsonpath="{.data.admin-password}" | base64 --decode; echo

# Expose Grafana and open in browser
kubectl expose service grafana \
  --type=NodePort --target-port=3000 --name=grafana-np

minikube service grafana-np
```

Login with username `admin` and the password from the command above.

### Configure Grafana

1. Go to **Configuration > Data Sources** and add a new Prometheus source.
2. Set the URL to `http://prometheus-server:80`.
3. Go to **Create (+) > Import** and load dashboard IDs **6417** and **1860**.
4. Select the Prometheus data source you just created.

Clean up:

```bash
helm uninstall prometheus
helm uninstall grafana
```

---

## Lab 4 — cert-manager (TLS Certificates)

**Concepts:** CRDs, ClusterIssuers, TLS Secrets, Let's Encrypt

Install cert-manager:

```bash
cd cert-manager/

# Download cert-manager manifest
curl -LO https://github.com/jetstack/cert-manager/releases/download/v1.10.0/cert-manager.yaml

# Install into the cluster
kubectl apply --validate=false -f cert-manager.yaml

# Verify all components are running
kubectl -n cert-manager get all
```

Expected output:
```
NAME                                           READY   STATUS    RESTARTS   AGE
pod/cert-manager-...                           1/1     Running   0          77s
pod/cert-manager-cainjector-...                1/1     Running   0          77s
pod/cert-manager-webhook-...                   1/1     Running   0          77s
```

Test self-signed certificate issuance:

```bash
kubectl create ns cert-manager-test
kubectl apply -f ./selfsigned/issuer.yaml
kubectl apply -f ./selfsigned/certificate.yaml

# Check the certificate was issued
kubectl describe certificate -n cert-manager-test

# Check the TLS secret was created
kubectl get secrets -n cert-manager-test

# Clean up the test namespace
kubectl delete ns cert-manager-test
```

Create a Let's Encrypt ClusterIssuer (for real domains):

```bash
kubectl apply -f cert-issuer-nginx-ingress.yaml

# Verify the issuer is ready
kubectl describe clusterissuer letsencrypt-cluster-issuer
```

---

## Tear Down Everything

Delete the entire minikube cluster:

```bash
minikube delete
```
