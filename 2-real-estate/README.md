# 實驗 2 - 房價預測（房地產）

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

在本實驗中，您將學習如何使用 Amazon SageMaker Canvas 匯入與分析一個住房價格的數據集，然後建構一個機器學習模型來進行預測，而在整個過程中無需撰寫任何代碼。由於住房價格可以是任何實際值，這類機器學習問題稱為 回歸。

我們在本實驗中所使用的數據集為：["California Housing Dataset"](https://www.dcc.fc.up.pt/~ltorgo/Regression/cal_housing.html)(Pace, R. Kelley and Ronald Barry, Sparse Spatial Autoregressions, Statistics and Probability Letters, 33 (1997) 291-297.)。而數據架構如下：

| 欄位名稱 | 資料型別 | 描述 |
| ----------- | ----------- | ----------- | 
| latitude | DECIMAL | 衡量房屋靠近西部的範圍；越靠近西部價值越高 | 
| longitude | DECIMAL| 衡量房屋靠近北部的範圍；越靠近北部價值越高 | 
| housing_median_age | INT | 街區內房屋的中位年齡；數字越小建築物越新 | 
| total_rooms | INT | 街區內的房間總數 |
| total_bedrooms | INT | 街區內的臥室總數 | 
| population | DECIMAL | 居住在一個街區中的人數 | 
| households | INT | 家庭總數，一群居住在家庭單位內的人 |
| median_income | DECIMAL | 房屋街區內家庭的中位收入（以 萬美元 為單位表示） |
| median_house_value (target) | DECIMAL | 一個街區內的家庭價值中位數（以 美元 為單位表示） |
| ocean_proximity | STRING | 房屋位置相對於海洋的距離 |

<br>

## 將數據集上傳 S3 存儲桶

第一步是下載我們將使用的數據集。您可以到這裡下載：[檔案](/static/datasets/housing.csv)

轉到 AWS 管理控制台，在控制台頂部的搜索框中尋找 **S3**，然後去到 **S3** 服務控制台。

![](/static/shared/search_s3.png)

在 S3 控制台中，點擊 **sagemaker-studio-\*** 存儲桶。

![](/static/shared/studio-bucket.png)

> **Warning**
> **sagemaker-studio-\*** 在當初建立 SageMaker Studio domain 的時候，就已經自動建立。如果你參與 **Event Engine** 活動, 則講師會預先準備存儲桶。

點擊 **Upload**。

![](/static/shared/s3_upload.png)

在上傳頁面上，拖放剛才下載的 `housing.csv` 檔案，然後點擊頁面底部的 **Upload**。上傳完成後，您可以點擊右上角 **Close** 按鈕。現在，您應該看到上傳到存儲桶中的文件。

![](/static/lab2/s3-uploaded-housing.png)

<br>

## 將數據導入 Canvas

返回到 Sagemaker Canvas。在左側選單上，您可以點擊第二個圖標，進入數據集部分，然後點擊 **Import** 按鈕。

![](/static/shared/import-data.png)

現在，選擇之前上傳到 **sagemaker-studio-\*** 存儲桶的數據集。

![](/static/shared/import-from-s3-studio.png)

您可以通過在其左側複選框來選擇先前上傳的 `housing.csv` 檔案。頁面底部將彈出兩個新的按鈕：**Preview all** 和 **Import Data**。讓我們選擇第一個。

![](/static/lab2/canvas-select-preview.png)

現在，您可預覽要導入的數據集的 100 筆資料。完成資料檢查，確定正確後，您可點擊 **Import Data**。

![](/static/lab2/canvas-preview.png)

<br>

## 建構和訓練 ML 模型

現在，讓我們通過點擊左邊選單上的第二個按鈕回到 **Models** 部分。

![](/static/shared/canvas-models.png)

點擊 **\+ New model**，並為您的模型輸入名稱，例如 `Housing Regression`，然後選擇 **Create**。

![](/static/lab2/new-housing-model.png)

如果這是您第一次建立 Canvas 模型，那麼您將看到一個彈出式歡迎，其中有關於如何通過 4 個簡單步驟建構您第一個模型的信息。您可以閱讀此信息，然後回到本實驗指南。

![](/static/shared/canvas-first-model-popup.png)

在模型視圖中，您將看到四個選項卡，它們對應於創建模型並使用它來生成預測的四個步驟：**Select**，**Build**，**Analyze**，**Predict**。在第一個選項卡 **Select** 中，選擇我們之前已經上載的 `housing.csv` 數據集。 該數據集包括 10 欄位和超過 20k 筆數據。點擊底部的按鈕 **Select dataset**。

![](/static/lab2/dataset-select.png)

Canvas 將自動移動到 **Build** 階段。在此選項卡中，選擇目標欄位，在我們的情況下是 `median_house_value`。Canvas 將自動偵測這是 **numeric prediction** 問題（也稱為回歸）。如果系統挑選的模型類型不正確，您也可以使用屏幕中心的鏈接 **Change type** 加以改變。

![](/static/lab2/housing-build.png)

在屏幕的下半部，您可以查看數據集的一些統計屬性，包括缺失和不匹配的值，獨特的值，平均值和中位數。如果我們不想使用特定欄位，可以使用左邊的複選框進行檢查，取消特定欄位的勾選。對於此次實驗，我們計劃使用所有可以使用的欄位。您可以稍後再回到此步驟，嘗試選取不同欄位的組合，以檢視對模型訓練的影響。

探索了本節後，是時候訓練模型了！在建立完整的模型前，最好我們對於訓練模型的表現，有一個基本想法。一個 **Quick Model** 訓練模型和超參數組合的配對較少，優先考慮速度勝過準確性，尤其當我們希望驗證 訓練模型 能夠產生價值。請注意，**Quick Build** 不適合大於 50k 筆數的模型。現在，讓我們繼續點擊 **Quick build**。

![](/static/shared/canvas-quick-vs-standard.png)

現在，我們大概需要等待 2 - 15 分鐘的時間讓 **Quick build** 完成模型的訓練。由於數據集很小，因此可能不需要等到2分鐘。完成後，Canvas 將自動移動到 **Analyze** 選項卡，向我們展示快速訓練的結果：

![](/static/lab2/housing-analyze.png)

> **Warning**
> 不用擔心以下截屏中顯示的數字是否與您的數字不同。機器學習在模型訓練過程中會引入一些隨機性，這將導致訓練產生出不同的結果。

頁面上顯示了回歸問題的指標，稱為均方根誤差（RMSE）。這代表預測的分佈情況，數值越低，代表我們對我們的目標的預測越準確。而這意味著這個模型的預測具有 +/-48k 的均方根誤差。我們可以在稍後檢視 預測值 跟 實際值 的差距有多大。

讓我們專注於第一個選項卡 **Overview**。這是向我們展示欄位影響 *Column impact*，或這些特徵，在預測目標時的重要性。在我們的案例中，即使是影響最低的欄位，仍然具有一定的影響力，所以我們應該保留所有欄位。我們稍後還是可以嘗試拿掉其中一些欄位，觀察這將如何影響模型的表現。

假如您想了解有關 Canvas 如何使用 SHAP 基線為機器學習帶來可解釋性的詳細資料，您可以查看 ["Evaluating Your Model's Performance in Amazon SageMaker Canvas" section](https://docs.aws.amazon.com/sagemaker/latest/dg/canvas-evaluate-model.html) 和 [SHAP Baselines for Explainability](https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-feature-attribute-shap-baselines.html)。

![](/static/lab2/housing-quick-scoring.png)

當我們移動到 **Scoring** 部分，我們可以看到一個預測值分佈的圖表：這將使我們能夠計算另一個指標，平均誤差（MAE），這表示我們的預測與實際值的平均差異。您可以通過點擊右側的 **Advanced metrics** 按鈕，進一步檢視指標並獲得更多信息。

現在，您有兩個選擇：

1. 您可以使用此模型運行一些預測，請選擇頁面底部的 **Predict** 按鈕；
1. 您可以使用 **Standard Build** 流程訓練此模型的新版本。這將花費更長的時間（大約 4 - 6 小時），但會產生更準確的結果。

在這個實驗中，我們將選擇選項 1。假如您有更多的空餘時間，也可以選擇運行選項 2。

> **Warning**
> 請注意，如果您要從 SageMaker Canvas 共享模型到 SageMaker Studio，您必須使用 **Standard Build** 來訓練模型。**Predictions** 並不需要**Standard Build**，但是它的性能和準確性會比經過完全訓練的模型低。

<br>

## 使用模型生成預測 Predictions

現在完成模型的訓練，我們可以做一些預測。在 **Analyze** 頁面底部選擇 **Predict**，或選擇 **Predict** 選項卡。

![](/static/lab2/housing-predict-setup.png)

現在，點擊 **Select dataset**，然後選擇 `housing.csv`。接下來，選擇頁面底部的 **Generate predictions** 來生成預測。Canvas 將使用這個數據集來生成我們的預測。雖然在實際環境下我們在訓練和測試時並不會使用相同的數據集，但為了簡單起見，我們在這次實驗中將使用相同的數據集。

幾秒鐘後，預測完成。您可以點擊長的很像 "眼睛" 的圖標做預覽，或點擊 download 按鈕下載 CSV 文件。 SageMaker Canvas 將會為每行數據返回一個 預測值。

![](/static/lab2/housing-batch-predictions.png)

![](/static/lab2/housing-batch-predictions-2.png)

另外，您還可以選擇 **Single prediction** 來進行單次預測。這個適合模擬場景的應用（*what-if* scenarios），以及測試不同的欄位會如何影響模型的預測結果，例如：如果房屋年齡較大，價格會如何變化？ 如果房屋靠近海洋，價格會怎樣？ 如果房屋有更多房間，價格又會怎樣？

![](/static/lab2/housing-single-inference.png)

----

**恭喜！** 您現在已經完成了實驗2。作為下一步，您可以：

1. 再次運行這個實驗，但建立 Standard Model以查看其模型的表現；
2. 選擇運行另一個實驗