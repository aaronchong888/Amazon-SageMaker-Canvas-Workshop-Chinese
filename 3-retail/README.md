# 實驗 3 - 銷售預測（零售電商）

> **Warning**
> 開始實驗前請確保您已執行並完成 **先決條件** 中的步驟。

> **Warning**
> 這個實驗大概需要 3 - 4 小時才能完成。

<br>

## 議程 Agenda

1. [概述 Overview](#概述-overview)
1. [前言 Forenote](#前言-forenote)
1. [將數據導入 Canvas](#將數據導入-canvas)
1. [建構和訓練 ML 模型](#建構和訓練-ml-模型)
1. [使用模型生成預測 Predictions](#使用模型生成預測-predictions)
1. [清理 Cleanup](#清理-cleanup)

<br>

## 概述 Overview

在本實驗中，您將擔任某電子商務公司銷售部門的業務分析師角色。您將使用零售商店的時間序列 歷史銷售數據 來構建一個模型，該模型可用於預測特定零售商店的銷售額。您將使用一個合成數據集，而數據架構如下：

| 欄位名稱 | 資料型別 |
| ----------- | ----------- |
| store | INT | 
| saledate | TIMESTAMP| 
| sales | DECIMAL |
| promo | INT (0 /1) |
| schoolholiday | INT (0/1) |

## 前言 Forenote

本實驗可以通過兩種方式進行，請選擇以下其中一種方法： 

### 1. Amazon S3

第一步是下載我們將使用的數據集。您可以到這裡下載：[檔案](/static/datasets/store_daily_sales_reduced.csv)

轉到 AWS 管理控制台，在控制台頂部的搜索框中尋找 **S3**，然後去到 **S3** 服務控制台。

![](/static/shared/search_s3.png)

在 S3 控制台中，點擊 **sagemaker-studio-\*** 存儲桶。

![](/static/shared/studio-bucket.png)

> **Warning**
> **sagemaker-studio-\*** 在當初建立 SageMaker Studio domain 的時候，就已經自動建立。如果你參與 **Event Engine** 活動，則講師會預先準備存儲桶。

點擊 **Upload**。

![](/static/shared/s3_upload.png)

在上傳頁面上，拖放剛才下載的 `store_daily_sales_reduced.csv` 檔案，然後點擊頁面底部的 **Upload**。上傳完成後，您可以點擊右上角 **Close** 按鈕。現在，您應該看到上傳到存儲桶中的文件。

### 2. Amazon Redshift

> **Warning**
> 注意：如果您是參與 AWS 講師指導活動並使用 **Event Engine** 提供的環境運行實驗，則 **無須** 按照這個部分的指示來創建 Amazon Redshift。您可以直接跳到 [將數據導入 Canvas](#將數據導入-canvas) 部分。

首先，您需要創建 Redshift，請下載此檔案：[CloudFormation 模板](/assets/lab3-module/redshift.yaml)，並按照以下步驟來創建 CloudFormation stack：

1. 轉到 **AWS CloudFormation** 服務控制台以建立堆疊（stack）： [鏈接](https://console.aws.amazon.com/cloudformation/home?#/stacks/create/template)
1. 在 **Specify template**（指定範本）頁面上，選擇 **Upload a template file**
1. 上傳剛才下載的檔案，然後點擊 **Next**
1. 指定堆疊名稱，然後點擊 **Next**
1. 在 **Review**（檢閱）頁面上，向下滾動到 **Capabilities** 部分，然後選擇並同意 "I acknowledge that AWS CloudFormation might create IAM resources"，最後點擊 **Create stack** 便完成

<br>

## 將數據導入 Canvas

返回到 Sagemaker Canvas。在左側選單上，您可以點擊第二個圖標，進入數據集部分，然後點擊 **Import** 按鈕。

![](/static/shared/import-data.png)

### 1. Amazon S3

現在，選擇之前上傳到 **sagemaker-studio-\*** 存儲桶的數據集。

![](/static/shared/import-from-s3-studio.png)

您可以通過在其左側複選框來選擇先前上傳的 `store_daily_sales_reduced.csv` 檔案。頁面底部將彈出兩個新的按鈕：**Preview all** 和 **Import Data**。讓我們選擇第一個。

![](/static/lab3/canvas-select-preview.png)

現在，您可預覽要導入的數據集的 100 筆資料。完成資料檢查，確定正確後，您可點擊 **Import Data**。

![](/static/lab3/canvas-preview.png)

### 2. Amazon Redshift

我們需要通過提供 IAM 憑證來創建與 Redshift 集群的連接，該憑證將授予我們訪問其中數據的權限。 讓我們首先單擊右側的 **Add connection** 按鈕，然後選擇 **Redshift**。

![](/static/shared/import-from-redshift.png)

在 **Add a new Redshift connection** 彈出頁面上：

- **Type** 選擇 IAM。
- **Cluster identifier** 輸入 `redshift-cluster-1`。
- **Database name** 輸入 `dev`。
- **Database user** 輸入 `awsuser`。
- **Unload IAM Role** 提供 Redshift IAM Role ARN，您可以通過進入 [IAM 管理控制台](https://console.aws.amazon.com/iamv2/home?#/roles) 來搜索 "CanvasImmDayRedshiftConnectorRole" 角色。
- **Connection name** 輸入任意名稱，例如：`redshiftconnection`。
- 最後，單擊 **Add connection** 按鈕。

![](/static/shared/new-redshift-connector-2.png)

我們已成功建立與 Redshift 集群的連接！您現在可以在頂部看到 `redshiftconnection` 按鈕，然後選擇 `public` schema 下的 `storesales` 表，再拖放到右側面板。

![](/static/lab3/import-data-redshift.png)

現在您已成功添加`storesales`表，並可以在頁面底部看到數據的預覽。如有需要，您可以通過將其他數據集與當前的表連接來加載其他數據集：您可以拖放一個新表，然後通過選擇 *left join* 或其他來更改連接選項。

對於這次實驗，我們並不會加入其他表格，但您可以在進行下一步之前探索此選項。完成數據集探索後，您可以單擊 **Import Data**。

![](/static/lab3/table-added.png)

最後，提供一個數據集名稱，例如 `store_sales_data`，然後再次單擊 **Import data** 按鈕。您將被重定向到 **Datasets** 頁面，在這裡您能夠看到您的數據集以及數據集當中的列數和行數。

![](/static/lab3/imported-data.png)

<br>

## 建構和訓練 ML 模型

現在，讓我們通過點擊左邊選單上的第二個按鈕回到 **Models** 部分。

![](/static/lab3/new-model.png)

點擊 **\+ New model**，並為您的模型輸入名稱，例如 `store_sales_forecast_model`，然後選擇 **Create**。

![](/static/lab3/create-new-model.png)

在 **Select dataset** 中，選擇 `store_sales_data` 數據集，並點擊底部的按鈕 **Select dataset**。

![](/static/lab3/select-dataset.png)

Canvas 將自動移動到 **Build** 階段。在此選項卡中，選擇目標欄位，在我們的情況下是 `sales`。Canvas 將自動偵測這是 **Time series forecasting** 問題（也稱為時間序列預測），然後點擊 **Configure** 。

![](/static/lab3/target-and-problem.png)

在 時間序列預測 配置頁面中，您需要設置以下資料：

- **Item ID column**：在數據集中，識別特定筆數的方式；對於這個案例，請選擇 `store`，因為我們計劃預測每家商店的銷售額
- **Group column**：如果您需要對上面的選擇進行邏輯分組，您可在此選擇該功能；對於這個案例，我們並沒有分組欄位，但在其他案例中，有可能會出現 "州"、"地區"、"國家" 或其他商店的分組。
- **Time stamp column**：選擇 `saledate`，即包含時間戳記的特徵；Canvas 需要的時間戳記格式為 `YYYY-MM-DD HH:mm:ss`（例如：`2022-01-01 01:00:00`）
- **Days**：輸入 `120`，表示我們需要預測未來 120 日的數值。

最後，點擊 **Save** 按鈕來完成設定。

![](/static/lab3/time-series-configuration.png)

現在配置已經完成，我們準備訓練模型。在撰寫本實驗時，SageMaker Canvas 並不支持使用 *Quick Build* 來建立時間序列預測模型，因此我們將使用 **Standard Build** 來模型訓練，大約需要 3 - 4 小時的時間來完成。

![](/static/lab3/start-standard-build.png)

![](/static/lab3/model-training.png)

<br>

## 使用模型生成預測 Predictions

完成後，Canvas 將自動移動到 **Analyze** 選項卡，您可看到平均預測準確度，以及欄位對於預測結果的影響。

現在，我們可以做一些預測。在 **Analyze** 頁面底部選擇 **Predict**，或選擇 **Predict** 選項卡。

![](/static/lab3/model-accuracy.png)

為了創建預測，您必須提供進行預測的日期範圍。然後，您可以為數據集中的 所有項目 或 特定項目 來生成預測。

在這次的實驗中，我們選擇 **Single item** 選項，然後從下拉表單中選擇任何數字為 特定項目 來生成預測。例如我們選擇項目 `5`，讓 Canvas 為我們生成預測，顯示平均預測、預測上限、和 預測下限。 Canvas 會為我們提供 預測界限 而不是單個預測點，讓您可選擇最適合的預測結果：透過選擇 **預測下限**，您可以減少資源浪費；也可以選擇 **預測上限**，讓您可以確保能滿足客戶需求。

對於生成的預測結果，您可點擊 **Download** 選單，將預測圖表下載為 圖檔，或將預測值下載為 CSV 檔案。

![](/static/lab3/forecasts.png)

<br>

## 清理 Cleanup

在實驗結束時，不要忘記刪除在本實驗開始時所創建的 Redshift 集群。請轉到 [CloudFormation 管理控制台頁面](https://console.aws.amazon.com/cloudformation)，然後刪除您創建的堆疊（stack）。

![](/static/lab3/delete-cfn-stack.png)

<br>

-------

**恭喜！** 您現在已經完成了實驗3。您可選擇要繼續運行的新實驗。