---
title: Using Data Catalog to download files from S3
keywords: Data Catalog S3
last_updated: 'January, 2017'
tags:
  - Data Catalog S3
summary: Using Data Catalog to download files from S3. 
sidebar: mydoc_sidebar
permalink: data_catalog_s3.html
folder: mydoc
published: true
---

## Using Data Catalog to download files from S3
1. Log on to Amazon* S3 and go into the bucket that contains your file.

9. Right-click your file and select **Download**.

    ![AWS S3 Download](/images/Data_Ingestion_Download_AWS_S3_1.png)

9. After clicking **Download**, a pop-up window should open. Right-click the **Download** button in the window and choose **Copy link address**.

    ![Copy Link Address](/images/Data_Ingestion_Download_AWS_S3_2.png)

1. Paste this URL into **Link** field on the TAP Console **Data Catalog > Submit Transfer** tab. 

    ![TAP Data Catalog Submit Transfer](/images/Data_Ingestion_Download_AWS_S3_v8.png)

**Note:** This link is only valid for a certain amount of time (usually 3 minutes or less). Transfers can take longer, but you must submit the transfer request within this time.
