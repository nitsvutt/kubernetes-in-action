# Kubernetes in Action

![license](https://img.shields.io/github/license/nitsvutt/kubernetes-in-action)
![stars](https://img.shields.io/github/stars/nitsvutt/kubernetes-in-action)
![forks](https://img.shields.io/github/forks/nitsvutt/kubernetes-in-action)

## Table of Contents
1. [Introduction](#introduction)
2. [Set up devlopment environment](#set-up-dev-env)
3. [Spark on Kubernetes](#spark-on-k8s)
3. [ScyllaDB on Kubernetes](#scylladb-on-k8s)

<div id="introduction"/>

## 1. Introduction

Kubernetes, also known as K8s, is an open source system for automating deployment, scaling, and management of containerized applications. Nowadays, many data systems leverage K8s as a robust container orchestration and resource management at scale.

<div id="set-up-dev-env"/>

## 2. Set up devlopment environment

- Install Docker (typically include `kubectl`): https://docs.docker.com/desktop/setup/install/mac-install/

- Install KIND (Kubernetes IN Docker): https://kind.sigs.k8s.io/docs/user/quick-start/

- Install Helm: https://helm.sh/docs/intro/install/

- Create a K8s cluster:
```
envsubst < $PROJECT_PATH/kubernetes-in-action/k8s/kind_cluster.yml | kind create cluster --config -
```

- Check `cluster-info`:
```
kubectl cluster-info --context kind-my-cluster
```

- Connect to pre-exsisting network:
```
docker network connect lakehouse_platform my-cluster-control-plane
docker network connect lakehouse_platform my-cluster-worker
docker network connect lakehouse_platform my-cluster-worker2
```

<div id="spark-on-k8s"/>

## 3. Spark on Kubernetes

- Create `spark` namespace:
```
kubectl create namespace spark
```

- Create `spark-event-pv` and `spark-event-pvc`:
```
kubectl apply -f ./spark/spark-event-volume.yml
```

- Add repositories for `spark-operator` and `spark-history-server`:
```
helm repo add --force-update spark-operator https://kubeflow.github.io/spark-operator
```
```
helm repo add --force-update stable https://charts.helm.sh/stable
```

- Install `spark-operator` and `spark-history-server` charts:
```
helm install spark-operator spark-operator/spark-operator \
    --namespace spark \
    --set spark.jobNamespaces={spark} \
    --set webhook.enable=true \
    --wait
```
<!-- - Inspect `spark-operator` values if needed:
```
helm get values spark-operator -n spark --all
``` -->
```
helm install spark-history-server stable/spark-history-server \
    --namespace spark \
    --set service.type=NodePort \
    --set service.nodePort=30080 \
    --set nfs.enableExampleNFS=false \
    --set pvc.enablePVC=true \
    --set pvc.existingClaimName=spark-event-pvc \
    --wait
```
<!-- - Inspect `spark-history-server` values if needed:
```
helm get values spark-history-server -n spark --all
``` -->

- Configure `spark-operator-controller` and `spark-submit` RBACs:
```
kubectl apply -f ./spark/spark-operator-controller-rbac.yml
```
```
kubectl apply -f ./spark/spark-submit-rbac.yml
```

- Run a spark application:
```
kubectl apply -f ./spark/spark-pi.yml
```

- Check `spark-pi` application:
```
kubectl describe sparkapp spark-pi -n spark
```

- Check Spark UI at `http://localhost:4045/`:
<p align="center">
    <img src="https://github.com/nitsvutt/kubernetes-in-action/blob/main/asset/spark-ui.png" title="Spark UI" alt="spark-ui" width=100%/>
</p>

- Check Spark History Server UI at `http://localhost:18080/`:
<p align="center">
    <img src="https://github.com/nitsvutt/kubernetes-in-action/blob/main/asset/spark-history-server-ui.png" title="Spark History Server UI" alt="spark-history-server-ui" width=100%/>
</p>

<div id="scylladb-on-k8s"/>

## 4. ScyllaDB on K8s

- Create `scylla` namespace:
```
kubectl create namespace scylla
```

- Create persistent volume:
```
kubectl apply -f $PROJECT_PATH/kubernetes-in-action/scylladb/scylla-persistent-volume.yml
```

- Create service:
```
kubectl apply -f $PROJECT_PATH/kubernetes-in-action/scylladb/scylla-service.yml
```

- Create config map:
```
kubectl apply -f $PROJECT_PATH/kubernetes-in-action/scylladb/scylla-configmap.yml
```

- Create Scylla and Scylla Manager statefulset:
```
kubectl apply -f $PROJECT_PATH/kubernetes-in-action/scylladb/scylla-statefulset.yml
```

- Check Scylla Cluster:
```
kubectl exec -it scylla-0 -n scylla -- \
    nodetool status
```

- Add Scylla Cluster for Scylla Manager:
```
kubectl exec -it scylla-manager-0 -c scylla-manager -n scylla -- \
    sctool cluster add \
    --host scylla-0.scylla-clusterip.scylla.svc.cluster.local \
    --name my-cluster \
    --auth-token $SCYLLADB_AUTH_TOKEN
```