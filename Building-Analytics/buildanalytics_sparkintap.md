---
title: Building Analytics
keywords: building analytics spark TAP
last_updated: 'January, 2017'
tags:
  - Building Analytics spark
summary: How to get started using open-source spark in TAP. 
sidebar: mydoc_sidebar
permalink: buildanalytics_sparktk.html
folder: mydoc
published: true
---

Apache Spark is a general engine for cluster scale computing. It provides APIs for multiple languages including Python, Scala, and SQL. This page explains how to use Spark in TAP.

## Getting started with Spark

The easiest way to get started with Spark on TAP is within a Jupyter notebook, as follows:

1. First, [create a Jupyter notebook](/Building-Analytics/Creating_Jupyter_Notebook_Instance.md).

2. Open your Jupyter instance and navigate to **examples/spark/README.ipynb**

![Accessing Readme files](/images/Build_Analytics_SparkScreen1.png)

3. The README notebook demonstrates how to create a SparkContext and some simple Spark code.

![Readme files in Jupyter Sample](/images/Build_Analytics_SparkScreen2.png)

The other example notebooks show how to use Spark dataframes, RDDs, streaming, SQL, and machine learning with K-Means and Linear Regression.

![Readme files in Jupyter Sample](/images/Build_Analytics_Spark_Screen3_New1.png)

>More information about Spark is available on the [Apache Spark website](http://spark.apache.org/)

###Accessing a terminal from Jupyter
1. From the Jupyter dashboard, select the **>New** button located in the upper right.

![Accessing a Terminal from Jupyter](/images/Build_Analytics_SparkScreen4.jpg) 

2. Select **>Terminal** from the sub menu to open a new terminal within Jupyter. 

![Jupyter Terminal](/images/Build_Analytics_SparkScreen5.jpg) 

You can enter Spark commands (spark-shell, spark-submit, etc.) in the terminal window.
