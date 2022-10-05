---
title : "實驗 1 - 客戶流失預測（市場營銷）"
weight : 20
---

::alert[開始實驗前請確保您已執行並完成 **先決條件** 中的步驟。]{type=warning}

## 議程 Agenda

1. [概述 Overview](#overview)
1. [將數據導入 Canvas](#canvas)
1. [建構和訓練 ML 模型](#ml)
1. [使用模型生成預測 Predictions](#predictions)

## 概述 Overview

In this lab, we assume the role of a marketing analyst in the marketing department of a mobile phone operator. We have been tasked with identifying customers that are potentially at risk of churning. We have access to service usage and other customer behavior data, and want to know if this data can help explain why a customer would leave. If we can identify factors that explain churn, then we can take corrective actions to change predicted behavior, such as running targeted retention campaigns.

For our dataset, we use a synthetic dataset from a telecommunications mobile phone carrier. You can download it :link[here]{href="https://sagemaker-sample-files.s3.amazonaws.com/datasets/tabular/synthetic/churn.csv" action=download}. This sample dataset contains 5,000 records, where each record uses 21 attributes to describe the customer profile. The attributes are as follows:

| Feild      | Description |
| ----------- | ----------- |
| **State**      | The US state in which the customer resides, indicated by a two-letter abbreviation; for example, OH or NJ     |
| **Account Length**  | The number of days that this account has been active        |
| **Area Code** | The three-digit area code of the customer’s phone number        |
| **Phone** | The remaining seven-digit phone number       |
| **Int’l Plan** | Whether the customer has an international calling plan (yes/no)       |
| **VMail Plan** | Whether the customer has a voice mail feature (yes/no)       |
| **VMail Message** | The average number of voice mail messages per month       |
| **Day Mins** | The total number of calling minutes used during the day       |
| **Day Calls** | The total number of calls placed during the day       |
| **Day Charge** | The billed cost of daytime calls       |
| **Eve Mins, Eve Calls, Eve Charge** | The billed cost for evening calls       |
| **Night Mins, Night Calls, Night Charge** | The billed cost for nighttime calls       |
| **Intl Mins, Intl Calls, Intl Charge** | The billed cost for international calls       |
| **CustServ Calls** | The number of calls placed to customer service       |
| **Churn?** | Whether the customer left the service (true/false)       |

The last attribute, **Churn?**, is the attribute that we want the ML model to predict. The target attribute is binary, meaning our model predicts the output as one of two categories (True or False).

## 將數據導入 Canvas

Go back to the SageMaker Canvas tab created in the **Prerequisites** section. On the left menu, you can click the second icon to head to the Datasets section, then click the **Import** button.

![](/static/shared/import-data.png)

Now, select **Upload** option and browse the `churn.csv` file which we downloaded previously.

![](/static/lab1/import-from-local.png)

 In the pop-up at the bottom of your page: Select **Preview**.

![](/static/lab1/import-preview.png)

You now see a  preview of the dataset you're looking to import. Once you're done checking that it's indeed the right one, you can click on **Import Data**.

![](/static/lab1/final-import.png)

The import process takes approximately 10 seconds (this can vary depending on dataset size). When it’s complete, we can see the dataset is in Ready status.

![](/static/lab1/finaldataset.png)

After we confirm that the imported dataset is ready, we create our model.

## 建構和訓練 ML 模型

Now, let's head back to the **Models** section of the web page, by clicking the second button on the left menu.

![](/static/shared/canvas-models.png)

Click on **\+ New model**, and name your model `CustomerChurn`. 

![](/static/lab1/new-model.png)

In the Model view, you will see four tabs, which correspond to the four steps to create a model and use it to generate predictions: **Select**, **Build**, **Analyze**, **Predict**. In the first tab, **Select**, click the radio button to select the `churn.csv` dataset we've uploaded previously. This dataset includes 21 columns and 41K rows. Click the bottom button **Select dataset**.

![](/static/lab1/model-dataset.png)

Canvas will automatically move to the **Build** phase. In this tab, choose the target column, in our case `churn?`. Your marketing team has informed you that this column indicates  whether the customer left the service (true/false). This is what you want to train your model to predict. Canvas will automatically detect that this is a **2 Category** problem (also known as binary classification). If the wrong model type is detected, you can change it manually with the **Change type** link at the center of the screen.


![](/static/lab1/model-build.png)

We now validate some assumptions. We want to get a quick view into whether our target column can be predicted by the other columns. We can get a fast view into the model’s estimated accuracy and column impact (the estimated importance of each column in predicting the target column).

Select the **Preview** option

![](/static/lab1/model-preview.png)

This feature uses a subset of our dataset and only a single pass at modeling. For our use case, the preview model takes approximately 2 minutes to build.

![](/static/lab1/model-preview-1.png)

If you scroll down you will notice, the Phone and State columns have much less impact on our prediction. We want to be careful when removing text input because it can contain important discrete, categorical features contributing to our prediction. Here, the phone number is just the equivalent of an account number—not of value in predicting other accounts’ likelihood of churn, and the customer’s state doesn’t impact our model much.

 

![](/static/lab1/feature-1.png)

Let us remove the Phone and State columns, for this let us uncheck those features



![](/static/lab1/feature-2.png)

let’s run the preview again. Select **Update**

![](/static/lab1/feature-3.png)

As shown in the following screenshot, the model accuracy increased by 0.1%.

![](/static/lab1/feature-4.png)

As shown in the above screenshot, the model accuracy increased by 0.1%. Our preview model has a 95.9% estimated accuracy, and the columns with the biggest impact are Night Calls, Eve Mins, and Night Charge. This gives us an insight into what columns impact the performance of our model the most. Here we need to be careful when doing feature selection because if a single feature is extremely impactful on a model’s outcome, it’s a primary indicator of target leakage, and the feature won’t be available at the time of prediction. In this case, few columns showed very similar impact, so we continue to build our model.


For this Lab, we're planning on using all of the available features. You can come back to this step later and try to change some of the features to see the impact on the model training. 

Once you've explored this section, it's time to finally train the model! Before building a complete model.

Canvas offers two build options:

   * Standard build – Builds the best model from an optimized process powered by AutoML; speed is exchanged for greatest accuracy
   * Quick build – Builds a model in a fraction of the time compared to a standard build; potential accuracy is exchanged for speed.

![](/static/shared/canvas-quick-vs-standard.png)


 It is a good practice to have a general idea about the performances that our model will have by training a **Quick Model**. A quick model trains fewer combinations of models and hyper-parameters in order to prioritize speed over accuracy, especially in cases like ours where we want to prove the value of training an ML model for our use case. Note that quick build is not available for models bigger than 50k rows. Let's go ahead and click **Quick build**.

![](/static/lab1/model-building.png)

Now, we wait anywhere from 2 to 15 minutes. Since the dataset is small, this will take probably even less than 2 minutes. 

When the model building process is complete, the model predicted churn 95.9% of the time. This seems fine, but as analysts we want to dive deeper and see if we can trust the model to make decisions based on it. On the Scoring tab, we can review a visual plot of our predictions mapped to their outcomes. This allows us a deeper insight into our model.

Let's focus on the first tab, **Overview**. This is the tab that shows us the **Column impact**, or the estimated importance of each column in predicting the target column. In this example, the duration column has the most significant impact in predicting if a customer will churn. 

![](/static/lab1/model-status-1.png)

::alert[Don\'t worry if the numbers in the below images differ from yours. Machine Learning introduces some stochasticity in the process of training models, which can lead to different results to different builds.]{type=warning}

Canvas separates the dataset into training and test sets. The training dataset is the data Canvas uses to build the model. The test set is used to see if the model performs well with new data. The Sankey diagram in the following screenshot shows how the model performed on the test set. To learn more, you can check the ["Evaluating Your Model's Performance in Amazon SageMaker Canvas" section](https://docs.aws.amazon.com/sagemaker/latest/dg/canvas-evaluate-model.html) of the Canvas documentation, as well as the page for [SHAP Baselines for Explainability](https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-feature-attribute-shap-baselines.html).

Let’s go to Overview tab, to review the impact of each column. This information can help the marketing team gain insights that lead to taking actions to reduce customer churn. For example, we can see that both low and high CustServ Calls increase the likelihood of churn. The marketing team can take actions to prevent customer churn based on these learnings. Examples include creating a detailed FAQ on websites to reduce customer service calls, and running education campaigns with customers on the FAQ that can keep engagement up.

![](/static/lab1/model-overview-1.png)

To get more detailed insights beyond what is displayed in the Sankey diagram, business analysts can use a confusion matrix analysis for their business solutions. For example, we want to better understand the likelihood of the model making false predictions. We can see this in the Sankey diagram, but want more insights, so we choose Advanced metrics. We’re presented with a confusion matrix, which displays the performance of a model in a visual format with the following values, specific to the positive class—we’re measuring based on whether they will in fact churn, so our positive class is True in this example:

   True Positive (TP) – The number of True results that were correctly predicted as True

   True Negative (TN) – The number of False results that were correctly predicted as False

   False Positive (FP) – The number of False results that were wrongly predicted as True

   False Negative (FN) – The number of True results that were wrongly predicted as False

We can use this matrix chart to determine not only how accurate our model is, but when it is wrong, how often that might be and how it’s wrong.

The advanced metrics look good. We can trust the model result. We see very low false positives and false negatives. These are if the model thinks a customer in the dataset will churn and they actually don’t (false positive), or if the model thinks the customer will churn and they actually do (false negative). High numbers for either might make us think more on if we can use the model to make decisions.



![](/static/lab1/advanced-metrics-1.png)


Our model looks pretty accurate. We can directly perform an interactive prediction on the Predict tab, either in batch or single (real-time) prediction. 

Now, you have two options: 

1. you can use this model to run some predictions, by clicking on the button **Predict** at the bottom of the page;
2. you can create a new Version of this model to train with the **Standard Build** process. This will take much longer - about 4-6 hours - but will be much more accurate. 

For the sake of this lab, we will go forward with Option 1.

::alert[Note that training a model with **Standard Build** is necessary to share the model with a Data Scientist with the SageMaker Studio integration. **Predictions** do not require the full build, however they can lack in performances with respect to a fully-trained model.]{type=warning}

## 使用模型生成預測 Predictions

Now that the model is trained, let's use for some predictions. Select **Predict** at the bottom of the **Analyze** page, or choose the **Predict** tab.


Now, choose **Select dataset**

![](/static/lab1/batch-predict.png)

Choose the `churn.csv`. Next, choose **Generate predictions** at bottom of the page.

![](/static/lab1/batch-predict-ds.png)

 Canvas will use this dataset to generate our predictions. Although it is generally a good idea not to use the same dataset for both training and testing, we're using the same dataset for the sake of simplicity. After a few seconds, the prediction is done.

  You can click **View** to see the prediction.

  ![](/static/lab1/batch-predict-results.png)

   Optionally,click the download button to download a CSV file containing the full output. SageMaker Canvas will return a prediction for each row of data and the probability of the prediction being correct. 


![](/static/lab1/batch-predict-results-1.png)

You can also choose to predict one by one values, by selecting **Single prediction** instead of batch prediction. Canvas will show you a view where you can provide manually the values for each feature, and generate a prediction. This is ideal for situations like **what-if scenarios**: e.g.  What if Night charge is increased? 

![](/static/lab1/singleprediction.png)

**Congratulations\!** You've now completed lab 1. As next steps you can:

1. run this lab again, but building a Standard Model to see its performances;
1. choose another lab to run