---
title: Enabling Hue
keywords: Enabling Hue
last_updated: 'November, 2016'
tags:
  - Enabling Hue
summary: Enabling Hue. 
sidebar: mydoc_sidebar
permalink: enabling_hue.html
folder: mydoc
published: true
---

## Enabling Hue

TAP uses Hue, which is an open-source Web interface, for analyzing data with Hadoop and Spark. For more information on HUE, see http://gethue.com. 

To enable HUE on TAP, follow these instructions:

1. To add a HUE service to the Cloudera cluster, note on which machines it was placed.

9. Create a new Elastic Load Balancing (ELB) instance in AWS:
    1. Forward port 443 HTTPS to port 8888 HTTPS.
    2. Should be on the same subnet as the NAT instance (one is sufficient).
    3. Fill in the private key, certificate, and the chain.
    4. Create a new security group allowing access to all on port 443.
    5. Perform a simple TCP health check.
9. Modify the security group on Cloudera to allow access from the ELB security group to _all_ ports. (This is suggested by AWS support; using only one port does *not* work).

9. Modify HUE instance settings on the Cloudera cluster:
    a. Enable HTTPS.
    b. Put the private key and certificate on the HUE server in a place reachable by HUE and chown hue:hue the files.
    c. Fill in the required paths in HUE configuration.

9. Add a CNAME domain to route53 for the created ELB.

