# 020DO-SU22-Repository for minikube labs #

This is meant to familiarize people with working with:

* [minikube](https://github.com/kubernetes/minikube) 
* [MongoDB](https://www.mongodb.com/)
* [Prometheus](https://prometheus.io/)
* [Grafana](https://grafana.com/)
* [Helm](https://helm.sh/)
* [Redis](https://redis.io/)
* [Kubernetes](https://kubernetes.io/) and it's objects: [deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [services](https://kubernetes.io/docs/concepts/services-networking/service/), [ingresses](https://kubernetes.io/docs/concepts/services-networking/ingress/), [configMaps](https://kubernetes.io/docs/concepts/configuration/configmap/), [secrets](https://kubernetes.io/docs/concepts/configuration/secret/), [volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

Directory tree
```
.
├── mongo-webapp
│   ├── mongo-cm.yaml
│   ├── mongo-secret.yaml
│   ├── mongo.yaml
│   ├── README.md
│   └── webapp.yaml
├── prometheus-grafana
│   ├── helm-tutorial
│   │   └── README.md
│   └── README.md
├── README.md
└── redis-server
    ├── README.md
    ├── redis-cm.yaml
    ├── redis-deploy.yaml
    └── redis-pod.yaml
```
this is the easy way