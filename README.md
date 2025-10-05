# Kubernetes in Action

![license](https://img.shields.io/github/license/nitsvutt/kubernetes-in-action)
![stars](https://img.shields.io/github/stars/nitsvutt/kubernetes-in-action)
![forks](https://img.shields.io/github/forks/nitsvutt/kubernetes-in-action)

## Table of Contents
1. [Introduction](#introduction)
2. [Set up devlopment environment](#set-up-dev-env)
3. [Spark on Kubernetes](#spark-on-k8s)


<div id="introduction"/>

## 1. Introduction

Kubernetes, also known as K8s, is an open source system for automating deployment, scaling, and management of containerized applications. Nowadays, many data systems are build on top of K8s as a robust container orchestration and resource management at scale.

<div id="set-up-dev-env"/>

## 2. Set up devlopment environment

- Install Docker (typically include `kubectl`): https://docs.docker.com/desktop/setup/install/mac-install/

- Install KIND (Kubernetes IN Docker): https://kind.sigs.k8s.io/docs/user/quick-start/

- Install Helm: https://helm.sh/docs/intro/install/

- Create a K8s cluster:
```
envsubst < ./k8s/kind_cluster.yml | kind create cluster --config -
```

- Check `cluster-info`:
```
kubectl cluster-info --context kind-my-cluster
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

- Run a spark application using `kubectl apply`:
```
kubectl apply -f ./spark/spark-pi.yml
```

- Run a spark application using `spark-submit`:
```
...
```

- Check `spark-pi` application:
```
kubectl describe sparkapp spark-pi -n spark
```