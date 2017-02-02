
#Platform Maintenance Manual for Trusted Analytics Platform 0.8.0#

**Note: This information is intended for internal Intel use only. Relevant information will be extracted and presented in a public documentation web page for users.**

## 1. Platform upgrades

In general, in order to upgrade platform version, it is sufficient to simply download and unpack new release, copy configuration files formerly used to deploy the platform and run deployment again.

Please note that more manual changes may be necessary in case of a major version update, like additional required parameters in configuration files.

It is important to ensure that the platform is backed up before performing upgrade.

Reminder: TAP deployment automation is based on Ansible. Thanks to idempotency, you can run deployment automation multiple times. Each of those times, only changes will be applied.

**Warning:** Automatic upgrade from versions earlier than 0.8.0 is NOT POSSIBLE - all data and platform configuration from earlier versions of TAP must be migrated manually to TAP 0.8+.

### 1.1. Design

There are 4 major components that might require upgrade:

1. Hardware/VM Infrastructure (AWS, additional malchines, changes in role assigments)
2. Additional base OS based components
3. Newer versions of base OS packages
4. Newer versions of TAP container images

First three components shall follow typical Ansible upgrade logic - additional version check and actions performed when upgrade is necessary.

Upgrade process for containers is simpler: Ansible-Kubernetes module "tapkube" edits deployment instances updating image version and then Kubernetes automatically performs rolling deployment.

Desired rolling deployment strategy can be adjusted in the deployment metadata itself, allowing for zero downtime upgrades in post-0.8 TAP releases.

Please note that core platform componnents, which are using database, will detect an older schema version and perform data/schema upgrades on their own. This process happens after those components are (re)started.

### 1.2. Upgrade procedure details

Please follow the upgrade procedure included with every released version.

### 1.3. Generic upgrade procedure

Please attempt generic upgrade procedure in not detailed upgrade guidelines are provided for a given patch/upgrade.

1. Download TAP installation/upgrade package.
2. Unpack it.
3. Read CHANGELOG file and flow upgrade procedure if attached.
4. Ensure that you are performing upgrade between versions with supported upgrade transition.
5. Use configuration files (primarily tap.config and tap.config.secrets) from previous deployment.
6. Adjust those files and add new options/parameters if needed. Details shall be included in the upgrade procedure.
7. Perform safe platform shutdown.
8. Perform platform backup.
9. Run TAP deployment as usual.

Details on configuration files, troubleshooting, configuration and general deployment procedure you will find in Platform Deployment Manual. 

## 2. Persistent storage maintenance

### 2.1. Health and performance monitoring
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

### 2.2. Free space monitoring
  * Log in to ceph-master node: `ssh ceph-master`. Afterwards get root privileges: `sudo -i`.
  * From now on you have access to ceph and RBD CLI.
  * To monitor space available on the cluster you should use `ceph -s`. You will get information: `space MB used, space MB allocated / space MB available`
  * To monitor space allocated/available on volumes attached to kubernetes containers use `rbd du`

### 2.2. Allocated space extension/reduction
You should watch your cluster capacity. Never let cluster/volume to reach full ratio. There can happen unexcepted errors if OSD hit the limit.

#### 2.2.1. Adding OSDs
You can expand your cluster at runtime. You can use two methods to achieve that.

*Recomended method:*
  * Prepare a centos machine same as you have used to deploy ceph.
  * Login to TAP Jumpbox. Go to tap-deploy directory.
  * In file inventory/k8s section [osds] add IP of the centos machine
  * Execute: `ansible-playbook ceph.yml -i inventory/k8s`

*Alternative method:*
  * Refer to [Ceph documentation](http://docs.ceph.com/docs/jewel/rados/operations/add-or-rm-osds/)

#### 2.2.2. Removing OSDs
Never delete OSD if you do not have enough storage to balance data.

*Recomended method:*
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

*Second method:*
  * Refer to [Ceph documentation](http://docs.ceph.com/docs/jewel/rados/operations/add-or-rm-osds/#removing-osds-manual)

#### 2.2.3. Resizing Kubernetes volumes
  * SSH to ceph-mon and authorize as a root: `sudo -i`
  * execute: `rbd du` to check space and volume name.
  * SSH to kubernetes master node.
  * Scale deployment to 0 using: `kubectl scale deployment --replicas=0 name-of-deployment`
  * Go back to ceph-mon node and execute: `rbd resize --image image-name --size=value` - Value can have size suffix i.e. `1G` = `1024MB`
  * Map volume with rbd CLI: `rbd map image-name` - read mapped i.e.  `/dev/rbd0`
  * Resize file system on volume: `resize2fs /dev/rbd0`
  * Revoke volume mapping: `rbd unmap /dev/rbd0`
  * Scale application deployment back to normal: `kubectl scale deployment --replicas=1 name-of-deployment`

### 2.3. Recovery procedure after Ceph cluster failure
Ceph has no single point-of-failure, and can service requests for data in a “degraded” mode. Ceph is generally self-reparing however, when problems persist, monitoring OSDs and placement groups will help you identify the problem.
Please refer to [Ceph documentation](http://docs.ceph.com/docs/jewel/cephfs/disaster-recovery/) for disaster recovery scenarios.

### 2.4. Safe stop and start procedure 

Ceph have auto-start procedure. If nodes will be restarted due to powerloss they should comeback healthy without administrator help.

There can be two scenarios for maintenance - whole cluster and single machine.

#### 2.4.1. Cluster maintenance:
  * Login to ceph-mon node. 
  * Authorize as a root: `sudo -i`
  * execute: `ceph osd set noout` - prevents osds nodes to be out of cluster.
  * execute command: `systemctl stop ceph.target` first on all osds nodes, then on all mons nodes.

#### 2.4.2. Cluster start procedure:
  * Login to ceph-mon node.
  * Authorize as a root: `sudo -i`
  * execute: `systemctl start ceph.target` first on all mons nodes, then on all osds nodes.
  * execute: `ceph osd unset noout`.

#### 2.4.3. Single node maintenance:
  * If instance have mon and osd roles:
    * execute: `ceph osd set noout`
    * execute command: `systemctl stop ceph.target`

## 3. Platform health, space and performance metrics

TAP provides comprehensive metrics (resource utilization, performance, etc.) via integrated combo of Prometheus and Grafana [[https://prometheus.io/docs/visualization/grafana/]] - platform administrator can easily access them directly from TAP Console.

## 4. Compute resource quota monitoring

TAP sets static memory and CPU limits on every application it creates, to ensure stable and fair scheduling.

Use the following command to list current resource usage, which is listed per Kubernetes worker:

`$ kubectl describe node worker-0.kubernetes.cluster.local worker-1.kubernetes.cluster.local worker-2.kubernetes.cluster.local  (more worker node hostnames can follow)`
