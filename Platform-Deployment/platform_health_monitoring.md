---
title: Platform Health Monitoring
keywords: Platform Health Monitoring
last_updated: 'November, 2016'
tags:
  - Platform Health Monitoring
summary: Platform Health Monitoring. 
sidebar: mydoc_sidebar
permalink: platform_health_monitoring.html
folder: mydoc
published: true
---

## Platform Health Monitoring
TAP monitoring architecture consists of two main elements: 
  * A server in the Cloud that runs a web interface and sends change notifications.
  * A proxy that is deployed onsite and reports to the central server.
We are using Zabbix version 2.4 due to its automatic configuration capabilities.

##### Monitoring architecture
![Monitoring Architecture](/images/TAP_Monitoring_Zabbix_v7.png)

## Central server

Currently the only central server supported is an internal Trusted Analytics. Exporting the configuration for external parties to create their own is a WIP.

## Proxy

The proxy server configuration is bundled with each Trusted Analytics installation and is completely automatic.

## Monitoring components

### Base infrastructure monitoring (host accessibility)

Hosts are checked for common problems, such as a lack of disk space or CPU overload.

*Status*: Done.

### Monitoring of BOSH components

Cloud Foundry infrastructure is monitored via BOSH. This includes disk space/CPU on those machines.

*Status*: Planning.

### Monitoring of Cloud Foundry internals

Cloud Foundry components are monitored via REST API tests. This includes organizational quota, etc.

*Status*: In review, may need extending.

### Monitoring of the crucial applications for an environment via REST API

Crucial applications are monitored via REST API tests.

*Status*: Work in progress.

### Monitoring of the Cloudera services

Cloudera services are monitored via REST API tests.

*Status*: In review, may need extending.

