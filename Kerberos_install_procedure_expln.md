---
title: Kerberos Installation Procedure Explained
keywords: Kerboeros Installk
last_updated: 'November, 2016'
tags:
  - Jupyter
summary: Eplanation of Kerberos installation procedure. 
sidebar: mydoc_sidebar
permalink: Kerberos_install_procedure_expln.html
folder: mydoc
published: true
---

## Kerberos Installation Procedure Explained
The Ansible scripts that DP2 uses to boostrap Kerberos on the CDH cluster we are using, are loosely based on two things:
- The official documentation on [Cloudera Kerberos manual](http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cm_sg_intro_kerb.html)
- An open-source script for bootstraping the CDH cluster found [here](https://github.com/gdgt/cmapi/blob/master/cmxDeploy.py#L879-L928)

There are three main steps:

### Setting up the server 
The Ansible file can be found [here](https://github.com/trustedanalytics/platform-ansible/blob/master/roles/kerberos_base_server/tasks/main.yml).

It creates the the Kerberos server machine, installing the appropriate packages and creating configuration files from templates. After that, it creates the initial users.

### Setting up the clients 
The Ansible file can be found [here](https://github.com/trustedanalytics/platform-ansible/blob/master/roles/kerberos_base_common/tasks/main.yml).

It installs the client libraries on every machine and creates the `krb5.conf` file that is used by applications to get the Kerberos setting in a given environment.

### Enabling Kerberos support on the CDH cluster 
The Ansible file can be found [here](https://github.com/trustedanalytics/platform-ansible/blob/master/roles/cloudera_api_manager/tasks/kerberos.yml).

Since there is no one API call to enable Kerberos support, we enable it for each service and then generate the appropriate credentials. The whole process is available as a one-click wizard in the Cloudera Manager web interface.
