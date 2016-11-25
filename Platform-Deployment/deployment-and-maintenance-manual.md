---
title: TAP 0.8 Platform Deployment & Maintenance Manual
keywords: platform deployment maintenance manual
last_updated: 'November 24, 2016'
tags:
  - Platform Deployment
summary: >-
  Insert the summary paragraph here. To edit the summary you must edit the meta
  data for this post.
sidebar: mydoc_sidebar
permalink: deployment-and-maintenance-manual.html
folder: mydoc
published: true
---

Deployment and Maintenance Manual
Trusted Analytics Platform 0.8 

>This information is intended for internal Intel use only. Relevant information will be extracted and presented in a public documentation web page for users.

# Introduction


Welcome to the TAP 0.8 Deployment and Maintenance Manual.


The goal of this document is to explain TAP deployment automation and its architecture design, and how to perform essential maintenance procedures.

## Required knowledge


While platform installation for testing purposes is easy, long-term production deployments require more in-depth knowledge of the underlying services and components.


Domain areas where administrator skills are assumed:


* Linux administration
* Kubernetes administration (see [[http://kubernetes.io/docs/user-guide/]])
* Docker (user-level) (see [[https://docs.docker.com/]]) 
* Hadoop administration (see [[http://www.cloudera.com/documentation.html]]) 
* Ceph administration (see [[http://docs.ceph.com/docs/master/]])
* IP network administration

Local workstation requirements:
* Linux (???)
* Web browser: Chrome, Firefox


This document freely uses common terminology associated with those areas.

# Deployment

## Deployment requirements


TAP version 0.8 supports CentOS 7+ on x86_64 architecture. 

[move to section on security / initial node setup] It is best to provide machines with passwordless `sudo` access and key-based authentication.


TAP 0.8 core functionalities are based on these technologies:

* computing cluster: Kubernetes 1.3
* persistent storage: ceph 11
* hadoop cluster: CDH 5.7.1 (Cloudera Hadoop distribution)


These are “typical” TAP installation configurations:


## Minimum configuration

This configuration is recommended for trying out TAP and its analytics features.

* The suggested Minimum Configuration layout consist of 2 nodes (hosts) only: 
      1. 1 node:
         * compute-master (in TAP 0.8: kubernetes master)
         * storage-master (in TAP 0.8: ceph master)
         * compute-worker (in TAP 0.8: kubernetes worker)
         * storage-slave (in TAP 0.8: ceph worker)
      1. 1 node:
         * jumpbox
         * hadoop-master (in TAP 0.8: CDH Master Primary)
         * hadoop-manager (in TAP 0.8: CDH Manager)
         * hadoop-worker (in TAP 0.8: CDH Worker)


* Hardware requirements for each node:
     * 24 GB of RAM
     * 4 CPU cores
     * 250 GB of HDD


**Note:** The minimal configuration does *not* provide important reliability features that are required for production deployments.


## Medium configuration

Characteristics of this configuration:

* Redundant functions are distributed among separate nodes to provide high availability and data replication [UPDATE HOST ROLES TO REFLECT HA REQUIREMENTS]
* *Not* focused on raw platform performance

* Suggested Medium Configuration layout consist of 8 nodes (hosts):
      1. 1 node:
         * compute-master (in TAP 0.8: kubernetes master)
         * storage-master (in TAP 0.8: ceph master)
         * load-balancer
      1. 3 nodes:
         * compute-worker (in TAP 0.8: kubernetes workers)
         * storage-slave (in TAP 0.8: ceph workers)
      1. 1 node:
         * jumpbox
         * hadoop-master (in TAP 0.8: CDH Master Primary)
         * hadoop-manager (in TAP 0.8: CDH Manager)
      1. 3 nodes:
        * hadoop-worker (in TAP 0.8: CDH Workers)

* Hardware requirements for each node:
     * 16 GB of RAM
     * 4 CPU cores
     * 200 GB of HDD


## Production configuration

Characteristics of this configuration:
* Redundant functions are distributed among separate nodes to provide performance, high availability and replication [UPDATE HOST ROLES TO REFLECT HA REQUIREMENTS

* Suggested Production Configuration layout consist of 20+ nodes (hosts):
      1. 1 node [PROVIDE H/W REQUIREMENTS]:
         * bastion host [jumpbox ?]
      1. 3 nodes [PROVIDE H/W REQUIREMENTS]:
         * load-balancer
      1. 1 node [PROVIDE H/W REQUIREMENTS][UPDATE FOR HA]:
         * compute-master (in TAP 0.8: kubernetes master)
      1. 3+ nodes [PROVIDE H/W REQUIREMENTS]:
         * compute-worker (in TAP 0.8: kubernetes workers)
      1. 3 nodes [PROVIDE H/W REQUIREMENTS]:
         * storage-master (in TAP 0.8: ceph master)
      1. 3+ nodes [PROVIDE H/W REQUIREMENTS]:
         * storage-slave (in TAP 0.8: ceph workers)
      1. 1 node [PROVIDE H/W REQUIREMENTS][UPDATE FOR HA]:
         * hadoop-master (in TAP 0.8: CDH Master Primary)
      1. 1 node [PROVIDE H/W REQUIREMENTS][UPDATE FOR HA]:
         * hadoop-master-secondary (in TAP 0.8: CDH Master Primary)
      1. 1 host [PROVIDE H/W REQUIREMENTS][UPDATE FOR HA]:
         * hadoop-manager (in TAP 0.8: CDH Manager)
      1. 3+ hosts [PROVIDE H/W REQUIREMENTS][UPDATE FOR HA]:
         * hadoop-worker (in TAP 0.8: CDH Worker)

* Persistent storage is installed on separate nodes (roles: storage-master and storage-slave). You should follow Ceph documentation on recommended hardware setups: [[http://docs.ceph.com/docs/jewel/start/hardware-recommendations/]]


# Deployment procedure in a nutshell


[ToDo: include flowchart and description of deployment flow]


TAP has a unified installation procedure for both bare metal (hardware and operating systems provisioned by the user) and cloud deployments (TAP deployment automation creates infrastructure). Here are the steps:


1. Ensure that your CentOS 7 (x86_64) OS is up to date
   1. Run: `sudo yum update`
   2. Run: `sudo yum upgrade`
   3. Reboot, to ensure both kernel and userland are running with up-to-date code.
1. If using SSH keys to authenticate inside the non-cloud-provider cluster, put the keys on the deployment machine. (You will need paths to them in the configuration file.)
2. Download the `TAP-0.8.0.tar.gz` package (or other version of TAP if applicable).
[ToDo: FROM?]
1. Run: `tar -xf TAP-0.8.0.tar.gz`
2. Copy the selected config file template from `tap.config.example\*` to `tap.config`  
*Note:* there are several example configuration files provided with different environment size and underlying platforms.
1. Edit `tap.config`. See the _Config Master File_ subsection below for details.
    1. If you need to create aws cloud infrastructure, run: `./deploy.sh infra-aws`
    2. If you need to install TAP on existing infrastructure, run: `./deploy.sh infra-bare-metal`
3. Once the cloud infrastructure is successfully provisioned, run: `./deploy.sh deploy`
        [ToDo: Make sure all necessary artifacts are automatically copied to the provisioned jumpbox machine]


**Notes:**
* Deployment scripts internally use Ansible, following its idempotent nature. This means you can run `./deploy.sh` deploy over and over again, and only changes will be applied.

* This feature can be used to repair broken host with TAP 0.8 only.

* Automatic cluster size extension, shrinking or node role change is not yet supported in TAP 0.8.*.




### Idempotency

"The concept that change commands should only be applied when they need to be applied, and that it is better to describe the desired state of a system than the process of how to get to that state. As an analogy, the path from North Carolina in the United States to California involves driving a very long way West but if I were instead in Anchorage, Alaska, driving a long way west is no longer the right way to get to California. Ansible’s Resources like you to say “put me in California” and then decide how to get there. If you were already in California, nothing needs to happen, and it will let you know it didn’t need to change anything."
     (from: [[http://docs.ansible.com/ansible/glossary.html]])


# Deployment flow


[ToDo: describe mapping of Config Master File roles to Ansible roles!]


The following diagram shows which Ansible role is executed on which machine group:


[[img/docs.png]]

[ TODO: merge roles to the generic ones - diagram to be smaller thanks to that ]
  



Deployment is performed by Ansible playbooks. All roles used during deployment are available in the `roles/` directory. The diagram above shows which role is executed on which machine over time during the deployment process.


For Cloud-Provider-specific code, please refer to the Cloud Deployment Design section below. 


Deployment starts with setting up a jump node (bastion).


TODO 




The last deployment step is to install applications by creating Kubernetes objects like “Deployment”, “Service”, “ConfigMap”, etc.


After platform deployment, service offerings (those available in the TAP Marketplace) are installed. Role execution flow is as follows:

[[img/apps_docs.png]]
  


[ TODO: merge roles to the generic ones - diagram to be smaller thanks to that ]


# Config master File - tap.config


To provide a single configuration point for the entire, quite complex platform, TAP deployment uses the file format described in this section.

During the deployment process, other configuration files are being generated, like Ansible inventory file and group var files for various subsystems.


## Sample tap.config file:

```
    ntp_servers: ['0.pool.ntp.org', '1.pool.ntp.org']
    smpt_server: 192.168.0.100
    wildcard_tap_domain_name: "example.com"
    tap_version: 0.8.0-1801
    deployment_type: minimal
    tap_kerberos_secured: yes  [ToDo: expose this deployment param]
    
    
    all_platform_offerings:
      - consul
      - etcd
      - couchdb16
      - redis30
      - postgresql93
      - rabbitmq36
      - memcached14
      - logstash14
      - influxdb088
      - cassandra21
      - jupyter
      - mosquitto14
      - gateway
      - h2o
      - mysql56
      - mongodb30
      - hive
      - hdfs
      - zookeeper
      - orientdb
    
    
    instances:
      hadoop-machine:
            ip: 10.0.211.8
            user: centos
            type: largex
            connection:
                      type: ssh-key
            storage:
                    size: 500G
            roles:
              - hadoop-manager
              - hadoop-worker
              - hadoop-master-primary
      compute-machine:
            ip: 10.0.97.158
            user: centos
            type: largex
            connection:
                  type: local
            storage:
                size: 200G
            roles:
              - jumpbox
              - compute-master
          - compute-worker
          - compute-etcd
          - storage-master
          - storage-slave
```


## Persistent storage allocation

All machines defined in the Config Master File should define the persistent storage they will be allocating for use.


There are 2 modes of persistent storage allocation that these machines can use:

- Claimed storage
- Attached storage.

**Note:** These two modes of persistent storage allocation are exclusive; during deployment only one of them can be used per machine.


Persistent storage is used by TAP itself, TAP-hosted services, and Hadoop, depending on the role of a given machine.


### Claimed storage

Claimed storage allocation is mostly used in cloud deployments and other installations without dedicated disk devices allocated for platform’s persistent storage.

To claim persistent storage the following attribute (part of machine description) must be used:

```
     storage:
           size: ???G
```  
  
     where ??? is the actual amount of disk storage claimed on this machine for persistent storage use.

In a cloud-based installation (or, generally, whenever platform-as-a-service is used), this definition creates a dedicated persistent storage of the requested size that is mounted as a separate disk device on the machine being created.

In a bare-metal installation (or, generally, whenever storage is provided as OS-visible storage devices), this definition creates virtual persistent storage on the main/root disk allocating as much memory as it is specified here.

**Note:** Creation of such virtual persistent storage will fail if the amount of disk space left for OS is smaller than 100GB (this default size limit can be changed via optional deployment parameter called: xxxxxx).


### Attached storage

Attached storage allocation is used whenever storage is provided as OS-visible storage devices that can be exclusively allocated for TAP persistent storage.

Allocation is recommended in a bare-metal installations: the OS resides on one storage device (should not be smaller than 100GB) while the other available storage devices (physical or virtual discs) can be exclusively allocated for TAP persistent storage.

To attach a persistent storage in this mode, the following attribute (a part of machine description) must be used:

```
     storage:
            disks: * 
```  
  

     and all available unused (unattached) storage devices will be allocated here.


Alternatively, the user can provide a list of named devices to be used for this purpose, such as:

```
     storage:
           disks:
               - /dev/hdd2
               - /dev/hdd3
               - /dev/hdd4
               - /dev/hdd5
```


**Note:** This storage allocation mode (attached storage) does *not* create any storage when the deployment is done in the cloud; it just attaches storage devices already made available for TAP.


## Required Parameters

* roles
   * Comma-separated list of roles that this particular machine will perform. The full list of available roles is available in the section *Supported Machine Roles*.
* admin-password
   * Administrator password used for sign in to TAP command line tool and web console.
   * Suggested: At least 8 characters; include special characters.
* cdh-manager-password
   * Password for CDH manager. Cannot be changed.
* wildcard_tap_domain_name
   * Domain name pointing to the TAP Primary Load Balancers. It should direct all traffic directed to this name and all subdomains below to those hosts.
   * Example value: `example.com`
      * It should point to the IP address (or CNAME, etc.) of the Primary Load Balancers
      * All subdomains, like `sub.example.com`, should point to those hosts too.
* deployment_type
   * Use value `minimal` to enable various optimizations to make TAP work in a resource constrained environments. Do *not* use `minimal` on production environments.
* all_platform_offerings
   * List of all offerings that shall be available on TAP Marketplace by default.
   * Additional offerings can be included later via TAP APIs.

(question .. are the lines below placeholders for additional paramters?)

smtp_server  
smtp_user  
smtp_pass  
kerberos_protected  


## Optional Parameters

* ntp_servers
   * List of NTP servers. NTP is required for TAP, as many clustered services require precise and synchronized time.
   * Sample value: `['0.pool.ntp.org', '1.pool.ntp.org']`


### Supported Machine Roles

* jumpbox
   * Hardened bastion host dedicated to protecting TAP installation from external attacks. Refer to [[https://en.wikipedia.org/wiki/Bastion_host]] for more details.
   * Suggested hardware: Any machine. Assign proper network security rules.
* nat
   * Used only on AWS deployments. Refer to [[http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-nat-gateway.html]] for more details.
   * Suggested hardware: Role used only on AWS.
* compute-master
   * Kubernetes Master Nodes. Hosts management processes for Kubernetes, like:
      * Kube-apiserver
      * Kube-scheduler
      * Kube-controller
      * flannel (virtual networking; [[https://github.com/coreos/flannel]])
   * TAP supports multiple kubernetes master nodes.
   * Suggested hardware: ?
* compute-worker
   * Kubernetes Worker Node. Here pods and containers are being executed. Hosts the following components:
      * docker (container runtime; [[https://www.docker.com/]])
      * Kubelet
      * Kube-proxy
      * flannel (virtual networking; [[https://github.com/coreos/flannel]])
   * TAP supports multiple kubernetes worker nodes.
   * Suggested hardware: ?
* compute-etcd[a]
   * Machine (or VM) that hosts etcd clusters used strictly by Kubernetes cluster.
   * As etcd is Raft-based cluster, suggested number of nodes is 3, 5 or 7. Please follow the official guide for more details: [[https://coreos.com/etcd/docs/latest/admin_guide.html]]
* hadoop-manager
   * Cloudera Hadoop distribution manager application ([[http://www.cloudera.com/products/cloudera-manager.html]]) is an friendly web interface to the Hadoop cluster. One of the primary roles of this component is to automate CDH installation and monitoring.
   * **Note:** This node is required in TAP 0.8.
   * Suggested hardware: TODO FIXME - follow CDH guidelines.
* hadoop-master
   * Cloudera Hadoop distribution master applications installed on this node are:
      * NameNode,
      * YARN,
      * FIXME TODO
   * Suggested hardware: TODO FIXME - follow CDH guidelines
* hadoop-worker
   * Cloudera Hadoop distribution worker components installed on those nodes are:
      * HDFS storage ([[http://hadoop.apache.org/]])
   * Suggested hardware: TODO FIXME - follow CDH guidelines
* storage-worker
   * Ceph storage nodes ([[http://ceph.com/]]) - provide reliable persistent network storage functionality. On those hosts Ceph volumes keeps their data.
   * Suggested number of nodes is 3, 5 or 7.
   * Follow Ceph documentation for guidelines of hardware setup.
* storage-master
   * Ceph monitor nodes. Those hosts supervise reliable Ceph cluster operation.
   * Follow Ceph documentation for guidelines of hardware setup.



## Bare Metal Deployment Design

TAP deployment functionality is universal and assumes only SSH connectivity to all the hosts you want to install TAP on. While this allows you to install platform on almost any kind of physical and virtual machines, to improve performance and reliability, TAP provides functionality to *not* use generic hardware features of your hardware infrastructure.

**Note:**  Bare metal deployment, where an installed supported operating system is required to install TAP, is different than network boot-based deployment. Plans are for TAP to support network boot installation method in the future.



### Customizations on Bare Metal

TODO FIXME list from config file - ceph volume mgmt, 


# Cloud deployment design

## AWS - Overview

TAP deployment automation has the functionality to provision infrastructure on AWS EC2.

The infrastructure being deployed consists of VPC, subnets, EC2 instances, ELB (Elastic Load Balancers).

Provisioning is implemented in Ansible, using AWS module ([[http://docs.ansible.com/ansible/guide_aws.html]]). Thanks to that, we obtain proper idempotent design, where re-running automation should fix infrastructure problems.

Infrastructure provisioning deployment steps generates necessary configuration and inventory files for later use. 

There are two major subnets allocated: one for compute resources (Kubernetes cluster) and one for Hadoop resources (Cloudera cluster). 


# Architecture

## Deployment troubleshooting


As with any complex system, both expected failures (e.g., network connectivity) and unexpected failures may appear. This section should prepare you to handle and debug those.


## General problem investigation procedure


Make sure the target machine is in an overall good shape:
* Check available disk storage using `df -h`. Machines without free disk space will experience issues in many different areas.
* Check available memory using `free -m`. Machines without memory will invoke OOM Killer, which will terminate processes.
* Ensure that machines have at least 1GB of memory free (or allocated as buffers/cache)
* Check machine load, using uptime, top, or ps auxf. Machines under very high load can drop packages, timeouts may appear. Machine load should not exceed 70% in healthy, responsive clusters.
* Check logs:
   * Modern Linux distributions use Systemd. Use `sudo systemctl` and `sudo journalctl` to view services and logs for the machine. For systemd reference, refer to [[https://wiki.archlinux.org/index.php/Systemd]]
   * Some older components still use `/var/log/` directory to store logs.
   * The package integrates an ELK stack, which can be used to browse logs across all nodes, applications, and containers. 
      * TODO provide instructions.
* Check metrics:
   * Using either TAP web console, or
   * Grafana integrated with TAP. It is available via [[https://grafana.${PLATFORM_WILDCARD_DNS_NAME}]]
* On Kubernetes nodes:
   * Check pods health, using ``sudo -u tap-admin kubectl get pods`. If everything is in a "Running" state, the pods are healthy. If not, check unhealthy pods logs using `sudo -u tap-admin kubectl logs $POD_ID`, and for overall status `sudo -u tap-admin kubectl describe $POD_ID`.


### Deployment investigation data


When troubleshooting, in addition to information gathered in *General problem investigation* section above, make sure the following files are correct:


* `tap.config`
* `deployment.log`


**Note:** some configurations files and log files may contain sensitive information, like user credentials.





# TAP Core Components

## TAP Load Balancing


TAP manages load balancers for the platform in order to:


* Improve bastion host security, as load balancer machines have stripped and hardened configuration
* Provide transparent TLS functionality
* Provide scaling functionality, where single application URL can be redirected to multiple application instances.

### Load Balancers architecture


     ____________
    (            )
    (  Internet  )
    (____________)
  
  
          /\
    ***** || ********* TAP BELOW ************ TAP BELOW ********** TAP BELOW ****
          \/
     __________________         ________________________ /--- application1, instance 1
    |   Primary LB     |       | Ingress Load Balancers |
    | (nginx TCP mode) |  <==> | nginx HTTPS mode       |--- application1, instance 2
    |__________________|       |________________________|
                                                        |  \___ application2, instance 1
                                                        |
                                                        |___ service1, instance 1
                                                        |___ core component, instance 1
                                                        |___ .....
                                                        |___ .....
  

### Primary Load Balancers


This group represents dedicated hosts (physical, VMs) directly accessible from the Internet. The package selection on those is minimal. Hardened packages are installed where possible.


The primary applications installed here are:
* Firewall (iptables) - to block unnecessary traffic and allow administrators to cut traffic on the network level
* Load Balancer application - Nginx
   * Nginx is configured to redirect all traffic on ports 80 and 443 to the Ingress Load Balancers machine group.




### Ingress Load Balancers


This group represents Pods running on Kubernetes, hosting Nginx load balancer dynamically reconfigured by management components hosted on the same host.


Mode of operation:
* Periodically get all the "Ingress" objects from Kubernetes.
* For each "Ingress" object - lookup relevant "Service" objects.
* Generate Nginx configuration file using those objects.
* Reload Nginx in graceful, no downtime, mode.


With proper generated Nginx configuration files, traffic is distributed to targeted applications, services and components based on the "Host" header in HTTP(S) requests.


* Please note that the TLS certificate passed in Ingress objects are supported in TAP 0.8 (ones over which session key shall be negotiation with the Client from the Internet).
* Please note that "Endpoint" Kubernetes objects, when backed up with proper "Service" and "Ingress" objects, are supported in TAP 0.8.
* Please note that Ingress Load Balancers hosts are pre-loaded with TAP CA certificate and should access internal services over HTTPS.


DNS and entry router configuration with multiple Load Balancers


In order to support multiple Primary Load Balancer hosts, the administrator shall configure:


* When using AWS EC2 Elastic Load Balancer (ELB):
   * Ensure that traffic is directed to all the machines in a round-robin fashion.
   * Ensure that the wildcard TAP domain name points to the ELB CNAME DNS records.
* When using company-internal router / load balancer:
   * Ensure that traffic is directed to all the machines in a round-robin fashion.
   * Ensure that the wildcard TAP domain name points to the proper endpoint of the company-internal router / load balancer.
* When DNS-based load distribution is expected:
   * Ensure that all Primary Load Balancer hosts have their entry under the wildcard TAP domain name DNS entry.


## Transport Level Security in TAP


To ensure that all traffic inside of clusters is protected by TLS, TAP provides its own Certificate Authority (CA) components. This component is used by all components, to generate keys and certificates necessary for platform operation.


### Implementation


CA Service uses CFSSL ([[https://github.com/cloudflare/cfssl]]) to generate and manage local Certification Authority. It is exposed via a simplified REST API (a custom one, as cfssl exposes too much functionality).


To every Kubernetes Pod the platform starts (one exposing some functionality over integrated HTTP server) we add an additional container - with nginx. This Nginx is generated over a CA Service API key and certificate. It is proxying exposed port :443  to 127.0.0.1:80 in the same Pod (targetting an actual application).


To allow other services and applications to connect to the ones secured with platform's CA, we mount an additional secret as a volume to each Pod. This secret contains CA certificates (in popular formats) used by the platform. This certificate is used by default for all applications.


### Architecture Diagram

The following diagram describes how certificates and keys are attached to the Pods:




[[img/ca.png]]

 
  

Lifecycle of the TLS-related Kubernetes Secrets objects:

[[img/ca2.png]]

## Network connectivity inside TAP


All compute nodes and Hadoop nodes, as well as containers running on TAP, are interconnected using Flannel ([[https://github.com/coreos/flannel]]). 


This allows every container instance (specifically: Kubernetes Pod) to get its own IP address and full connectivity to other hosts and pods in the TAP environment (assuming  it is supporting only a single organization).


DNS - SkyDNS
 
TAP uses SkyDNS with a Kubernetes plug-in to provide internal domain names for every Kubernetes service. Some DNS records are created during the deployment process for Cloudera Hosts (CDH require such DNS entries).

The Kubernetes SkyDNS integration works by polling the list of “Service” objects and adjusting DNS entries automatically. The important aspect of this is that the SkyDNS cluster IP is by default added to `/etc/resolv.conf` on each container Kubelet starts (so it should work out of the box with everything which is dynamically linked with `glibc`).


## Logs aggregation - ELK stack

All TAP hosts are configured to push their logs (via SystemD) to the ElasticSearch stack (ElasticSearch, Logstash, Kibana).


In addition, a Kubernetes addon is deployed. Because to this addon, all containers also push their stdout and stderr to the ELK instance.


You can deploy more than one ELK instance - they will form a cluster. At least 3 nodes are recommended for production clusters.


## Aggregated metrics

TAP provides TODO


## Container images - docker registry

Most of the TAP components are being shipped as a Docker Container Images. In order to save disk space and network bandwidth, there is a whole Docker Registry v2 ([[https://github.com/docker/distribution]]) data directory distributed inside a TAP package.


During platform installation:

1. A dedicated Docker Registry v2 is started, backed by empty persistent storage.
2. A temporary Docker Registry v2 is started, with a data directory mounted from the unpacked TAP package.
3. Images are pushed from the temporary Registry to the permanent one.


Using this approach, update packages can easily be distributed, which will contain only new versions of *some* images.

**Note:** This update approach is not yet supported.


## In the works: HA for docker registry (not tested):


There are two endpoints for docker registry: one is read-write; the second one is read-only.


Since the Docker Registry is backed by a Ceph storage and only a single instance can have a volume mounted in read/write mode, additional instances mount this volume in read-only mode.


**Note:** minio may also be used as a registry backend.






# Persistent storage


## Persistent storage backend for TAP - Ceph


TAP 0.8 uses Ceph RADOS Block Device (RBD) to provide reliable, persistent, and distributed network attached storage for containers data storage.


When assigning roles to machines, ensure your follow either official guidance from the Ceph project [[http://docs.ceph.com/docs/master/start/hardware-recommendations/]] or follow TAP configuration recommendations.


In general, you should use a dedicated disk drive on each Ceph storage node.


### Resize available storage


As databases grow over time, the platform may need additional persistent disk space.


To add additional storage, follow the following procedure.


Downtime is (TODO YES OR NO) required to perform those tasks.


1. TODO
   1. screenshot 1
1. Step TODO


## Resource quotas


TAP sets static memory and CPU limits on every application it creates, to ensure stable and fair scheduling.


Use the following command to list current resource usage, which is listed per Kubernetes worker:


`$ kubectl describe node worker-0.kubernetes.cluster.local worker-1.kubernetes.cluster.local worker-2.kubernetes.cluster.local  (more worker node hostnames can follow)`






## Adding custom components to deployment automation

To extend TAP installation with your custom services that are not managed by Kubernetes, you simply have to follow general Ansible guidelines. Create a new Role in the `roles` directory and include its execution in the `site.yml` file. You can re-use existing machine group to assign role to, or create a new machine group. 


# Platform Backup and Restore


Assets that require backup:


* TAP deployment configuration files
   * $HOME/TAP/...
* Persistent Storage - Ceph
* Data on CDH cluster
* RR: We don’t need to backup K8s etcd… but there is a task for that: [[https://intel-data.atlassian.net/browse/DPNG-9295]] 


Having those in place allows you to perform a disaster recovery procedure by simply running the deployment script over a fresh set of machines.


When a fresh deployment reaches the end of the Ceph cluster deployment, it should ask you to manually perform a restore of Ceph data. Then you can continue.

As core components are being installed, they will get the same storage volume mounted.


For user applications, the volume-application mapping is stored on one of the core service ??? (missing word?). Application-handling components will use the volume restored from backup when restarting applications.







# Platform Upgrades


Currently, only full upgrades are supported


In future, TAP will support partial upgrades. Each upgrade will have a specific, step-by-step set of instructions.


In general, TAP uses Ansible to automate the deployment process. By its nature, Ansible playbooks (scripts) are idempotent. It means, that you can run particular playbook (script) multiple times and only changes will be applied. This behavior is very useful during the platform upgrade process.


The flow is limited to:

* Downloading and extracting a new TAP package.
* Apply configuration changes from previous a deployment to the extracted configuration file.
* Run the deployment again.


TODO FIXME: Platform upgrades functionality is planned for the next Sprints.

##Detailed procedure

TBD





# Maintenance


[ToDo] Describe safe/graceful platform shutdown and start procedures


TO DO:


1. Describe offline deployment procedures.

2. Describe building TAP from sources.


CEPH:

# Deployment: recommended Ceph configuration (in various TAP cluster layouts)

Ceph need to run 2 daemons - ceph-mon and ceph-osd; good practice is to have minimum 3 ceph-mon and 3 ceph-osd. 
Minimum Hardware recommendation:
* ceph-osd: 1x 32/64-bit i386, 1GB RAM per 1TB storage, 1 storage drive per deamon (SSD - better performance/higher cost, HDD lower performance/lower cost), 20GB root HDD
* ceph-mon: 1x 32/64-bit i386, 1GB RAM, 20GB root HDD.
Kubernetes-workers need to have installed ceph-client to use Ceph as persistent volume. 


## Minimal TAP cluster - evaluation only:
  * 1 VM with ceph-mon, ceph-osd - Can be installed on kubernetes-master.
  * Kubernetes-workers got ceph-client installed

## Playground TAP cluster:
  * 2 VM with ceph-mon and ceph-osd - Can be installed on kubernetes-masters
  * Kubernetes-workers got ceph-client installed

## Production TAP cluster:
  * Minimum 3VMs with 3x mons and 6x osds
  * Kubernetes-workers got ceph-client installed

# Deployment: configuration of Ceph components (disk space allocation mode - shared with OS vs exclusive, disk 
allocation quotas, how to use persistent storage in cloud deployments) [TODO]

# Maintenance:

## Health and performance monitoring 
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

## Free space monitoring
  * Log in to ceph-master node: `ssh ceph-master`. Afterwards get root privileges: `sudo -i`.
  * From now on you have access to ceph and RBD CLI.
  * To monitor space available on the cluster you should use `ceph -s`. You will get information: `space MB used, space MB allocated / space MB available`
  * To monitor space allocated/available on volumes attached to kubernetes containers use `rbd du`

## Allocated space extension/reduction

## Configuration & data backup procedure [TODO]
## Recovery procedure after Ceph cluster failure [TODO]
## Safe stop and start procedure [TODO]