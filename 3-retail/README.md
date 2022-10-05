---
title : "實驗 3 - 銷售預測（零售電商）"
weight : 20
---

::alert[開始實驗前請確保您已執行並完成 **先決條件** 中的步驟。]{type=warning}

::alert[這個實驗大概需要 3 - 4 小時才能完成。]{type=warning}

## 議程 Agenda

1. [概述 Overview](#overview)
1. [前言 Forenote](#forenote)
1. [將數據導入 Canvas](#canvas)
1. [建構和訓練 ML 模型](#ml)
1. [使用模型生成預測 Predictions](#predictions)
1. [清理 Cleanup](#cleanup)

## 概述 Overview

In this lab, you will assume the role of a business analyst working for an e-commerce company, in the sales department. You will use historical time-series sales data for retail stores to build a model which can be used to forecast sales for a particular retail store. This dataset has been generated synthetically for the purpose of this lab. The data schema is as follows:

| Column Name | Data type |
| ----------- | ----------- |
| store | INT | 
| saledate | TIMESTAMP| 
| sales | DECIMAL |
| promo | INT (0 /1) |
| schoolholiday | INT (0/1) |

## 前言 Forenote

This lab can proceed in two ways: 

::::tabs{variant="container" groupId=canvasSource}
:::tab{id="s3" label="Amazon S3"}
If you're using going for this option, download the :link[CSV file]{href="/static/datasets/store_daily_sales_reduced.csv" action=download}.

Go to the AWS Management Console, search **S3** in the searchbox on top of your console, then go to **S3** service console.

![](/static/shared/search_s3.png)

In the S3 console, click on the **sagemaker-studio-\*** bucket.

![](/static/shared/studio-bucket.png)

::alert[The **sagemaker-studio-\*** bucket was created automatically when you created the SageMaker Studio domain in the **Prerequisites** section. If you follow the **Event Engine** track, the bucket was pre-provisioned by you instructor.]

Click **Upload**.

![](/static/shared/s3_upload.png)

On the Upload page, drag and drop the `store_daily_sales_reduced.csv` file you've just downloaded, then click **Upload** at the bottom of the page. Once the upload is complete, you can now click the top-right **Close** button. You should now see the file uploaded in your bucket.
:::
:::tab{id="redshift" label="Amazon Redshift"}

::alert[Note: if you're running the lab with these labs in environments provided by AWS trainers (Event Engine), you don't need to launch the Redshift cluster as described here. You can skip to the [Import the dataset in Canvas](#import-the-dataset-in-canvas) section.]{type=warning}

Set-up a connection to a Redshift cluster, and load data from there - you can create your Redshift cluster downloading this :link[CloudFormation template]{href="/assets/lab3-module/redshift.yaml" action=download} and then creating the CloudFormation stack by following the documentation on :link[Creating a stack on the AWS CloudFormation console]{href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html"}:

1. Follow this link to the :link[AWS CloudFormation console - Create Stack page]{href="https://console.aws.amazon.com/cloudformation/home?#/stacks/create/template"}.
1. In the **Specify template** portion of the page, choose **Upload a template file**
1. Choose the file you've just downloaded, hit **Next**
1. Provide a stack name of your choice, then hit **Next** twice.
1. In the review page, scroll down to the **Capabilities** section, and select "I acknowledge that AWS CloudFormation might create IAM resources". Hit **Create stack**.
:::
::::

## 將數據導入 Canvas

Go to the SageMaker Canvas tab created in the **Prerequisites** section. On the left menu, you can click the second icon to head to the Datasets section, then click the **Import** button.

![](/static/shared/import-data.png)

::::tabs{variant="container" groupId=canvasSource}
:::tab{id="s3" label="Amazon S3"}

Now, select the bucket where we've previously uploaded our dataset, the **sagemaker-studio-\*** bucket.

![](/static/shared/import-from-s3-studio.png)

You can now select the `store_daily_sales_reduced.csv` file uploaded previously by selecting the checkbox at its left. Two new buttons will pop-up at the bottom of your page: **Preview all** and **Import Data**. Let's choose the first one.

![](/static/lab3/canvas-select-preview.png)

You now face a 100-rows preview of the dataset you're looking to import. Once you're done checking that it's indeed the right one, you can click on **Import Data**.

![](/static/lab3/canvas-preview.png)

:::
:::tab{id="redshift" label="Amazon Redshift"}
We need to create a connection to the Redshift cluster, by providing the IAM credentials that will give us permission to access the data in it. Let's start by clicking the **Add connection** button on the right, then select **Redshift**.

![](/static/shared/import-from-redshift.png)

On the **Add a new Redshift connection** popup screen: 

- Select IAM for Type field. 
- Write your Redshift cluster identifier in the Cluster identifier field (`redshift-cluster-1` for the AWS-led workshop).
- Provide the Database name field (`dev` for the AWS-led workshop). 
- Provide the Database user field (`awsuser` for the AWS-led workshop). 
- Provide the Redshift IAM Role ARN for the Unload IAM Role field (you can search this role by going into the [IAM management console](https://console.aws.amazon.com/iamv2/home?#/roles), and searching for `CanvasImmDayRedshiftConnectorRole`). 
- Choose a name for the Connection name field - any name works here, we will use `redshiftconnection`
- Finally, click on the Add connection button.

![](/static/shared/new-redshift-connector-2.png)

The connection to the Redshift cluster is now set-up! You can see `redshiftconnection` button on the top. Select `storesales` table under the `public` schema and drag and drop to the right-hand side panel.

![](/static/lab3/import-data-redshift.png)

The table has now been added. You can see a preview of the data in the bottom section of the page. If you wanted, you could load other datasets by joining them with the current table: you can drag & drop a new table, then change the join options by choosing *left join* or other. For this workshop, we will not join other tables, but you can explore this option before moving to the next step. Once you're done exploring the dataset, you can click on **Import Data**.

![](/static/lab3/table-added.png)

Now, you can provide a dataset name, for example `store_sales_data`, and click again the **Import data** button. You are now redirected to the **Datasets** view and you should be able to see your dataset listed here, together with the number of columns and rows in the dataset.

![](/static/lab3/imported-data.png)
:::
::::

## 建構和訓練 ML 模型

Now that the dataset is imported, you can create a new model by going to the Models screen, and clicking on the **\+ New Model** button.

![](/static/lab3/new-model.png)

On the **Create new model** popup screen, write `store_sales_forecast_model` for the model name and click on the **Create** button.

![](/static/lab3/create-new-model.png)

On the **Select dataset** screen, select `store_sales_data` for the dataset and click on the **Select dataset** button.

![](/static/lab3/select-dataset.png)

On the next screen, you can configure the model for training. You can also select columns to see statistics of each column. Select `sales` for the **Target column** field. Canvas will automatically select **Time series forecasting** as the model type. Click on the **Configure** link.

![](/static/lab3/target-and-problem.png)

The Time Series Forecasting configuration popup screen, you are asked to provide a few information:
- The **items field**: how you identify you items in the datasets in a unique way; for this use case, select `store` since we are planning to forecast sales per store 
- The **group column**: if you have logical groupings of the items selected above, you can choose that feature here; we don't have one for this use case, but examples would be `state`, `region`, `country`, or other groupings of stores.
- The **time stamp field**: select `saledate` here, which is the feature that contains the time stamp information; Canvas requires data timestamp in the format `YYYY-MM-DD HH:mm:ss` (e.g.: `2022-01-01 01:00:00`)
- Write `120` in the **number of Days** field. 

Finally, click on the **Save** button.

![](/static/lab3/time-series-configuration.png)

Now that the configuration is done, we're ready to train the model. At the moment of writing, SageMaker Canvas does not support *Quick Build* for Time-Series Forecasting, therefore we will select the **Standard Build** option, and start training the model. Model will take around 3-4 hours to train.

![](/static/lab3/start-standard-build.png)

![](/static/lab3/model-training.png)

## 使用模型生成預測 Predictions

When the model training finishes, you will be routed to the **Analyze** tab. There, you can see the average prediction accuracy, and the column impact on prediction outcome. Click on the **Predict** button, to be brought to the **Predict** tab.

![](/static/lab3/model-accuracy.png)

In order to create forecast predictions, you have to provide first the date range for which the forecast prediction can be made. Then, you can generate forecast predictions for all the items in the dataset or a specific item. 

In our workshop, we choose the **Single item** option, and select any of the items from the item dropdown list. We choose number 5 here, and Canvas generates a prediction for our item, showing the average prediction, an upper bound and a lower bound. Canvas provides both results, since it is generally suggested to have bounds rather than a single prediction point so that you can pick whichever fits best your use case: you might want to reduce waste of resources by choosing to use the lower bound, or you might want to choose to follow the upper bound to make sure that you meet customer demand. 

For the generated forecast prediction, you can click on the **Download** dropdown menu button to download the forecast prediction chart as image or forecast prediction values as CSV file. 

![](/static/lab3/forecasts.png)

## 清理 Cleanup

At the end of your experimentation, don't forget to delete the Redshift cluster created at the beginning of this lab. In order to do so, head over to the [CloudFormation Management Console page](https://console.aws.amazon.com/cloudformation), and delete the stack you created.

![](/static/lab3/delete-cfn-stack.png)

-------

**Congratulations\!** You've now completed lab 3. You can now go ahead and choose a new lab to run.