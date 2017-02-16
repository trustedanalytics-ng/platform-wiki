Spark-tk is an analytics toolkit library that is compatible with Apache Spark. It provides APIs for Python and Scala. This page explains how to use Spark in TAP.

>Visit the [https://github.com/trustedanalytics/spark-tk](https://github.com/trustedanalytics/spark-tk) for additional information.

## Getting started with Spark-tk

The easiest way to get started with Spark-tk on TAP is within a Jupyter notebook, as follows:

1. [Create a Jupyter notebook](/images/BuildAn_GS_Sparktk_Start_v8_Screen1.PNG.md).

2. Open your Jupyter instance and navigate to **jupyter-default-notebooks/notebooks/examples/tklibs/sparktk/README.ipynb**  
  
    ![Accessing Readme files](/images/BuildAn_GS_Sparktk_Start_v8_Screen1.PNG)  
  
3. The README notebook demonstrates how to create a TkContext for Spark-tk and some simple Spark code.

![Readme files in Jupyter Sample](/images/BuildAn_GS_Sparktk_Start_v8_Screen2.PNG)

The other example notebooks show how to use Datacatalog, Frame, Latent Dirichlet Allocation, and Logistic Regression with Spark-tk.

![Readme files in Jupyter Sample](/images/BuildAn_GS_Sparktk_Start_v8_Screen3.PNG)


>More information about Spark is available on the [Apache Spark website](http://spark.apache.org/)

###Accessing a terminal from Jupyter
1. From the Jupyter dashboard, select the **New** button located in the upper right.

    ![Accessing a Terminal from Jupyter](/images/BuildAn_GS_Sparktk_Start_v8_Screen4.PNG) 

2. Select **Terminal** from the sub menu to open a new terminal within Jupyter. 

![Jupyter Terminal](/images/BuildAn_GS_Sparktk_Start_v8_Screen5.PNG)  

You can enter CLI commands (spark-shell, spark-submit, etc.) in the terminal window.

##Troublshooting Tips

### Q: I am using spark-tk and want to save files/export models to my local file system instead of HDFS. How do I do that?

The SparkContext created by `TkContext` follows the system's current Spark configuration. If your system defaults to HDFS, but you want to use a local file system instead, include `use_local_fs=True` when creating your `TkContext`, as follows:  
  
      tc = sparktk.TkContext(use_local_fs=True)  
  
