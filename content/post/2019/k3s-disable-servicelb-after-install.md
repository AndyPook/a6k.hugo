---
title: Disable ServiceLB after install
date: 2019-12-11T13:56:00Z
summary: Removing the default ServiceLB
tags: ["k3s"]
---

Simple... Just re-install the master node.

```
curl -sfL https://get.k3s.io | sh -s - server --no-deploy servicelb
```

This should be very similar to the command used to do the initial install. See the syntax under the "INSTALL_K3S_EXEC" option in [Install Options](https://rancher.com/docs/k3s/latest/en/installation/install-options/)

This just removed the DeamonSet for the ServiceLB.

You're now ready to install some other LB (eg [MetalLB](https://metallb.universe.tf/))