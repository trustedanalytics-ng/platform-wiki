---
title: Streaming Data Ingestion
keywords: TAP data ingestion
last_updated: 'January, 2016'
tags:
  - Data Ingestion
summary: Using the Gearpump engine to ingest streaming data. 
sidebar: mydoc_sidebar
permalink: dataingest_streamgearpump.html
folder: mydoc
published: true
---
## Introduction

Gearpump is a lightweight real-time big data streaming engine using Akka, it can also be used as an underlying platform for other general distributed applications, like distributed cron job, distributed log collection. Gearpump is inspired by recent advances in the Akka framework and a desire to improve on existing streaming frameworks. Gearpump draws from a number of existing frameworks including MillWheel, Apache Storm, Spark Streaming, Apache Samza, Apache Tez and Hadoop YARN while leveraging Akka actors throughout its architecture.

Gearpump is incubated by the Apache Software Foundation. For more details about the Gearpump engine and to access the documentation, please refer to the [Gearpump Apache page](https://gearpump.apache.org).

## Using Gearpump in TAP

Within TAP, Gearpump can be used for high velocity data ingestion, stream data processing, online analytics, etc. Currently, a new GearPump Cluster can be created on demand by a TAP user from the Marketplace. TAP console also includes Gearpump section under the Data Science menu. Users are able to list service instances there. They can also deploy Gearpump applications directly from TAP console.

### Deploying a Gearpump Application using TAP Console

To deploy an application, go to **Data Science** tab in menu and select **Apache Gearpump** option.

*updated screenshot to be inserted here*

If there is no instances or you want to have a new one, type the name and click **Create new Apache Gearpump instance** button.

*updated screenshot to be inserted here*

Select desired instance and click **Deploy application** link.

*updated screenshot to be inserted here*

In this step you have to select some variables.

1. Application JAR: select jar with '-with-dependiences' extension
2. Config file: file to provide settings for application.

To create config file use [this page](https://github.com/intel-data/ingestion-ws-kafka-gearpump-hbase/blob/master/gearpump/README_DEPLOY.md?raw=true) to fill [file template](https://github.com/intel-data/ingestion-ws-kafka-gearpump-hbase/blob/master/gearpump/src/main/resources/example.conf?raw=true)

*updated screenshot to be inserted here*

Your application should be now up and running. Click **Go to dashboard** link to check it. 

*updated screenshot to be inserted here*

### Deploying an Application using the Gearpump Dashboard

When you have running GearPump cluster, it is ready to accept your applications.

To deploy an application, go to **Applications** tab in menu and use **Submit** button.
 
![Data-Ingestion-application-list-submit.png.jpg](/images/Data-Ingestion-application-list-submit.png)

In the pop-up window, choose the JAR file containing application you want to run: and click **Submit** again.

![Data-Ingestion-submit-application.png](/images/Data-Ingestion-submit-application.png)

You can also add configuration file or launch arguments if needed.

In our example we use maven-assembly-plugin to build a jar with all required dependencies and to show main class in the manifest file (see pom.xml). This way main class can be automatically detected by Gearpump.

Your application should be now up and running.
![Data-Ingestion_application-list.png](/images/Data-Ingestion_application-list.png)


Click **Details** to see more information.

Now, you can see many basic details, including up time, number of processors, number of active executors, user data, actor path, and advanced details like Message Send Throughput, Message Receive Throughput and Average Message Receive Latency as well.

