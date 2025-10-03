# Kubernetes in Action

![license](https://img.shields.io/github/license/nitsvutt/kubernetes-in-action)
![stars](https://img.shields.io/github/stars/nitsvutt/kubernetes-in-action)
![forks](https://img.shields.io/github/forks/nitsvutt/kubernetes-in-action)

## Table of Contents
1. [Introduction](#introduction)
2. [Set up devlopment environment](#set-up-dev-env)


<div id="introduction"/>

## 1. Introduction

Kubernetes, also known as K8s, is an open source system for automating deployment, scaling, and management of containerized applications. Nowadays, many data systems are build on top of K8s as a robust container orchestration and resource management at scale.

<div id="set-up-dev-env"/>

## 2. Set up devlopment envrionment

- Install Docker (typically include `kubectl`): https://docs.docker.com/desktop/setup/install/mac-install/

- Install KIND (Kubernetes IN Docker): https://kind.sigs.k8s.io/docs/user/quick-start/

- Install Helm: https://helm.sh/docs/intro/install/

- Create a K8s cluster:
```
kind create cluster --config ./k8s/kind_cluster.yml
```
- Check `cluster-info`:
```
kubectl cluster-info --context kind-my-cluster
```