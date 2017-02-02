
#Platform Maintenance Manual for Trusted Analytics Platform 0.8#

**Note: This information is intended for internal Intel use only. Relevant information will be extracted and presented in a public documentation web page for users.**

## 1. Platform upgrades

In general, to upgrade the TAP platform version, it is sufficient to simply download and unpack a new release, copy the configuration files previously used to deploy the platform, and run the deployment again.

>Additional manual changes, such as additional required parameter changes in configuration files, may be necessary for major version updates.

**Note:** Make sure you back up the existing platform before performing an upgrade.

>TAP deployment automation is based on Ansible. Due to idempotency, you can run deployment automation multiple times. Each time, only changes will be applied.

**Warning: Automatic upgrades from versions earlier than 0.8.0 is NOT POSSIBLE; all data and platform configuration information from earlier versions of TAP must be migrated manually to TAP 0.8+.**

### 1.1. Upgrade planning

There are 4 major components that may require upgrading:

* Hardware/VM Infrastructure (AWS, additional machines, changes in role assigments, etc.)
* Additional base OS-based components
* Newer versions of base OS packages
* Newer versions of TAP container images

The first three components shall follow typical Ansible upgrade logic - additional version checks and actions are performed when an upgrade is necessary.

The upgrade process for containers is simpler: Ansible-Kubernetes module `tapkube` edits deployment instances, updating the image version, and then Kubernetes automatically performs a rolling deployment.

The desired rolling deployment strategy can be adjusted in the deployment metadata itself, allowing for zero downtime upgrades in post-0.8 TAP releases.

>Note that core platform componnents that use a database will detect an older schema version and perform data/schema upgrades on their own. This process happens after those components are (re)started.

### 1.2. Upgrade procedure details

Follow the upgrade procedure provided with every released TAP version.

### 1.3. Generic upgrade procedure

Use the following generic upgrade procedure unless detailed upgrade guidelines are provided for a given patch/upgrade:

1. Download the TAP installation/upgrade package.
2. Unpack it.
3. Read the CHANGELOG file and flow upgrade procedure if attached.
4. Ensure that you are performing an upgrade between versions with a supported upgrade transition.
5. Use configuration files (primarily `tap.config` and `tap.config.secrets`) from the previous deployment.
6. Adjust those files and add new options/parameters as needed. Details are included in the upgrade procedure.
7. Perform a safe platform shutdown.
8. Perform a platform backup.
9. Run TAP deployment as usual.

Details on configuration files, configuration, general deployment procedure, and troublshooting are included in the Platform Deployment Manual. 

## 2. Persistent storage maintenance

### 2.1. Health and performance monitoring

1. Log in to the ceph-master node: `ssh ceph-master`.
2. Get root privileges: `sudo -i`. (You have access to the ceph CLI.)
3. To check health of the cluster (OSDs, MONs, PGs) run `ceph status`
4. You can watch your cluster health continuously using `ceph -w`. The output provides:
    * Cluster ID
    * Cluster health status
    * The monitor map epoch and the status of the monitor quorum
    * The OSD map epoch and the status of OSDs
    * The placement group map version
    * The number of placement groups and pools
    * The notional amount of data stored and the number of objects stored, and,
    * The total amount of data stored. 

### 2.2. Free space monitoring  
1. Log in to the ceph-master node: `ssh ceph-master`.
2. Get root privileges: `sudo -i`. (You now have access to the ceph CLI and rbd client.)
3. To monitor space available on the cluster, use `ceph -s`. This returns the information: `space MB used, space MB allocated / space MB available`
4. To monitor space allocated/available on volumes attached to Kubernetes containers use `rbd du`.

### 2.2. Allocated space extension/reduction  
Watch your cluster capacity. *Never* let the cluster/volume for an OSD reach full ratio. You can experience unexcepted errors if an OSD ever hits its capacity limit.

#### 2.2.1. Adding OSDs  
You can expand your cluster at runtime, using either of two methods to achieve this:

*Recomended method:*  

1. Prepare a CentOS machine like the one you used to deploy ceph.  
2. Login to your TAP Jumpbox and go to the `tap-deploy` directory.  
3. In the file inventory/k8s section [osds], add the IP of the CentOS machine  
4. Execute: `ansible-playbook ceph.yml -i inventory/k8s`  

