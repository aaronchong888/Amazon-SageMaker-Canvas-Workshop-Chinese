# 實驗 6 - 機器故障預測（製造業）

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

在本實驗中，您將擔任某大型製造業公司的業務分析師角色。作為業務分析師，您的維護團隊要求您協助預測常見故障。他們為您提供了一個歷史數據集，其中包含了與特定故障類型相關的特徵數據，並希望您能預測將來會發生哪種故障。故障類型包括：*No Failure*, *Overstrain* 或 *Power Failures*。

我們在本實驗中所使用的數據集為：["AI4I 2020 Predictive Maintenance Dataset Data Set"](https://archive.ics.uci.edu/ml/datasets/AI4I+2020+Predictive+Maintenance+Dataset) (available on the UCI Machine Learning Repository - Dua, D. and Graff, C. (2019). UCI Machine Learning Repository [http://archive.ics.uci.edu/ml]. Irvine, CA: University of California, School of Information and Computer Science.)。而數據架構如下：

| 欄位名稱 | 資料型別 | 描述 | 
| ----------- | ----------- | ----------- |
| UID | INT | 唯一標識符，範圍從 1 到 10000 | 
| productID | STRING | 由字母 L、M 或 H 組成，作為產品質量變體的低、中和高，以及變體特定的序列號 | 
| type | STRING | 與 productID 關聯的首字母，僅由 L、M 或 H 組成 | 
| air temperature [K] | DECIMAL | 空氣溫度，以 Kelvin 為單位表示 |
| process temperature [K] | DECIMAL | 精確控制的目標溫度以確保特定產品的質量，以 Kelvin 為單位表示 | 
| rotational speed [rpm] | DECIMAL | 旋轉速度，以 每分鐘轉數 為單位表示 | 
| torque [Nm] | DECIMAL | 扭矩，以 Newton meters 為單位表示 |
| tool wear [min] | INT | 工具磨損，以 分鐘 為單位表示 |
| failure type (target) | STRING | 故障類型：*No Failure*, *Overstrain* 或 *Power Failures*（目標欄位） |

<br>

## 將數據集上傳 S3 存儲桶

第一步是下載我們將使用的數據集。您可以到這裡下載：:link[檔案]{href="/static/datasets/maintenance_dataset.csv" action=download}.

轉到 AWS 管理控制台，在控制台頂部的搜索框中尋找 **S3**，然後去到 **S3** 服務控制台。

![](/static/shared/search_s3.png)

在 S3 控制台中，點擊 **sagemaker-studio-\*** 存儲桶。

![](/static/shared/studio-bucket.png)


> **Warning**
>  **sagemaker-studio-\*** 在當初建立 SageMaker Studio domain 的時候，就已經自動建立。如果你參與 **Event Engine** 活動, 則講師會預先準備存儲桶。

點擊 **Upload**。

![](/static/shared/s3_upload.png)

在上傳頁面上，拖放剛才下載的 `maintenance_dataset.csv` 檔案，然後點擊頁面底部的 **Upload**。上傳完成後，您可以點擊右上角 **Close** 按鈕。現在，您應該看到上傳到存儲桶中的文件。

<br>

## 將數據導入 Canvas

返回到 Sagemaker Canvas。在左側選單上，您可以點擊第二個圖標，進入數據集部分，然後點擊 **Import** 按鈕。

![](/static/shared/import-data.png)

現在，選擇之前上傳到 **sagemaker-studio-\*** 存儲桶的數據集。

![](/static/shared/import-from-s3-studio.png)

您可以通過在其左側複選框來選擇先前上傳的 `maintenance_dataset.csv` 檔案。頁面底部將彈出兩個新的按鈕：**Preview all** 和 **Import Data**。讓我們選擇第一個。

![](/static/lab6/canvas-dataset-preview.png)

現在，您可預覽要導入的數據集的 100 筆資料。完成資料檢查，確定正確後，您可點擊 **Import Data**。

<br>

## 建構和訓練 ML 模型

現在，讓我們通過點擊左邊選單上的第二個按鈕回到 **Models** 部分。

![](/static/shared/canvas-models.png)

點擊 **\+ New model**，並為您的模型輸入名稱，例如 `Predictive Machine Maintenance`，然後選擇 **Create**。

![](/static/lab6/pred-machine-maintenance-model.png)

如果這是您第一次建立 Canvas 模型，那麼您將看到一個彈出式歡迎，其中有關於如何通過 4 個簡單步驟建構您第一個模型的信息。您可以閱讀此信息，然後回到本實驗指南。

![](/static/shared/canvas-first-model-popup.png)

在模型視圖中，您將看到四個選項卡，它們對應於創建模型並使用它來生成預測的四個步驟：**Select**，**Build**，**Analyze**，**Predict**。在第一個選項卡 **Select** 中，選擇我們之前已經上載的 `maintenance_dataset.csv` 數據集。 該數據集包括 9 欄位和 10k 筆數據。點擊底部的按鈕 **Select dataset**。

![](/static/lab6/canvas-model-dataset-select.png)

Canvas 將自動移動到 **Build** 階段。在此選項卡中，選擇目標欄位，在我們的情況下是`Failure Type`。維護團隊表示，此欄位代表了根據現有機器的歷史數據所看到的常見故障類型，這便是我們想要訓練模型來進行預測的目標。

Canvas 將自動偵測這是 **3+ category prediction** 問題（也稱為多類分類）。如果系統挑選的模型類型不正確，您也可以使用屏幕中心的鏈接 **Change type** 加以改變。

![](/static/lab6/canvas-model-build.png)


> **Warning**
> 請注意，這個數據集對於 "No Failure"（無故障）類的數據是高度不平衡的。雖然 Canvas 和其使用的 AutoML 功能可以部分處理數據集的不平衡，但這可能會導致一些性能偏差。作為本實驗後的額外下一步，您可以查看有關如何通過 Amazon SageMaker Data Wrangler 等服務來平衡數據集。

在屏幕的下半部，您可以查看數據集的一些統計屬性，包括缺失和不匹配的值，獨特的值，平均值和中位數。如果我們不想使用特定欄位，可以使用左邊的複選框進行檢查，取消特定欄位的勾選。

探索了本節後，是時候訓練模型了！在建立完整的模型前，最好我們對於訓練模型的表現，有一個基本想法。一個 **Quick Model** 訓練模型和超參數組合的配對較少，優先考慮速度勝過準確性，尤其當我們希望驗證 訓練模型 能夠產生價值。請注意，**Quick Build** 不適合大於 50k 筆數的模型。現在，讓我們繼續點擊 **Quick build**。

![](/static/shared/canvas-quick-vs-standard.png)

現在，我們大概需要等待 2 - 15 分鐘的時間讓 **Quick build** 完成模型的訓練。由於數據集很小，因此可能不需要等到2分鐘。完成後，Canvas 將自動移動到 **Analyze** 選項卡，向我們展示快速訓練的結果，這裡的分析估計我們的模型大概有 99.2% 的預測準確率。

讓我們專注於第一個選項卡 **Overview**。這是向我們展示欄位影響 *Column impact*，或這些特徵，在預測目標時的重要性。在我們的案例中，**duration** 這個欄位對預測將發生什麼類型的故障的影響最大。

![](/static/lab6/canvas-model-analyze1.png)

> **Warning**
> 不用擔心以下截屏中顯示的數字是否與您的數字不同。機器學習在模型訓練過程中會引入一些隨機性，這將導致訓練產生出不同的結果。

讓我們轉到 **Scoring** 選項卡，您可以看到表示模型的預測值相對於實際值的分佈圖，而根據預測結果，大部分情況也屬於 **無故障**（"No Failure"）類別。假如您想了解有關 Canvas 如何使用 SHAP 基線為機器學習帶來可解釋性的詳細資料，您可以查看 ["Evaluating Your Model's Performance in Amazon SageMaker Canvas" section](https://docs.aws.amazon.com/sagemaker/latest/dg/canvas-evaluate-model.html) 和 [SHAP Baselines for Explainability](https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-feature-attribute-shap-baselines.html)。

![](/static/lab6/canvas-model-analyze2.png)

我們可以點擊 **Advanced metrics** 部分更深入了解模型的性能及更詳細的信息。

您會看到頁面顯示出一個矩陣，而在機器學習中，這稱為混淆矩陣（Confusion Matrix）。

混淆矩陣默認顯示為 "No Failure"，而您可以從最右側的下拉列表選擇並查看其他兩種故障類型 "Overstrain Failure" 和 "Power Failure" 的詳細資料。

在機器學習中，模型的準確性定義為正確預測的數量除以預測的總數。藍色框表示模型針對具有已知結果的測試數據子集做出的正確預測。在默認情況下，在總共 `1986` 的預測中有 `1923` 個正確的 "No Failure" 預測，因此準確率為 `99%`。

當然，您會更感興趣的是模型對 **故障** 的預測程度。對於 **Overstrain Failure** 類別，在總共 `38` 的預測中有 `32` 個正確的預測，因此準確率為 `84%`。而對於 **Power Failure** 類別，在總共 `19` 的預測中有 `16` 個正確的預測，因此準確率為 `84%`。

在機器學習中，我們也可以用 **召回率** 來衡量模型的性能，其定義為 TP / (TP + FN)。

![](/static/lab6/canvas-model-analyze3-class_default-nofailure.png)

現在，您有兩個選擇：

1. 您可以使用此模型運行一些預測，請選擇頁面底部的 **Predict** 按鈕；
1. 您可以使用 **Standard Build** 流程訓練此模型的新版本。這將花費更長的時間（大約 1 - 2 小時），但會產生更準確的結果。

由於我們正在嘗試預測故障，而模型的準確率為 84%，因此我們有信心能使用該模型來識別可能的故障，而選擇選項 1。如果我們沒有信心，那麼我們可以讓數據科學家來審查 Canvas 建構的模型，並通過選項 2 來提高潛在的準確率。

> **Warning**
> 請注意，如果您要從 SageMaker Canvas 共享模型到 SageMaker Studio，您必須使用 **Standard Build** 來訓練模型。**Predictions** 並不需要**Standard Build**，但是它的性能和準確性會比經過完全訓練的模型低。

<br>

## 使用模型生成預測 Predictions

現在完成模型的訓練，我們可以做一些預測。在 **Analyze** 頁面底部選擇 **Predict**，或選擇 **Predict** 選項卡。

![](/static/lab6/canvas-selectdataset-predict.png)

現在，點擊 **Select dataset**，然後選擇 `maintenance_dataset.csv`。接下來，選擇頁面底部的 **Generate predictions** 來生成預測。Canvas 將使用這個數據集來生成我們的預測。雖然在實際環境下我們在訓練和測試時並不會使用相同的數據集，但為了簡單起見，我們在這次實驗中將使用相同的數據集。

幾秒鐘後，預測完成。您可以點擊長的很像 "眼睛" 的圖標做預覽，或點擊 download 按鈕下載 CSV 文件。 SageMaker Canvas 將會為每行數據返回一個 預測值 以及 預測正確的概率。

![](/static/lab6/canvas-selectdataset-predict.png)

![](/static/lab6/canvas-predictdataset-fulldata.png)

另外，您還可以選擇 **Single prediction** 來進行單次預測。這個適合模擬場景的應用（*what-if* scenarios），以及測試不同的欄位會如何影響模型的預測結果，例如：工具磨損程度如何影響故障類型？假如轉速改變會否帶來影響？

![](/static/lab6/canvas-singleprediction.png)

<br>

----

**恭喜！** 您現在已經完成了實驗6。作為下一步，您可以：

1. 再次運行這個實驗，但建立 Standard Model以查看其模型的表現；
2. 選擇運行另一個實驗