---
title: K3s cgroup setup
date: 2019-12-10T14:00:00Z
summary: Fixing cgroups in K3s on Alpine
tags: ["k8s"]
---

I'm building a kubernetes cluster on baremetal using some little x64 boxes at home. Installing Alpine and k3s is pretty easy. 

[K3s for Apline](https://rancher.com/docs/k3s/latest/en/advanced/#alpine-linux) states several edits for setting up cgroups.

However, most pods will not start. Just reporting status "ContainerStarting" and "FailedCreatePodSandBox" when `describe`ing them.

The cgroup setup should be ignored. Simply `rc-update add cgroups default` will do the trick. (I just undid the fstab and cgconfig.conf edits)

Finally found the info via https://github.com/rancher/k3s/issues/660#issuecomment-514353060