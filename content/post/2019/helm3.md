---
title: Helm3
date: 2019-12-18T17:40:00Z
summary: Get started with Helm3
tags: ["k8s"]
---

[Helm3](https://helm.sh/) does not require the Tiller component to be installed in the cluster.

To install the helm cli on a Windows box via [Chocolatey](https://chocolatey.org/), simply, from a "Run as Administrator" command line:
```sh
choco install -y kubernetes-helm
```
Most current examples will be trying to pull charts from "Stable". No repo's are setup by default in Helm3, so you will need to add "stable"
```sh
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo update
```

Note that the ordering of the args have also changed, so
```sh
helm install stable/openebs --name openebs --namespace openebs
```
becomes
```sh
helm install openebs stable/openebs --namespace openebs
```