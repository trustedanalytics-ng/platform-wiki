---
title: Data Ingestion
keywords: TAP data ingestion sql import scheduler
last_updated: 'October, 2016'
tags:
  - Data Ingestion
summary: Insert the summary paragraph here.  To edit the summary you must edit the meta data for this post. 
sidebar: mydoc_sidebar
permalink: dataingest_sqlimport_scheduler.html
folder: mydoc
published: true
---

The Job Scheduler allows you to import data from a SQL database into HDFS connected to TAP. Data can be imported in batch mode or by scheduling periodic, automatic updates.

### Import data from a SQL database
From the TAP console main menu, navigate to **Job Scheduler** and then  **Import data**.

![](/images/Step1.jpg)

TAP displays a form for you to fill out, starting with a job name, as shown below. Pick a unique name for your job.

![](/images/job-name.png)

* `JDBC URI` - You can enter the URI directly, or you can fill in the fields above the URI field to create the required schema for the jdbc URI: 
```
jdbc:driver://host:port/database_name
```
You can add optional JDBC URI parameters, for example, to turn on SSL for postgresql connection:
```
jdbc:postgresql://host:port/database_name?ssl=true&sslfactory=org.postgresql.ssl.NonValidatingFactory
```
(Please note that parameters are driver-specific, check database driver documentation for details)

![](/images/TAP_job-scheduler_jdbc-uri.png) 

* `Username` and `Password` - These are the credentials to connect to the data source
![](/images/TAP_job-scheduler_username-password.png)

* `Table` - This is the name of the database table to be imported into HDFS. 
![](/images/TAP_job-scheduler_table.png)

* `Destination dir` - This is the directory in the target HDFS where you will store the imported data. **Note:** Make sure you have write access rights to this directory.

* `Choose import mode` - There are 3 import modes available: `Append`, `Overwrite`, and `Incremental`
  * `Append` - Each import will fetch the whole table into a separate file. Results of previous imports will *not* be overwritten. Files on HDFS will have names in the pattern: `part-m-00000`, `part-m-00001`, and so on.
  * `Overwrite` - Each import will fetch the entire source table and overwrite results of the previous import, using part-m-00000 for the filename.
  * `Incremental` - The import will fetch records having, in a column identified by `Column name` parameter, values *not* lower than the value provided by the `Value` parameter. Each subsequent import will fetch records with values in the aforementioned column higher than the previously fetched values. For this purpose (identifying), we recommend using a numeric column, which is auto-incremented.  
    * `Column name` - The column from the database (unique numeric format), against which `Value` will be checked; used for unique identification of data to be imported.
    * `Value` - A reference value used to filter out records from the source database - only records with values (in a column identified by ‘Column name’) not smaller than this reference ‘Value’ will be imported.
 
![](/images/TAP_job-scheduler_import-mode-incremental.png)

* `Start time` - The start time of your job.
  * **Note:** When you enter a `Start time` prior to the current time, Oozie will try to “catch up” by executing jobs from the past.
* `End time` - The end time of your job. 
  * `End time` should always be later than `Start time`.
* `Frequency` - The frequency with which your job will be submitted.
* `Timezone` - The id of the time zone for the entered start and end time.

![](/images/TAP_job-scheduler_set-job-schedule.png)

### Job browser
Selecting **Job Scheduler** then **Job browser** from the TAP main menu allows you to view scheduled jobs. There are two tabs on the **Job browser** page: `Workflow jobs` and `Coordinator jobs`.

![](/images/TAP_job-scheduler_workflow-jobs.png)

* `Workflow jobs` - On this tab, you can see a list of workflow jobs. Workflow jobs represent imports from databases to HDFS. Click on `See details` to the right of a job name for additional information (example shown below).
![](/images/TAP_job-scheduler_details.png)
  * `Details` - This section provides additional information about the specified workflow job.
  * `See logs` - This section provides logs related to the specified workflow job.
You can kill the job by clicking on the **Kill** button.

* `Coordinator jobs` -  This tab contains configuration information and manages workflow jobs. Click on `See details`to the right of a job for additional information (example shown below).
![](/images/TAP_job-scheduler_coordinator-jobs_details.png)
  * `Details` - Additional information about the coordinator job.
  * `Started workflow jobs` - List of workflow jobs spawned by the coordinator job. Each workflow job on the list has a `See details` link, which will redirect you to the selected workflow job details.

![](/images/TAP_job-scheduler_coordinator-started-workflows.png)

## See also
[Job Scheduler FAQ](https://github.com/trustedanalytics/platform-wiki-0.7/wiki/Job-scheduler-faq)

[Installing custom database drivers](https://github.com/trustedanalytics/platform-wiki-0.7/wiki/Installing-custom-sqoop-database-drivers-for-Import-Data-Scheduler)
