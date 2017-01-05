---
title: Application Deployment
keywords: binding applications to services application deployment on TAP
last_updated: 'January, 2017'
tags:
  - Application Deployment
summary: Easily integrate instances of data stores as well as 3rd party services to your application. 
sidebar: mydoc_sidebar
permalink: appdev_bindingapps.html
folder: mydoc
published: true
---

## Binding a Service to your Application

The TAP service broker, which is found in **Marketplace** in the TAP Console, makes it easy to add popular services like MongoDB to your application. This highly extensible aspect of TAP allows you to easily integrate instances of data stores as well as 3rd party services to your application. 

An instance of a service must be created in order to bind your application to that service. See [Creating a new Service Instance](/Platform-Marketplace/marketplace_createinstance.md) for instructions.

1. Navigate to **Applications** from the main menu.
1. Search for the application you wish to bind and select the **See Details>>** link on the far right of the application row.
![appdeploy_binding_screen1.jpg](/images/appdeploy_binding_screen1.jpg)
1. Select the **Bindings** tab to view the services already bound to the application and a list of services available to bind to the application. 
![appdeploy_binding_screen2.jpg](/images/appdeploy_binding_screen2.jpg)
1. Select the **Bind** button for the service you want to bind to the application.
1. Select the **Restage>>** button to restage the application.  

## Binding Using TAP CLI

You can accomplish the same process as above using the TAP CLI.  Visit the [TAP CLI documentation](/Reference-Documents/reference_cli.md) page for details. 
