---
title: Space Shuttle Example - Streaming Data
keywords: TAP streaming data use case example
last_updated: 'January, 2017'
tags:
  - examples space shuttle streaming
summary: Space shuttle streaming data example for spark-tk. 
sidebar: mydoc_sidebar
permalink: examples_spaceshuttle.html
folder: mydoc
published: true
---

## Use Case: Streaming Data - Space Shuttle Example

(Tested with TAP 0.7.3; needs updating for TAP 0.8.0)

This page describes the steps for deploying the spark-tk version of the space-shuttle-demo application to TAP. Most of the major steps have sub-steps. The link to the GitHub repository for this example is [https://github.com/trustedanalytics/space-shuttle-demo](https://github.com/trustedanalytics/space-shuttle-demo).

| Steps | Summary |
|:-----:| ------- |
| 0 | Prerequisites |
| 1 | Clone or Download the GitHub repository |  
| 2 | Uploading training data set to HDFS |  
| 3 | Creating the Jupyter instance and uploading Space Shuttle notebook |  
| 4 | Connecting to TAP server and generating the model |  
| 5 | Creating the scoring engine |  
| 6 | Creating instances of additional services (InfluxDB, Zookeeper, and Gateway) |  
| 7 | Building and uploading the central app |  
| 8 | Sending data to Gateway and Kafka |  


## Step 0: Prerequisites  
The following prerequisites are needed to successfully complete this example:
- An IDE environment such as IntelliJ IDE Community Edition (or Eclipse)
- Oracle JDK  1.8
- Apache maven
- Cloud Foundry CLI  

See the [*Sample environment variables for Windows*](#environment_windows) information at the end of this page when using Windows.


## Step 1: Clone or Download GitHub repository  
Clone/download [https://github.com/trustedanalytics/space-shuttle-demo](https://github.com/trustedanalytics/space-shuttle-demo) to your local system.
>New to GitHub?  Cloning instructions are [https://help.github.com/articles/cloning-a-repository/](https://help.github.com/articles/cloning-a-repository/).


## Step 2: Uploading training data set to HDFS  

A. Login to the TAP console and navigate to **Data catalog > Submit Transfer**.

B. Select the **input type: Local path**.

C. Find and select the sample training data file, which can be found here: `space-shuttle-demo-master/sparktkmodelgenerator/train-data.csv`

D. Enter a data set title in the **Title** field.

E. Choose a **Category** if you want keep things organized going forward. (**Other** is the default.)

F. Click the **Upload** button.

G. When the upload is completed, click **OK** in the dialog box. Then click the **Data catalog > Data sets** tab. Your data set is listed.

H. Click on the name of the data set to see detailed information. (You will need the URI to the data set later.) 


## Step 3: Creating Jupyter instance and uploading Space Shuttle notebook  

A. In the TAP console, navigate to **Data Science > Jupyter**.

B. If a Jupyter instance already exists, skip this step.  If *no* Jupyter instance exists, enter a notebook **Instance Name** and click the **Create a new Jupyter instance** button. This takes less than a minute to complete.

C. Click the **App Url** link to login to your Jupyter instance.

D. In Jupyter, click the (white) **Upload** button, then navigate to the Space Shuttle notebook file, which can be found here: `space-shuttle-demo-master/sparktkmodelgenerator/sparktk_shuttle_model_generator.ipynb`

E. Select the file and click the second (blue) **Upload** button.

F. In Jupyter, click on the `sparktk_shuttle_model_generator.ipynb` file to open the file.


## Step 4: Connecting to TAP server and generating the model

In this step, you modify the generic notebook script, then run it to create the model.

A. In the TAP Console, navigate to the **Data catalog > Data sets** tab and then click on the name of the data set to show detailed information. Copy the URI for the data set for use in the next step.

B In Jupyter, paste this URI over the default name for data set `ds=` in the third cell.  

```
    ds=”your_dataset_uri_here”
```  

C. In Jupyter, select the second cell, then on the Jupyter menu, select **Cell > Run Cell, Select Below** (same as the ![](https://github.com/trustedanalytics/platform-wiki/blob/master/wikiImages/Jup_Run_Sel_Next_Icon.png) icon) to run the script in the cell.

D. When execution of the cell is finished, the asterisk in the `In [*]` text on the right side of the cell is replaced by the cell number. Work through Cells 3 through 6,  one at a time, using **Cell > Run, Highlight Next**. Make sure you wait for the current cell to complete before moving to the next one.

E. When you start Cell 7, you will be prompted for:  
- The URL of your TAP server. Paste the domain name of the TAP server into the field and press **Enter**. (Example: `ontap07.demo-gotapaas.com`.)  
- Your TAP user name. Type your user name and press **Enter**.  
- Your TAP password. Type your password and press **Enter**.  

F. In the next cell, you are prompted for the organization number, for example **[1-2]**. Enter the organization number and press **Enter**.
>If there is only one organization, you will *not* be prompted.

G. Work through the remaining cells, one at a time, using **Cell > Run, Highlight Next**. Make sure you wait for the current cell to complete before moving to the next one.

H. When finished, the trained model will appear in the **Data catalog > Data sets** tab. Filename = `spaceshuttleSVMmodel.mar`.
>The Model ARchive format (.mar files) is a new model format for use with spark-tk.

I. Click on the model name to see detailed information. Copy the targetURI for the model for use in Step 5.


## Step 5: Create the scoring model  

To create the Scoring Engine service instance:

A. From the TAP console, navigate to **Services > Marketplace**. Select the "Scoring Engine for Spark-tk" service.

B. Type the name `space-shuttle-scoring-engine`

C. Click **+ Add an extra parameter** and add the URI to your model (from Step 4I) as part of this key/value pair:

```
    key: uri value: hdfs://path_to_your_model_here  
```

D. Click the **Create new instance** button.  

>This may take a minute or two. You can monitor the process via the **Event Log** tab.


## Step 6: Create instances of additional services  

Create the following required service instances (if they do *not* exist already):  

- InfluxDB (recommended name: `space-shuttle-db`)  
- Zookeeper (recommended name: `space-shuttle-zookeeper`, create as **Shared** plan)  
- Gateway (recommended name: `space-shuttle-gateway`)  

These services can all be found in **Services > Marketplace**.

The application will connect to these service instances using Spring Cloud Connectors.

>If you use the recommended names for the required service instances, they will be bound automatically with the application when it is pushed to Cloud Foundry. Otherwise, service instance names will need to be either: (1) adjusted in the 'manifest.yml' file manually or (2) removed from 'manifest.yml' and bound manually after the application is pushed to Cloud Foundry. These two options are *not* covered here.

## Step 7: Building and uploading central app

A. Open a command line window and change directory to the location where you cloned your GitHub repository.  


>If you list/dir the contents of this directory, you should see a `pom.xml` file. Maven uses the contents of the `pom.xml` file to perform the build.

B Using your IDE, locate the following Java file and open it:  

```
    src\main\java\org\trustedanalytics\serviceinfo\ScoringEngineServiceInfoCreator.java  
```  

C. Change the name of the `SCORING-ENGINE-ID`:  

* Before:  `scoring-engine`  
* After:  `scoring-engine-spark-tk`  


D. In the CLI, invoke maven to create a Java package:

```
    mvn package  
```

>If you created service instances with different names than were recommended, edit the auto-generated `manifest.yml` file to adjust the names of service instances in the services section to match those that you have created. These files are located at: `src/cloudfoundry/`. You can also remove the services section and bind them manually later. You may also want to change the application host/name.  


>The Linux `cp` command will fail in a Windows environment. In this case, manually copy two manifest (YAML) files from the subdirectories to the current working directory. Locations and files are:

```
    target/classes/manifest.yml
    target/classes/manifest-mqtt.yml
```

E. Log into the Cloud Foundry CLI using the following command:

```
    cf login -a your_analytics_platform_uri_here
``` 

>Copy the TAP instance URI, then replace `console` at the beginning with `api`. Examples:  

* Before:  `console.ourtap07.teamanalytics.com`  
* After:  `api.ourtap07.teamanalytics.com`  


F. When prompted, enter your:
* TAP user name
* TAP password
* Organization number (if there are multiple organizations)  


G. Push the application to the platform using the Cloud Foundry (CF) command:  

    `cf push`  


>You can see this login and push command sequence in the TAP Console **App Development** tab.

CF uses the `manifest.yml` file to upload the application to TAP. If you used the recommended names earlier, or edited the `manifest.yml` file in the previous step, the application will be started, but *no* data is being sent to it yet.


>If you removed the services section from `manifest.yml`, the application will *not* be started yet. First, bind the required service instances (`cf bind-service`) to the application and then restage (`cf restage`) it.  


>If the default application name (`space-shuttle`) is already uploaded to any organization on the TAP platform, you will get an error message stating that the host name is taken. In this case, open the `manifest.yml` file and rename the application. Then attempt the `cf push` again.


## Step 8: Sending data to Kafka  

To send data to the Kafka queue (integrated into TAP), through the Gateway service instance, you can either:  

- Change your directory to `client`, then push `space_shuttle_client` from the client directory to the TAP space with the existing Gateway instance, using this command:

```
     cf push space-shuttle-client
```  
 
>CF uses the `manifest.yml` file in the client directory.  

*or*  

- Use the Python file `space_shuttle_client.py`, locally passing the Gateway URL as a parameter.
Data will start to be sent to the application.

In the TAP Console, navigate to **Applications** and click the link for the Space Shuttle Client. The space shuttle image appears, followed by the anomaly data grid, and, finally, by anomaly data.

>It may take several minutes for the data grid to appear, and up to several hours for anomaly data to appear. Good data, however, is being sent immediately. You can check on data flow to the application via the following command in your CLI window:  
`cf logs space-shuttle`  
Use your actual application name if you chose something other than the default name.  

##<a name="environment_windows">Sample environment variables for Windows:

###PATH system environment variable:

- C:\CF\CloudFoundry
- C:\Program Files\Java\jdk1.8.0_112\bin
- C:\maven\apache-maven-3.3.9\bin

###JAVA_HOME system environment variable:

- C:\Program Files\Java\jdk1.8.0_112\bin

