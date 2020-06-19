---
title: deploy-spark-dotnet-app-on-databricks
date: 2020-06-19 10:35:53
tags:
categories:
description:
---

# Deploy Spark .net job on databricks

We can run .Net for Apache Spark jobs on Databricks, but it is not what we do for Python or Scala jobs. For the Python or Scala jobs, we can just start the Notebook task for them. But for Spark .net job, we need to use the "spark-submit" or "set jar" tasks. 

## How

### Triggered by Azure Data Factory

#### Deploy using Set Jar

1. Generate a **Databricks access token** for Azure Data Factory to access. 

   1.1 In Databricks workspace, select your user profile in the upper right, and select  **User Settings**.

   ![image-20200617105838264](/images/image-20200617105838264.png)

   1.2 Select **Generate New Token** under the **Access Tokens** tab.

   ![image-20200617105851323](/images/image-20200617105851323.png)

   1.3 Save the access token for later use in creating a Databricks linked service. Usually save it in Azure Key Vault for security.

2. Navigated to the **Pipelines** page on Azure Data Factory, create a new pipeline, search for **Databricks** activities, drag the Jar task to panel.

   ![image-20200617104722883](/images/image-20200617104722883.png)

3. In the **Jar** activity Demo, updates the paths and settings as needed. **Databricks linked service** should be created using **access token** generated on Databricks previously. Remember to add init script for cluster settings.

   ![image-20200617104957870](/images/image-20200617104957870.png)

4. Check the **Jar** settings. **Main class name** is org.apache.spark.deploy.dotnet.DotnetRunner. **Parameters** will pass to the main class. it must have your app publish.zip and your app name as the first two parameters. The rest parameters are what your app need. **Append libraries** are microsoft-spark-2.4.x-0.10.0.jar on dbfs.

   ![image-20200617113228776](/images/image-20200617113228776.png)

5. Click **Debug** to run a test for the current pipeline.

6. Save the newly added pipeline by click **Publish all**.

### Directly on databricks

#### 1. Deploy using Spark-submit

1. Navigate to Databricks Workspace and create a job. Select Task as spark-submit. Set job parameters。

   ![image-20200617113422955](/images/image-20200617113422955.png)

2. When configure Cluster, need to add init script located on DBFS (databricks filesystem).

   ![image-20200616164936163](/images/image-20200616164936163.png)

3. select **Run Now** to test the job. Once the job's cluster is created, your Spark job will be submitted.

   ![image-20200617104003375](/images/image-20200617104003375.png)

#### 2. Deploy using Set Jar

We can also use the **Jar** task to deploy on databricks. The settings should be the same with the one [triggered by Azure Data Factory](#deploy-using-set-jar).

## Reference

https://docs.microsoft.com/en-us/dotnet/spark/how-to-guides/databricks-deploy-methods

https://docs.microsoft.com/en-us/azure/data-factory/solution-template-databricks-notebook

https://docs.microsoft.com/en-us/azure/data-factory/transform-data-databricks-jar

