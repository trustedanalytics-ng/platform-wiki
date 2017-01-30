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

Gearpump is a lightweight real-time big data streaming engine. Gearpump is inspired by recent advances in the Akka framework and a desire to improve on existing streaming frameworks.

Gearpump is incubated by the Apache Software Foundation. For more details about the Gearpump engine and to access the documentation, please refer to the [Gearpump Apache page](https://gearpump.apache.org).

## Using Gearpump in TAP

Within TAP, Gearpump can be used for high velocity data ingestion, stream data processing, online analytics, etc. Currently, a new Gearpump Cluster can be created on demand by a TAP user from the Marketplace or from the Data Science menu. 

### Deploying a Gearpump Application using TAP Console

To deploy an application, go to **Data Science** tab in menu and select **Apache Gearpump** option.

*updated screenshot to be inserted here*

Click inside the form field labeled **Instance name** and type a name for your instance. Select the button **Create new Apache Gearpump instance** to create a new instance. *The button is not available until a name has been provided in the Instance name field*.
After 30-60 seconds, refresh your browser screen. Your new Gearpump instance will be listed on the **Data Science > Gearpump** page

Select desired instance and click **Deploy application** link.

*updated screenshot to be inserted here*

In this step you have to select some variables.

1. Application JAR: select jar with '-with-dependencies' extension
2. Config file: file to provide settings for application.

To create config follow the [manual Gearpump deployment instructions](https://github.com/intel-data/ingestion-ws-kafka-gearpump-hbase/blob/master/gearpump/README_DEPLOY.md?raw=true).
Fill in the [example config file template](https://github.com/intel-data/ingestion-ws-kafka-gearpump-hbase/blob/master/gearpump/src/main/resources/example.conf?raw=true) using the output from previous manual deployment step.

*updated screenshot to be inserted here*

Click the **Go to dashboard** link to view the status of your application.

*updated screenshot to be inserted here*

### Deploying an Application using the Gearpump Dashboard

To deploy an application, go to **Applications** tab in menu and use **Submit** button.
 
![Data-Ingestion-application-list-submit.png.jpg](/images/Data-Ingestion-application-list-submit.png)

In the pop-up window, click to choose the JAR file containing application you want to run and click **Submit** again.

![Data-Ingestion-submit-application.png](/images/Data-Ingestion-submit-application.png)

You can also add configuration file or launch arguments from this page if needed.

In our example we use maven-assembly-plugin to build a jar with all required dependencies and to show main class in the manifest file (see pom.xml). This way main class can be automatically detected by Gearpump.

Your application should be now up and running.
![Data-Ingestion_application-list.png](/images/Data-Ingestion_application-list.png)


Select **Details** to see more information.

