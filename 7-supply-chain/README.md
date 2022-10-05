---
title : "實驗 7 - 供應鏈準時交付預測（運輸與物流）"
weight : 20
---

::alert[開始實驗前請確保您已執行並完成 **先決條件** 中的步驟。]{type=warning}

## 議程 Agenda

1. [概述 Overview](#overview)
1. [將數據集上傳 S3 存儲桶](#s3)
1. [將數據導入 Canvas](#canvas)
1. [建構和訓練 ML 模型](#ml)
1. [使用模型生成預測 Predictions](#predictions)

## 概述 Overview

在本實驗中，您將擔任某物流行業公司的業務分析師角色，目標是要以 日數 為單位來預測貨件的預計到達時間。您將使用一個名為 **Shipping Logs Dataset** 的合成數據集，當中包含所有已交付貨件的完整運輸數據，包括預計到達時間、運輸優先級別、運輸承運商和運輸來源。數據集有大約 10000 行，12 個特徵列，而數據架構如下：

|ActualShippingDays	| 目標欄位 - 運送貨物所需的日數	|
|---	|---	|
|Carrier	| 運輸承運商 |
|YShippingDistance	| 出貨距離（Y軸） |
|XShippingDistance	| 出貨距離（X軸） |
|ExpectedShippingDays	| 預計運輸日數 |
|InBulkOrder	| 表示是否為批量訂單 |
|ShippingOrigin	| 運輸來源 |
|OrderDate	| 訂單日期 |
|OrderID	| 訂單編號 |
|ShippingPriority	| 運輸優先級別 |
|OnTimeDelivery	| 表示貨物是否準時送達 |
|ProductId	| 產品編號 |
|ComputerBrand	| 表示發貨的電腦品牌 |
|ComputerModel	| 表示發貨的電腦型號 |
|ScreenSize	| 表示屏幕尺寸 |
|PackageWeight	| 發貨包裹的重量 |

## 將數據集上傳 S3 存儲桶

第一步是下載我們將使用的數據集。這個數據集分為 2 個檔案，您可以通過以下鏈接下載：

1. :link[檔案-1]{href="/static/datasets/ProductDescriptions.csv" action=download}
2. :link[檔案-2]{href="/static/datasets/ShippingLogs.csv" action=download}

轉到 AWS 管理控制台，在控制台頂部的搜索框中尋找 **S3**，然後去到 **S3** 服務控制台。

![Image](/static/lab7/image1.png)

在 S3 控制台中，點擊 **sagemaker-studio-\*** 存儲桶。

![Image](/static/lab7/image2.png)

::alert[ **sagemaker-studio-\*** 在當初建立 SageMaker Studio domain 的時候，就已經自動建立。如果你參與 **Event Engine** 活動, 則講師會預先準備存儲桶。]

點擊 **Upload**。

![Image](/static/lab7/image3.png)

在上傳頁面上，拖放剛才下載的 2 個檔案，然後點擊頁面底部的 **Upload**。上傳完成後，您可以點擊右上角 **Close** 按鈕。現在，您應該看到上傳到存儲桶中的文件。

## 將數據導入 Canvas

返回到 Sagemaker Canvas。在左側選單上，您可以點擊第二個圖標，進入數據集部分，然後點擊 **Import** 按鈕。

![Image](/static/lab7/image4.png)

現在，選擇之前上傳到 **sagemaker-studio-\*** 存儲桶的數據集。

![Image](/static/lab7/image5.png)

現在，您可以通過在其左側複選框來選擇先前上傳的 2 個檔案（`ShippingLogs.csv` 和 `ProductDescriptions.csv`）。頁面底部將彈出兩個新的按鈕：**Preview all** 和 **Import Data**。讓我們選擇第一個。

![Image](/static/lab7/image6.png)
![Image](/static/lab7/image6a.png)

現在，您可預覽要導入的數據集的 100 筆資料。完成資料檢查，確定正確後，您可點擊 **Import Data**。

現在，點擊 **Join data**，然後將兩個貸款數據集檔案（`ShippingLogs.csv` 和 `ProductDescriptions.csv`）從屏幕左側拖放到右側。點擊連接符號，然後為兩個數據集選擇 `ProductId` 作為連接鍵。將連接類型保留為 **Inner**。最後結果應如下圖所示：

![Image](/static/lab7/image6b.png)

選擇 **Save \& Close**，然後點擊 **Import Data**；為數據集定義名稱，例如 `ConsolidatedShippingData`，然後點擊 **Import Data**。

## 建構和訓練 ML 模型

現在，讓我們通過點擊左邊選單上的第二個按鈕回到 **Models** 部分。

![Image](/static/lab7/image7.png)

點擊 **\+ New model**，並為您的模型輸入名稱，例如 `Shipping Forecast`，然後選擇 **Create**。

![Image](/static/lab7/image8.png)

如果這是您第一次建立 Canvas 模型，那麼您將看到一個彈出式歡迎，其中有關於如何通過 4 個簡單步驟建構您第一個模型的信息。您可以閱讀此信息，然後回到本實驗指南。

![Image](/static/lab7/image9.png)

在模型視圖中，您將看到四個選項卡，它們對應於創建模型並使用它來生成預測的四個步驟：**Select**，**Build**，**Analyze**，**Predict**。在第一個選項卡 **Select** 中，選擇我們之前已經處理好的 `ConsolidatedShippingData` 數據集。該數據集包括 12 欄位和大約 10k 筆數據。點擊底部的按鈕 **Select dataset**。

![Image](/static/lab7/image10.png)

Canvas 將自動移動到 **Build** 階段。在此選項卡中，選擇目標欄位，在我們的情況下是 `ActualShippingDays`。由於我們想要預測貨物到達客戶總共需要多少天，Canvas 將自動偵測這是 **numeric prediction** 問題（也稱為迴歸）。迴歸 是基於一個或多個其他變量或與其相關的屬性來估計目標變量的值。如果系統挑選的模型類型不正確，您也可以使用屏幕中心的鏈接 **Change type** 加以改變，最後應如下圖所示：

![Image](/static/lab7/image11.png)

在屏幕的下半部，您可以查看數據集的一些統計屬性，包括缺失和不匹配的值，獨特的值，平均值和中位數。以下是您可以看到的一些統計數據和信息：

* 欄位視圖為您提供所有列的列表、它們的數據類型和它們的基本統計信息，包括缺失和不匹配的值，獨特的值，平均值和中位數。這將有助於您設計一種策略來處理數據集中的缺失值：

![Image](/static/lab7/image12.png)

* 網格視圖為您提供每列值的圖形分佈和示例數據，讓您可以開始分析訓練模型的相關欄位：

![Image](/static/lab7/image13.png)

* 如果我們不想使用特定欄位，可以使用左邊的複選框進行檢查，取消特定欄位的勾選。訓練模型的時候，就不會涵蓋已經取消勾選的欄位。對於此次實驗，我們計劃剔除對模型訓練過程中沒有價值的欄位：`OrderID`（因為它是主鍵，沒有重要信息）； 出於同樣的原因，我們也會剔除 `ProductId`。

![Image](/static/lab7/image14.png)

* 從我們的探索性數據分析中，我們可以看到數據集中沒有太多缺失值，所以便不必處理缺失值。假如您看到自己的數據集的其中一些欄位有很多缺失值，您可以選擇過濾這些缺失值：

![Image](/static/lab7/image15.png)

探索了本節後，是時候訓練模型了！在建立完整的模型前，最好我們對於訓練模型的表現，有一個基本想法。一個 **Quick Model** 訓練模型和超參數組合的配對較少，優先考慮速度勝過準確性，尤其當我們希望驗證 訓練模型 能夠產生價值。請注意，**Quick Build** 不適合大於 50k 筆數的模型。現在，讓我們繼續點擊 **Quick build**。

![Image](/static/lab7/image16.png)

現在，我們大概需要等待 2 - 15 分鐘的時間讓 **Quick build** 完成模型的訓練。

![Image](/static/lab7/image17.png)

完成後，Canvas 將自動移動到 **Analyze** 選項卡，向我們展示快速訓練的結果：

![Image](/static/lab7/image18.png)

::alert[不用擔心以下截屏中顯示的數字是否與您的數字不同。機器學習在模型訓練過程中會引入一些隨機性，這將導致訓練產生出不同的結果。]{type=warning}

讓我們專注於第一個選項卡 **Overview**。這是向我們展示欄位影響 *Column impact*，或這些特徵，在預測目標時的重要性。在我們的案例中，**ExpectedShippingDays** 這個欄位對預測貨件預計到達時間的影響最大。

![Image](/static/lab7/image19.png)

在 **Scoring** 選項卡中，您可以看到表示 `ActualshippingDays` 的最佳擬合回歸線的圖。平均而言，模型預測與 *ActualShippingDays* 的實際值有 +/- 0.7 的誤差。用於數值預測的 **Scoring** 部分顯示了一條線，以表示模型的預測值與用於進行預測的數據的關係。模型預測的數值通常是 +/- RMSE（均方根誤差）值，而在線周圍的紫色區域的寬度則表示了 RMSE 範圍（代表預測值通常會落在該範圍內）。

![Image](/static/lab7/image20.png)

**Advanced metrics** 部分為希望深入了解其模型性能的用戶提供更詳細的信息。 Amazon SageMaker Canvas 數字預測的指標定義如下：

* R2 – 可以由輸入欄位解釋到的目標欄位差異的百分比
* MAE – 平均絕對誤差（Mean absolute error）。 平均而言，目標欄位的預測值與實際值相差 +/- {MAE}
* MAPE – 平均絕對百分比誤差（Mean absolute percent error）。平均而言，目標欄位的預測值與實際值相差 +/- {MAPE} %
* RMSE – 均方根誤差（Root Mean Square Error）。誤差的標準差（standard deviation）

下圖顯示了 殘差 或 誤差 的圖表。水平線表示誤差為 0 或完美預測；而藍點則表示了不同的誤差值，它們與水平線的距離代表了誤差值的大小程度。

![Image](/static/lab7/image21.png)

下圖顯示了誤差密度圖：

![Image](/static/lab7/image22.png)

現在，您有兩個選擇：

1. 您可以使用此模型運行一些預測，請選擇頁面底部的 **Predict** 按鈕；
1. 您可以使用 **Standard Build** 流程訓練此模型的新版本。這將花費更長的時間（大約 4 - 6 小時），但會產生更準確的結果。


假設我們對模型的準確率有信心，我們將選擇選項 1。如果我們沒有信心，那麼我們可以讓數據科學家來審查 Canvas 建構的模型，並通過選項 2 來提高潛在的準確率。

在這個實驗中，我們將選擇選項 1。假如您有更多的空餘時間，也可以選擇運行選項 2。

::alert[請注意，如果您要從 SageMaker Canvas 共享模型到 SageMaker Studio，您必須使用 **Standard Build** 來訓練模型。**Predictions** 並不需要**Standard Build**，但是它的性能和準確性會比經過完全訓練的模型低。]{type=warning}

## 使用模型生成預測 Predictions

現在完成模型的訓練，我們可以做一些預測。在 **Analyze** 頁面底部選擇 **Predict**，或選擇 **Predict** 選項卡。

點擊 **Batch prediction** 來進行批量預測，並選擇要用於批量生成預測的數據集 `ConsolidatedShipping`。接下來，選擇頁面底部的 **Generate predictions** 來生成預測。Canvas 將使用這個數據集來生成我們的預測。雖然在實際環境下我們在訓練和測試時並不會使用相同的數據集，但為了簡單起見，我們在這次實驗中將使用相同的數據集。

幾秒鐘後，預測完成。您可以點擊長的很像 "眼睛" 的圖標做預覽，或點擊 download 按鈕下載 CSV 文件。

![Image](/static/lab7/image23.png)

另外，您還可以選擇 **Single prediction** 來進行單次預測。這個適合模擬場景的應用（*what-if* scenarios），以及測試不同的欄位會如何影響模型的預測結果，例如：假如 `ShippingOrigin` 為 **Houston**，會否對預測的到達時間帶來影響？如果改變了 運輸承運商 或 包裹重量 又如何影響結果？

![Image](/static/lab7/image24.png)

----

**恭喜！** 您現在已經完成了實驗7。作為下一步，您可以：

1. 再次運行這個實驗，但建立 Standard Model以查看其模型的表現；
2. 選擇運行另一個實驗