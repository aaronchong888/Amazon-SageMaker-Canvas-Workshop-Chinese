---
title : "實驗 5 - 糖尿病患者再入院預測（醫療保健及生命科學）"
weight : 20
---

::alert[開始實驗前請確保您已執行並完成 **先決條件** 中的步驟。]{type=warning}

## 議程 Agenda

1. [概述 Overview](#overview)
1. [糖尿病患者再入院預測 Diabetes Readmission](#diabetes-readmission)
1. [在 Canvas 中創建模型並導入數據集](#canvas)
  1. [將數據集上傳 S3 存儲桶](#s3)
  1. [在 SageMaker Canvas 中創建模型](#sagemaker-canvas)
  1. [將數據集導入模型 Import Dataset](#import-dataset)
1. [建構和訓練 ML 模型](#ml)
1. [使用模型生成預測 Predictions](#predictions)

## 概述 Overview

在本實驗中，您將學習如何使用 Amazon Sagemaker Canvas 分析代表患者和其醫療結果的歷史數據集，並以無需編寫代碼的方式來構建機器學習（ML）模型來預測病患再入院率。該模型必須預測高危糖尿病患者是否有可能在 30 天內或在 30 天后再次入院。由於此案例需要預測多個結果，因此這為 **多類分類** 的 ML 問題。

## 糖尿病患者再入院預測 Diabetes Readmission

病患再入院佔醫療總開支的一大重要部分，也是衡量護理質量的指標。而糖尿病與其他慢性疾病相似，也有更高的再入院風險。

我們在本實驗中所使用的樣本數據集為：["Diabetes 130-US hospitals for years 1999-2008 Data Set"](https://archive.ics.uci.edu/ml/datasets/diabetes+130-us+hospitals+for+years+1999-2008) (Beata Strack, Jonathan P. DeShazo, Chris Gennings, Juan L. Olmo, Sebastian Ventura, Krzysztof J. Cios, and John N. Clore, “Impact of HbA1c Measurement on Hospital Readmission Rates: Analysis of 70,000 Clinical Database Patient Records,” BioMed Research International, vol. 2014, Article ID 781670, 11 pages, 2014.)。它包含超過 15 種患者和其醫療結果的特徵欄位，而數據集有大約 69,500 行，數據架構如下：

| 欄位名稱 | 資料型別 | 描述 |
| ----------- | ----------- | ----------- |
| race | STRING | Caucasian, Asian, African American 或 Hispanic
| time_in_hospital | INT | 入院和出院之間的天數，即是住院日數
| number_outpatient | INT | 就診前一年內患者的門診次數
| number_inpatient | INT | 就診前一年內患者的住院次數
| number_emergency | INT | 就診前一年內患者的急診次數
| number_diagnoses | INT | 已輸入於系統內的診斷次數
| num_procedures | INT | 就診期間進行的醫療程序次數（實驗室測試除外）
| num_medications | INT | 就診期間所發出的非專利藥物數量
| num_lab_procedures | INT | 就診期間進行的實驗室測試次數
| max_glu_serum | STRING | 表示測試結果範圍或是否未進行測試。可能的數值：">200", ">300", "normal" 或 "none" （未進行測試）
| gender | STRING | 性別："Male", "Female" 或 "Unknown/Invalid"
| diabetes_med | INT | 表示是否處方了任何糖尿病藥物
| change | STRING | 表示糖尿病藥物劑量或藥物名稱是否曾經改變。可能的數值："change" 或 "no change"
| age | INT | 就診時患者的年齡
| a1c_result | STRING | 表示採樣的結果範圍或是否未進行測試。可能的數值：">8", ">7", "normal" 或 "none"
| readmitted | STRING | 住院再入院的日數。可能的數值："<30"（如果患者重新入院少於 30 天）, ">30" （如果患者在就診 30 天后重新入院）, "no"（無再入院記錄）

## 在 Canvas 中創建模型並導入數據集

### 將數據集上傳 S3 存儲桶

第一步是下載我們將使用的數據集。您可以到這裡下載：:link[檔案]{href="/static/datasets/diabetic-readmission.csv" action=download}.

轉到 AWS 管理控制台，在控制台頂部的搜索框中尋找 **S3**，然後去到 **S3** 服務控制台。

在 S3 控制台中，點擊 **sagemaker-studio-\*** 存儲桶。

![](/static/shared/studio-bucket.png)

::alert[ **sagemaker-studio-\*** 在當初建立 SageMaker Studio domain 的時候，就已經自動建立。如果你參與 **Event Engine** 活動, 則講師會預先準備存儲桶。]

點擊 **Upload**。

![](/static/shared/s3_upload.png)

在上傳頁面上，拖放剛才下載的 `diabetes-readmission.csv` 檔案，然後點擊頁面底部的 **Upload**。上傳完成後，您可以點擊右上角 **Close** 按鈕。現在，您應該看到上傳到存儲桶中的文件。

![](/static/lab5/s3-file-upload.png)

### 在 SageMaker Canvas 中創建模型

返回到 Sagemaker Canvas。在左側選單上，您可以點擊第二個圖標轉到 **Models** 部分。

![](/static/shared/canvas-models.png)

點擊 **\+ New model**，並為您的模型輸入名稱，例如 `Diabetes Readmission Prediction`，然後選擇 **Create**。

![](/static/lab5/create-diabetes-readmission-prediction-model.png)

如果這是您第一次建立 Canvas 模型，那麼您將看到一個彈出式歡迎，其中有關於如何通過 4 個簡單步驟建構您第一個模型的信息。您可以閱讀此信息，然後回到本實驗指南。

![](/static/shared/canvas-first-model-popup.png)

### 將數據集導入模型 Import Dataset

在模型視圖中，您將看到四個選項卡，它們對應於創建模型並使用它來生成預測的四個步驟：**Select**，**Build**，**Analyze**，**Predict**。在第一個選項卡 **Select** 中，點擊 **+ Import** 按鈕。

![](/static/lab5/select-import-dataset.png)

現在，選擇之前上傳到 **sagemaker-studio-\*** 存儲桶的數據集。

![](/static/shared/import-from-s3-studio.png)

現在，選擇 `diabetes-readmission.csv` 檔案。您可以點擊 **View** 圖標來預覽要導入的數據集的 100 筆資料。

![](/static/lab5/preview-first-100-rows.png)

在預覽頁面上選擇 **Select Dataset** 將數據導入。

![](/static/lab5/select-dataset-from-preview.png)

Canvas 將自動移動到 **Build** 階段。在此選項卡中，選擇目標欄位，在我們的情況下是`readmitted`。該目標表示患者是否在 30 天內或 30 天后再次入院或無再入院記錄（數值："<30", ">30" 或 "no"）。

![](/static/lab5/target-selection.png)

::alert[由於這次實驗演示的數據集並不平衡，因此會導致準確性降低。]{type=warning}

Canvas 將自動偵測這是 **3+ category prediction** 問題（也稱為多類分類）。如果系統挑選的模型類型不正確，您也可以使用屏幕中心的鏈接 **Change type** 加以改變。

![](/static/lab5/model-type.png)

在屏幕的下半部，您可以查看數據集的一些統計屬性，包括缺失和不匹配的值，獨特的值，平均值和中位數。

![](/static/lab5/features-preview.png)

如果我們不想使用特定欄位，可以使用左邊的複選框進行檢查，取消特定欄位的勾選。在這次的實驗中，我們將會剔除 `a1c_result`, `max_glu_serum`, `gender`, `num_procedures`, `number_outpatient` 欄位，因為它們的欄位影響比較小。最後結果應如下圖所示：

![](/static/lab5/model-recipe.png)

您還可以轉到 `Grid View` 網格視圖來更深入地了解每一列，它會為您提供每列值的圖形分佈和示例數據：

![](/static/lab5/columns-grid-view-detail.png)

## 建構和訓練 ML 模型

在構建模型前，您可以通過 `Preview model` （預覽模型）來分析數據集並為我們提供 `Estimated Accuracy`（估計準確度）和欄位影響。

![](/static/lab5/preview-model.png)

在分析數據集並根據欄位影響修改相關欄位後，您可以選擇 `Standard Build`（標準構建，通常需要 2-4 小時的訓練時間並能提供更好的準確性），或 `Quick Build`（快速構建，通常只需 2-5 分鐘的訓練時間但會降低準確性）來構建模型。請注意，如果您要從 SageMaker Canvas 共享模型到 SageMaker Studio，您必須使用 **Standard Build** 來訓練模型。對於這次的多類分類問題，我們將會運行標準構建。

SageMaker Canvas 在構建 標準模型 時會自動為您進行一定程度的預處理和數據平衡，而 SageMaker Canvas 使用的其中一些算法將會透過它們的超參數來處理不平衡的數據。

現在，我們大約需要等待 2 個小時，實際時間可能因人而異。完成模型的訓練後，Canvas 將自動移至 **Analyze** 選項卡，向我們展示訓練的結果。這裡的分析估計我們的模型大概有 61% 的預測準確率。

![](/static/lab5/standard-build-analyze.png)

讓我們專注於第一個選項卡 **Overview**。這是向我們展示欄位影響 *Column impact*，或這些特徵，在預測目標時的重要性。在我們的案例中，`number_diagnoses` 這個欄位對預測病患再入院的影響最大，其次為 `number_inpatient`。

![](/static/lab5/analyze.png)

::alert[不用擔心截屏中顯示的數字是否與您的數字不同。機器學習在模型訓練過程中會引入一些隨機性，這將導致訓練產生出不同的結果。]{type=warning}

讓我們轉到 **Scoring** 選項卡，您可以看到表示模型的預測值相對於實際值的分佈圖，而根據預測結果，大部分病患也無須再入院。假如您想了解有關 Canvas 如何使用 SHAP 基線為機器學習帶來可解釋性的詳細資料，您可以查看 ["Evaluating Your Model's Performance in Amazon SageMaker Canvas" section](https://docs.aws.amazon.com/sagemaker/latest/dg/canvas-evaluate-model.html) 和 [SHAP Baselines for Explainability](https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-feature-attribute-shap-baselines.html)。

![](/static/lab5/scoring.png)

我們可以點擊 **Advanced metrics** 部分更深入了解模型的性能及更詳細的信息。

您會看到頁面顯示出一個矩陣，而在機器學習中，這稱為混淆矩陣（Confusion Matrix）。

如要了解 **無須再入院病患** 的預測結果，請從右側面板中選擇 `no`。

在機器學習中，模型的準確性定義為正確預測的數量除以預測的總數。藍色框表示模型針對具有已知結果的測試數據子集做出的正確預測。在這種情況下，在總共 `13824` 的預測中有 `8426` 個正確的預測，因此準確率為 `61%`。

當然，您會更感興趣的是模型對 **患者再入院** 的預測程度。該模型正確預測 `7890` 客戶不會重新入院（真陽性 - TP）。然而，它錯誤地預測了 `4801` 客戶不會再入院，而實際上他們是會的（假陰性 - FN）。在機器學習中，用來衡量這一點的比率是 TP / (TP + FN)，這稱為召回率。**Advanced metrics** 頁面計算並顯示出此模型的召回率為 `36%`。

![](/static/lab5/advanced-metrics-0.png)

## 使用模型生成預測 Predictions

現在完成模型的訓練，我們可以做一些預測。在 **Analyze** 頁面底部選擇 **Predict**，或選擇 **Predict** 選項卡。

![](/static/lab5/prediction.png)

現在，點擊 **Select dataset**，然後選擇 `diabetes-readmission.csv`。接下來，選擇頁面底部的 **Generate predictions** 來生成預測。Canvas 將使用這個數據集來生成我們的預測。雖然在實際環境下我們在訓練和測試時並不會使用相同的數據集，但為了簡單起見，我們在這次實驗中將使用相同的數據集。

幾秒鐘後，預測完成。您可以點擊長的很像 "眼睛" 的圖標做預覽，或點擊 download 按鈕下載 CSV 文件。 SageMaker Canvas 將會為每行數據返回一個 預測值 以及 預測正確的概率。

![](/static/lab5/prediction-preview.png)

![](/static/lab5/prediction-results.png)

另外，您還可以選擇 **Single prediction** 來進行單次預測。這個適合模擬場景的應用（*what-if* scenarios），以及測試不同的欄位會如何影響模型的預測結果。

![](/static/lab5/single-prediction-result.png)

----

**恭喜！** 您現在已經完成了實驗5。作為下一步，您可以：

1. 更新數據集並進行預處理和數據平衡，然後再次運行這個實驗來檢查對模型性能的影響
2. 選擇運行另一個實驗