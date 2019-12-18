---
title: Installing OpenEBS - NDM
date: 2019-12-18T17:50:00Z
summary: Installing dependencies of ropenebs on Alpine
tags: ["k3s"]
---

OpenEBS requires a few dependencies to be installed on Alpine.

#### Install the ISCSI Initiator:
Borrowed from https://www.hiroom2.com/2018/08/29/alpinelinux-3-8-open-iscsi-en/
```sh
apk add open-iscsi
rc-update add iscsid
rc-service iscsid start
```

#### Install udev:
Borrowed from https://wiki.alpinelinux.org/wiki/Gnome_Setup#Setting_up_udev
```sh
apk add udev
/etc/init.d/dev start && /etc/init.d/udev-trigger start && /etc/init.d/
udev-settle start
rc-update add udev sysinit
rc-update add udev-trigger sysinit
rc-update add udev-settle sysinit
```