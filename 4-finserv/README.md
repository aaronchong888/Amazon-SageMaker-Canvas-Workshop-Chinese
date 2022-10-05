# 實驗 4 - 貸款違約預測（金融服務）

> **Warning**
> 開始實驗前請確保您已執行並完成 **先決條件** 中的步驟。

<br>

## 議程 Agenda

1. [概述 Overview](#概述-overview)
1. [將數據集上傳 S3 存儲桶](#將數據集上傳-s3-存儲桶)
1. [將數據導入 Canvas](#將數據導入-canvas)
1. [建構和訓練 ML 模型](#建構和訓練-ml-模型)
1. [使用模型生成預測 Predictions](#使用模型生成預測-predictions)

<br>

## 概述 Overview

在本實驗中，您將擔任某銀行業公司的業務分析師角色。您將使用金融服務領域著名的數據集： [Lending Club Loan Dataset](https://www.kaggle.com/datasets/wordsforthewise/lending-club)，而目標是要預測您的客戶是否會償還貸款。該數據集包含 2007 - 2011 年發放的所有貸款的完整貸款數據，包括當前貸款狀態和最新的還款信息。該數據集是由 [Kaggle](https://www.kaggle.com) 根據 `CC0: Public Domain` 許可發佈，允許用於任何用途。數據集有大約 40,000 行，21 個特徵列，而數據架構如下：

| **欄位名稱**       | **描述**     | 
| :------------- | :---------- | 
|`loan_status`| 貸款的當前狀態（目標欄位）|
|`loan_amount`| 借款人申請的貸款金額。假如信貸部門在某個時間點減少了貸款金額，它將反映在此數值中| 
|`funded_amount_by_investors`| 投資者在該時間點為該貸款承諾的總金額| 
|`term`| 貸款的還款期數。數值以月份為單位，只能是 36 或 60|
|`interest_rate`| 貸款利率|
|`installment`| 假如貸款生效，借款人每月應支付的款項|
|`grade`| LC 指定的貸款等級|
|`sub_grade`| LC 指定的貸款子等級|
|`employment_length`| 就業年期。數值介於 0 和 10 之間，其中 0 表示不到一年，而 10 則表示十年或更長時間|
|`home_ownership`| 借款人在登記時提供的房屋所有權狀態。 數值為：RENT, OWN, MORTGAGE, OTHER|
|`annual_income`| 借款人在登記時提供的自報年收入|
|`verification_status`| 表示收入是否已通過 LC 驗證、未驗證 或 收入來源已驗證|
|`issued_amount`| 貸款獲得資金的月份|
|`purpose`| 借款人為貸款申請提供的類別|
|`dti`| 借款人每月總債務支付總額（不包括抵押貸款和申請的信用證貸款）除以借款人自我報告的月收入計算的比率|
|`earliest_credit_line`| 借款人最早報告的信用額度開通的月份|
|`inquiries_last_6_months`| 過去 6 個月的查詢次數（不包括汽車和抵押查詢）|
|`open_credit_lines`| 借款人信用檔案中未結信用額度|
|`derogatory_public_records`| 不良公共記錄的數量|
|`revolving_line_utilization_rate`| 循環線利用率，或借款人使用的信貸量相對於所有可用的循環信貸|
|`total_credit_lines`| 當前在借款人信用檔案中的信用額度總數|

<br>

## 將數據集上傳 S3 存儲桶

第一步是下載我們將使用的數據集。這個數據集分為 2 個檔案，您可以通過以下鏈接下載：

1. [檔案-1](/static/datasets/loans-part-1.csv)
1. [檔案-2](/static/datasets/loans-part-2.csv)

轉到 AWS 管理控制台，在控制台頂部的搜索框中尋找 **S3**，然後去到 **S3** 服務控制台。

![](/static/shared/search_s3.png)

在 S3 控制台中，點擊 **sagemaker-studio-\*** 存儲桶。

![](/static/shared/studio-bucket.png)

> **Warning**
>  **sagemaker-studio-\*** 在當初建立 SageMaker Studio domain 的時候，就已經自動建立。如果你參與 **Event Engine** 活動, 則講師會預先準備存儲桶。

點擊 **Upload**。

![](/static/shared/s3_upload.png)

在上傳頁面上，拖放剛才下載的 2 個檔案，然後點擊頁面底部的 **Upload**。上傳完成後，您可以點擊右上角 **Close** 按鈕。現在，您應該看到上傳到存儲桶中的文件。

<br>

## 將數據導入 Canvas

返回到 Sagemaker Canvas。在左側選單上，您可以點擊第二個圖標，進入數據集部分，然後點擊 **Import** 按鈕。

![](/static/shared/import-data.png)

現在，選擇之前上傳到 **sagemaker-studio-\*** 存儲桶的數據集。

![](/static/shared/import-from-s3-studio.png)

現在，您可以通過在其左側複選框來選擇先前上傳的 2 個檔案（`loans-part-1.csv` 和 `loans-part-2.csv`）。頁面底部將彈出兩個新的按鈕：**Preview all** 和 **Import Data**。讓我們選擇第一個。

![](/static/lab4/canvas-select-preview.png)

現在，您可預覽要導入的數據集的 100 筆資料。完成資料檢查，確定正確後，您可點擊 **Import Data**。

![](/static/lab4/canvas-preview.png)

現在，點擊 **Join data**，然後將兩個貸款數據集檔案（`loans-part-1.csv` 和 `loans-part-2.csv`）從屏幕左側拖放到右側。兩個數據集將會自動連接，並出現一個帶有紅色感嘆號的連接符號。點擊連接符號，然後為兩個數據集選擇 **id** 作為連接鍵。將連接類型保留為 **Inner**。最後結果應如下圖所示：

![](/static/lab4/join-datasets.png)

選擇 **Save \& Close**，然後點擊 **Import Data**；為數據集定義名稱，例如 `loans-training-data`，然後點擊 **Import Data**。

<br>

## 建構和訓練 ML 模型

現在，讓我們通過點擊左邊選單上的第二個按鈕回到 **Models** 部分。

![](/static/shared/canvas-models.png)

點擊 **\+ New model**，並為您的模型輸入名稱，例如 `Loan Prediction`，然後選擇 **Create**。

![](/static/lab4/new-loans-model.png)

如果這是您第一次建立 Canvas 模型，那麼您將看到一個彈出式歡迎，其中有關於如何通過 4 個簡單步驟建構您第一個模型的信息。您可以閱讀此信息，然後回到本實驗指南。

![](/static/shared/canvas-first-model-popup.png)

在模型視圖中，您將看到四個選項卡，它們對應於創建模型並使用它來生成預測的四個步驟：**Select**，**Build**，**Analyze**，**Predict**。在第一個選項卡 **Select** 中，選擇我們之前已經處理好的 `loans-training-data` 數據集。該數據集包括 23 欄位和 40k 筆數據。點擊底部的按鈕 **Select dataset**。

![](/static/lab4/image7.png)

Canvas 將自動移動到 **Build** 階段。在此選項卡中，選擇目標欄位，在我們的情況下是 `loan_status`。Canvas 將自動偵測這是 **3+ category prediction** 問題（也稱為多類分類）。如果系統挑選的模型類型不正確，您也可以使用屏幕中心的鏈接 **Change type** 加以改變。

![](/static/lab4/image12.png)

在屏幕的下半部，您可以查看數據集的一些統計屬性，包括缺失和不匹配的值，獨特的值，平均值和中位數。以下是您可以看到的一些統計數據和信息：

* 欄位視圖為您提供所有列的列表、它們的數據類型和它們的基本統計信息，包括缺失和不匹配的值，獨特的值，平均值和中位數。這將有助於您設計一種策略來處理數據集中的缺失值：

![Machine Learning Life Cycle](/static/lab4/image8.png)

* 網格視圖為您提供每列值的圖形分佈和示例數據，讓您可以開始分析訓練模型的相關欄位：

![Machine Learning Life Cycle](/static/lab4/image9.png)

* 如果我們不想使用特定欄位，可以使用左邊的複選框進行檢查，取消特定欄位的勾選。訓練模型的時候，就不會涵蓋已經取消勾選的欄位。對於此次實驗，我們計劃剔除對模型訓練過程中沒有價值的欄位：`id`（因為它是主鍵，沒有重要信息）； 出於同樣的原因，我們也會剔除 `employer_title`，`grade`，`sub_grade`。

![Machine Learning Life Cycle](/static/lab4/image10.png)

* 從我們的探索性數據分析中，我們可以看到數據集中沒有太多缺失值，所以便不必處理缺失值。假如您看到自己的數據集的其中一些欄位有很多缺失值，您可以選擇過濾這些缺失值：

![Machine Learning Life Cycle](/static/lab4/image11.png)

探索了本節後，是時候訓練模型了！在建立完整的模型前，最好我們對於訓練模型的表現，有一個基本想法。一個 **Quick Model** 訓練模型和超參數組合的配對較少，優先考慮速度勝過準確性，尤其當我們希望驗證 訓練模型 能夠產生價值。請注意，**Quick Build** 不適合大於 50k 筆數的模型。現在，讓我們繼續點擊 **Quick build**。

![](/static/shared/canvas-quick-vs-standard.png)

現在，我們大概需要等待 2 - 15 分鐘的時間讓 **Quick build** 完成模型的訓練。

![Machine Learning Life Cycle](/static/lab4/image13.png)

完成後，Canvas 將自動移動到 **Analyze** 選項卡，向我們展示快速訓練的結果：

![](/static/lab4/loans-analyze.png)

> **Warning**
> 不用擔心以下截屏中顯示的數字是否與您的數字不同。機器學習在模型訓練過程中會引入一些隨機性，這將導致訓練產生出不同的結果。

讓我們專注於第一個選項卡 **Overview**。這是向我們展示欄位影響 *Column impact*，或這些特徵，在預測目標時的重要性。在我們的案例中，**Credit History** 這個欄位對預測客戶是否會償還貸款金額的影響最大。

![](/static/lab4/loans-impact.png)

在 **Scoring** 選項卡中，您可以看到表示正確與錯誤預測值的分佈圖：

![](/static/lab4/loans-prediction-dist.png)

**Advanced metrics** 部分為希望深入了解其模型性能的用戶提供更詳細的信息。在我們的案例中，您會看到模型的準確率約為 82.9%。在機器學習中，模型準確率的定義為正確預測的數量除以預測的總數。混淆矩陣（Confusion Matrix）中的藍色框代表模型針對用於測試模型的數據集子集做出的正確預測。

當然，您會更感興趣的是如何衡量模型對違約者的預測程度。在這個例子中，模型正確預測了 75 名借款人將償還貸款，這在矩陣中表示為真陽性 (TP)。

而該模型錯誤地預測了約 1000 名借款人將被扣除全額付款，這被稱為假陰性（FN）。在機器學習中，用來衡量這一點的比率是 TP / (TP + FN)，這稱為召回率。**Advanced metrics** 頁面計算並顯示出此模型的召回率為 35%。如果您有興趣了解更多關於混淆矩陣（Confusion Matrix）的信息，請參閱[本文](https://towardsdatascience.com/understanding-confusion-matrix-a9ad42dcfd62)。

![](/static/lab4/loans-advanced-metrics.png)

現在，您有兩個選擇：

1. 您可以使用此模型運行一些預測，請選擇頁面底部的 **Predict** 按鈕；
1. 您可以使用 **Standard Build** 流程訓練此模型的新版本。這將花費更長的時間（大約 4 - 6 小時），但會產生更準確的結果。

在這個實驗中，我們將選擇選項 1。假如您有更多的空餘時間，也可以選擇運行選項 2。

> **Warning**
> 請注意，如果您要從 SageMaker Canvas 共享模型到 SageMaker Studio，您必須使用 **Standard Build** 來訓練模型。**Predictions** 並不需要**Standard Build**，但是它的性能和準確性會比經過完全訓練的模型低。

<br>

## 使用模型生成預測 Predictions

現在完成模型的訓練，我們可以做一些預測。在 **Analyze** 頁面底部選擇 **Predict**，或選擇 **Predict** 選項卡。

點擊 **Batch prediction** 來進行批量預測，並選擇要用於批量生成預測的數據集。

![Machine Learning Life Cycle](/static/lab4/predict-batch-select.png)

在這次的實驗中，我們將使用與之前相同的數據集 `loans-training-dataset`。而在實際的生產環境中，請確保您使用的預測數據集應該要與訓練數據集不同。

![Machine Learning Life Cycle](/static/lab4/predict-batch-dataset.png)

幾秒鐘後，預測完成。您可以點擊長的很像 "眼睛" 的圖標做預覽，或點擊 download 按鈕下載 CSV 文件。

另外，您還可以選擇 **Single prediction** 來進行單次預測。這個適合模擬場景的應用（*what-if* scenarios），以及測試不同的欄位會如何影響模型的預測結果。

![Machine Learning Life Cycle](/static/lab4/image19.png)

<br>

-------

**恭喜！** 您現在已經完成了實驗4。您可選擇要繼續運行的新實驗。