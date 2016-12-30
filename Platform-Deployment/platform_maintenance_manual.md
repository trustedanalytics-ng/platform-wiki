---
title: TAP 0.8 Platform Maintenance Manual
keywords: platform maintenance manual
last_updated: 'December 30, 2016'
tags:
  - Platform Maintenance
summary: >-
  Insert the summary paragraph here. To edit the summary you must edit the meta data for this post.
sidebar: mydoc_sidebar
permalink: platform-maintenance-manual.html
folder: mydoc
published: true
---

**Platform Maintenance Manual**
Trusted Analytics Platform 0.8

>This information is intended for internal Intel use only. Relevant information will be extracted and presented in a public documentation web page for users.


## Platform backup and restore

[???]
WALDEK TO PROVIDE PROCEDURE
[???]

## Platform upgrades

[???]
TODO
[???]

## Graceful/safe platfrom shutdown

[???]
WALDEK TO PROVIDE PROCEDURE
[???]

## Persistent storage maintenance

### Health and performance monitoring
  * Log in to ceph-master node: `ssh ceph-master`. Afterwards get root privileges: `sudo -i`.
  * From now on you have access to ceph CLI.
  * To check health of the cluster (OSDs, MONs, PGs) do `ceph status`
  * You can watch your cluster health all the time using `ceph -w`. The output provide:
    * Cluster ID
    * Cluster health status
    * The monitor map epoch and the status of the monitor quorum
    * The OSD map epoch and the status of OSDs
    * The placement group map version
    * The number of placement groups and pools
    * The notional amount of data stored and the number of objects stored; and,
    * The total amount of data stored. 

### Free space monitoring
  * Log in to ceph-master node: `ssh ceph-master`. Afterwards get root privileges: `sudo -i`.
  * From now on you have access to ceph and RBD CLI.
  * To monitor space available on the cluster you should use `ceph -s`. You will get information: `space MB used, space MB allocated / space MB available`
  * To monitor space allocated/available on volumes attached to kubernetes containers use `rbd du`

### Allocated space extension/reduction
You should watch your cluster capacity. Never let cluster/volume to reach full ratio. There can happen unexcepted errors if OSD hit the limit.

#### Adding OSDs
You can expand your cluster at runtime. You can use two methods to achieve that.

##### Recomended method:
  * Prepare a centos machine same as you have used to deploy ceph.
  * Login to TAP Jumpbox. Go to tap-deploy directory.
  * In file inventory/k8s section [osds] add IP of the centos machine
  * Execute: `ansible-playbook ceph.yml -i inventory/k8s`

##### Second method:
  * Refer to [Ceph documentation](http://docs.ceph.com/docs/jewel/rados/operations/add-or-rm-osds/)

#### Removing OSDs
Never delete OSD if you do not have enough storage to balance data.

##### Recomended method:
  * Login to TAP jumpbox, then to one of ceph-mon. Authorize as a root: `sudo -i`
  * Check the OSDs tree: `ceph osd tree` - read ID of the node you want to delete.
  * execute: `ceph osd out osd-id`
  * watch data migration with `ceph -w` - Status should change from `active+clean` -> `active, some degraded objects` -> `active+clean`. It can take from 5 minute to few hours depends on size of data.
  * When data migration end ssh to osd-host
  * execute: `sudo systemctl stop ceph.target`
  * Go back to ceph-mon and delete: 
    * OSD from crush map: `ceph osd crush remove osd.osd-id`
    * OSD authentication key: `ceph osd del osd.osd-id`
    * OSD: `ceph osd rm osd-id`
  * Check `ceph -s` - you should see that OSD has been removed.

##### Second method:
  * Refer to [Ceph documentation](http://docs.ceph.com/docs/jewel/rados/operations/add-or-rm-osds/#removing-osds-manual)

#### Resizing Kubernetes volumes
  * SSH to ceph-mon and authorize as a root: `sudo -i`
  * execute: `rbd du` to check space and volume name.
  * SSH to kubernetes master node.
  * Scale deployment to 0 using: `kubectl scale deployment --replicas=0 name-of-deployment`
  * Go back to ceph-mon node and execute: `rbd resize --image image-name --size=value` - Value can have size suffix i.e. `1G` = `1024MB`
  * Map volume with rbd CLI: `rbd map image-name` - read mapped i.e.  `/dev/rbd0`
  * Resize file system on volume: `resize2fs /dev/rbd0`
  * Revoke volume mapping: `rbd unmap /dev/rbd0`
  * Scale application deployment back to normal: `kubectl scale deployment --replicas=1 name-of-deployment`

### Recovery procedure after Ceph cluster failure
Ceph has no single point-of-failure, and can service requests for data in a “degraded” mode. Ceph is generally self-reparing however, when problems persist, monitoring OSDs and placement groups will help you identify the problem.
Please refer to [Ceph documentation](http://docs.ceph.com/docs/jewel/cephfs/disaster-recovery/) for disaster recovery scenarios.

### Safe stop and start procedure 

Ceph have auto-start procedure. If nodes will be restarted due to powerloss they should comeback healthy without administrator help.

There can be two scenarios for maintenance - whole cluster and single machine.

#### Whole cluster maintenance:
  * Login to ceph-mon node. 
  * Authorize as a root: `sudo -i`
  * execute: `ceph osd set noout` - prevents osds nodes to be out of cluster.
  * execute command: `systemctl stop ceph.target` first on all osds nodes, then on all mons nodes.

#### Whole cluster start procedure:
  * Login to ceph-mon node.
  * Authorize as a root: `sudo -i`
  * execute: `systemctl start ceph.target` first on all mons nodes, then on all osds nodes.
  * execute: `ceph osd unset noout`.

#### Single node maintenance:
  * If instance have mon and osd roles:
    * execute: `ceph osd set noout`
    * execute command: `systemctl stop ceph.target`


## Platform health, space and performance metrics

TAP provides comprehensive metrics (resource utilization, performance, etc.) via integrated combo of Prometheus and Grafana [[https://prometheus.io/docs/visualization/grafana/]] - platform administrator can easily access them directly from TAP Console.

## Compute resource quota monitoring

TAP sets static memory and CPU limits on every application it creates, to ensure stable and fair scheduling.

Use the following command to list current resource usage, which is listed per Kubernetes worker:

`$ kubectl describe node worker-0.kubernetes.cluster.local worker-1.kubernetes.cluster.local worker-2.kubernetes.cluster.local  (more worker node hostnames can follow)`