*Alternate method:*  

Refer to [Ceph documentation](http://docs.ceph.com/docs/jewel/rados/operations/add-or-rm-osds/).  

#### 2.2.2. Removing OSDs
**WARNING: Never delete an OSD if you do *not* have enough storage to balance data.**

*Recomended method:*  

1. Login to your TAP Jumpbox, then to the ceph-mon node.
2. Authorize as a root: `sudo -i`
3. Check the OSDs tree: `ceph osd tree` and read the ID of the node you want to delete.
4. Execute: `ceph osd out osd-id`
5. Watch data migration with `ceph -w` - Status should change from `active+clean` -> `active, some degraded objects` -> `active+clean`. 

   >It can take from 5 minutes to few hours for this sequence to complete, depending on data size.

6. When the data migration completes, ssh to osd-host
7. Execute: `sudo systemctl stop ceph.target`
8. Go back to ceph-mon and delete: 
    * OSD from crush map: `ceph osd crush remove osd.osd-id`
    * OSD authentication key: `ceph osd del osd.osd-id`
    * OSD: `ceph osd rm osd-id`
9. Check `ceph -s` - you should see that the OSD has been removed.

*Alternate method:*  

Refer to [Ceph documentation](http://docs.ceph.com/docs/jewel/rados/operations/add-or-rm-osds/#removing-osds-manual).

#### 2.2.3. Resizing Kubernetes volumes
1.  ssh to the ceph-mon node and authorize as a root: `sudo -i`
2.  Execute: `rbd du` to check space and volume name.
3.  ssh to the Kubernetes master node.
4.  Set `scale deployment` to 0 using: `kubectl scale deployment --replicas=0 name-of-deployment`
5.  Go back to ceph-mon node and execute: `rbd resize --image image-name --size=value` - Value can have size suffix, for example: `1G` = `1024MB`
6.  Map the volume with the rbd CLI: `rbd map image-name` - read mapped, for example: `/dev/rbd0`
7.  Resize the file system on the volume: `resize2fs /dev/rbd0`
8.  Revoke volume mapping: `rbd unmap /dev/rbd0`
9.  Set `scale application deployment` back to normal: `kubectl scale deployment --replicas=1 name-of-deployment`

### 2.3. Recovery procedure after Ceph cluster failure
Ceph has no single point-of-failure and can service requests for data in a “degraded” mode. Ceph is generally self-reparing. However, when problems persist, monitoring OSDs and placement groups will help you identify the problem.

Refer to [Ceph documentation](http://docs.ceph.com/docs/jewel/cephfs/disaster-recovery/) for disaster recovery scenarios.

### 2.4. Safe stop and start procedure 

Ceph has an auto-start procedure. If nodes are restarted due to power loss, they should comeback healthy without administrator help.

There are two scenarios for maintenance: entire cluster and single machine.

#### 2.4.1. Cluster maintenance:
1. Login to the ceph-mon node.  
2. Authorize as a root: `sudo -i`  
3. Execute: `ceph osd set noout` to prevent osds nodes from being out of the cluster.  
4. Execute command: `systemctl stop ceph.target` first on all osds nodes, then on all mons nodes.  

#### 2.4.2. Cluster start procedure:
1. Login to the ceph-mon node.  
2. Authorize as a root: `sudo -i`  
3. Execute: `systemctl start ceph.target` first on all mons nodes, then on all osds nodes.  
4. Execute: `ceph osd unset noout`.  

#### 2.4.3. Single node maintenance:
If an instance has mon and osd roles:  
1. Execute: `ceph osd set noout`  
2. Execute command: `systemctl stop ceph.target`  

## 3. Platform health, space and performance metrics

TAP provides comprehensive metrics (resource utilization, performance, etc.) via an integrated combination of Prometheus and Grafana [[https://prometheus.io/docs/visualization/grafana/]]. Platform administrators can easily access them directly from TAP Console.

## 4. Compute resource quota monitoring

TAP sets static memory and CPU limits on every application it creates, to ensure stable and fair scheduling.

Use the following command to list current resource usage, which is listed per Kubernetes worker:  

`$ kubectl describe node worker-0.kubernetes.cluster.local worker-1.kubernetes.cluster.local worker-2.kubernetes.cluster.local  (more worker node hostnames can follow)`
