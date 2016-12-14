---
title: Application Deployment
keywords: application deployment on TAP
last_updated: 'October, 2016'
tags:
  - Application Deployment
summary: Insert the summary paragraph here.  To edit the summary you must edit the meta data for this post. 
sidebar: mydoc_sidebar
permalink: appdev_deployapp.html
folder: mydoc
published: true
---

## Deploying Your First Application

If you are new to deploying applications on the Trusted Analytics platform (TAP), these steps will guide you through the process.

### Tools

Aside from having an IDE to code your application, the primary tool to deploy applications is the CLI (Command Line Interface) for TAP. You can prepare your environment and install this tool by visiting [this page](Contributing-to-TAP/contributing_devenvironment.md) or going to TAP Console >> Main navigation panel >> App Development.

### Sample Application

A helpful application is "Spring Music", which you can clone from github here: https://github.com/trustedanalytics/spring-music. This application requires little knowledge of the platform and also features hooks for Redis, a good service to be familiar with.

The TAP team has also built a sample application for working with this platform and pulling data from HDFS. While more advanced, this is also a good application example to clone and explore: https://github.com/trustedanalytics/dataset-reader-sample. We've created a full workflow around this application as it explains in more detail how to build applications around data ingested into TAP.

### Preparing the Application

#### Assembly

In order to deploy Java applications to the Trusted Analytics Platform, you must assemble them using a tool like Maven. You can download Maven here: https://maven.apache.org/download.cgi. If you are new to Maven, here are the steps to better understand the building/assembly process: https://maven.apache.org/guides/getting-started/.

#### Prepare run.sh script which will start an application:

```
#!/usr/bin/env bash

exec java -jar your-app.jar
```

#### Create an archive containing all files loosely:

```
tar czvf your-app.tar.gz ./*
```

### Pushing the Application

Use TAP CLI to push the application to your TAP instance.

```
$ tap-cli login <Your TAP instance Endpoint> <username> <password>
  Authenticating...
  Authentication succeeded
$ tap-cli push your-app.tar.gz
```

### Viewing the Application and its Instance in TAP

With the application successfully pushed, you can now view it on your instance of TAP. In the TAP Console navigate to **Applications** to view the application's instance. Clicking the URL will load the application. Clicking **See details** will reveal the profile of the application with controls to start|stop|restart it.

![TAP_console_spring_music_app_instance.png](/images/TAP_console_spring_music_app_instance.png)

### Binding a Service

The TAP service broker, which is found in **Marketplace** in the TAP Console, makes it easy to add popular services like MongoDB to your application. This highly extensible aspect of TAP allows you to easily integrate instances of data stores as well as 3rd party services to your application. You can also programmatically add services, including making your own server logs available as a service.

Continuing with the "Spring Music" application example above, you can easily create and bind an instance of the **Redis** service to your application. This is helpful since, up until now, the application has only been storing entries in its own memory. Should the application need to be restaged, it will lose all its data. 

From the TAP Console, navigate to **Marketplace** >> **Services** and browse to the **Redis** service. Click on it and in the service's profile, add a new instance with name "redis".

![TAP_console_add_redis_instance.png](/images/TAP_console_add_redis_instance.png)

In the TAP Console, return to **Applications** and view your "spring-music" profile by clicking the **See details** link. Click on the **Bindings** tab. You should see the "redis" instance listed in the **Available service instances** to the right. Click **Bind**. You will now be prompted to click **Restage** so the application will be restaged and bound to the **Redis** instance you created. Once restaged, all data entered will be stored in the **Redis** instance, thus creating a persistent data store for the application.

After binding to the **Redis** service all it's environment variables, like hostname or password, will be visible for spring-music as environment variables. 

![TAP_console_bind_service.png](/images/TAP_console_bind_service.png)

That's all, you have deployed your first application on TAP. If you would like to know more about binding, go to the [Binding your Applications on TAP](/Application-Development/appdev_bindingapps.md) documentation page.
