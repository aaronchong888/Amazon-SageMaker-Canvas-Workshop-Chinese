# 實驗 1 - 客戶流失預測（市場營銷）

> **Warning**
> 開始實驗前請確保您已執行並完成 **先決條件** 中的步驟。

<br>

## 議程 Agenda

1. [概述 Overview](#概述-overview)
1. [將數據導入 Canvas](#將數據導入-canvas)
1. [建構和訓練 ML 模型](#建構和訓練-ml-模型)
1. [使用模型生成預測 Predictions](#使用模型生成預測-predictions)

<br>

## 概述 Overview

在本實驗中，您將擔任某移動電話運營商營銷部門的營銷分析師的角色。作為業務分析師，您的任務是要識別可能存在流失風險的客戶。您可以獲取客戶的服務使用情況和其他行為的數據，並要使用這些數據來幫助解釋客戶離開的原因。如果我們能夠識別出導致客戶流失的因素，那麼我們就可以採取糾正措施來改變預計的行為，例如可以開展針對特定客戶的保留活動。

您將使用一個來自電信運營商的合成數據集。此示例數據集包含大約 5,000 行，21 個特徵列，而數據架構如下：

| 欄位名稱      | 描述 |
| ----------- | ----------- |
| **State**      | 客戶居住的美國州份，以兩個英文字母的縮寫表示；例如：OH（俄亥俄州）或 NJ（新澤西州） |
| **Account Length**  | 帳戶活躍的天數 |
| **Area Code** | 客戶電話號碼的三位數區號 |
| **Phone** | 區號以外的七位數電話號碼 |
| **Int’l Plan** | 客戶是否有國際通話計劃（yes/no） |
| **VMail Plan** | 客戶是否有語音信箱功能（yes/no） |
| **VMail Message** | 每月平均語音信息數量 |
| **Day Mins** | 日間使用的通話總分鐘數 |
| **Day Calls** | 日間撥打的電話總數 |
| **Day Charge** | 日間通話的計費費用 |
| **Eve Mins, Eve Calls, Eve Charge** | 黃昏通話的計費費用 |
| **Night Mins, Night Calls, Night Charge** | 夜間通話的計費費用 |
| **Intl Mins, Intl Calls, Intl Charge** | 國際電話的計費費用 |
| **CustServ Calls** | 撥打至客戶服務的電話數量 |
| **Churn?** | 客戶是否離開服務（true/false） |

最後的一個欄位 **Churn?** 是我們希望 ML 模型能作出預測的目標欄位。

<br>

## 將數據導入 Canvas

第一步是下載我們將使用的數據集。您可以到這裡下載：:link[檔案]{href="https://sagemaker-sample-files.s3.amazonaws.com/datasets/tabular/synthetic/churn.csv" action=download}.

轉到 Sagemaker Canvas。在左側選單上，您可以點擊第二個圖標，進入數據集部分，然後點擊 **Import** 按鈕。

![](/static/shared/import-data.png)

現在，點擊 **Upload** 並上傳我們已下載的 `churn.csv` 檔案。

![](/static/lab1/import-from-local.png)

在頁面底部選擇 **Preview**。

![](/static/lab1/import-preview.png)

現在，您可預覽要導入的數據集的 100 筆資料。完成資料檢查，確定正確後，您可點擊 **Import Data**。

![](/static/lab1/final-import.png)

導入過程大約需時 10 秒（因數據集大小而異）。完成後，我們可以看到數據集處於 **Ready**（就緒）狀態。

![](/static/lab1/finaldataset.png)

在確認導入的數據集準備好後，接下來我們將會創建機器學習模型。

<br>

## 建構和訓練 ML 模型

現在，讓我們通過點擊左邊選單上的第二個按鈕回到 **Models** 部分。

![](/static/shared/canvas-models.png)

點擊 **\+ New model**，並為您的模型輸入名稱，例如 `CustomerChurn`，然後選擇 **Create**。

![](/static/lab1/new-model.png)

在模型視圖中，您將看到四個選項卡，它們對應於創建模型並使用它來生成預測的四個步驟：**Select**，**Build**，**Analyze**，**Predict**。在第一個選項卡 **Select** 中，選擇我們之前已經上載的 `churn.csv` 數據集。 該數據集包括 21 欄位和 5k 筆數據。點擊底部的按鈕 **Select dataset**。

![](/static/lab1/model-dataset.png)

Canvas 將自動移動到 **Build** 階段。在此選項卡中，選擇目標欄位，在我們的情況下是 `churn?`。您的營銷團隊表示，此欄位代表了客戶是否離開了服務（true/false），這便是我們想要訓練模型來進行預測的目標。

Canvas 將自動偵測這是 **2 category prediction** 問題（也稱為二元分類）。如果系統挑選的模型類型不正確，您也可以使用屏幕中心的鏈接 **Change type** 加以改變。

![](/static/lab1/model-build.png)

我們首先可以驗證一些假設，例如我們想快速了解 目標欄位 是否可以使用 其他欄位 來預測，並初步了解模型的 預計準確度 和 欄位影響（不同欄位在預測目標時的重要性）。

點擊 **Preview model** 按鈕。

![](/static/lab1/model-preview.png)

預覽模型 將會使用數據集中的一小部分，並且在建模時僅使用一次。對於我們的示例，構建 預覽模型 大約需要 2 分鐘。

![](/static/lab1/model-preview-1.png)

完成後，當您向下滾動時，會注意到 `Phone` 和 `State` 這兩個欄位對我們的預測結果只有很少影響。我們在剔除 文本類別 的欄位時要多加留意，因為它可能包含有助於我們預測結果的重要離散分類特徵。而在這次實驗中，電話號碼 其實相當於一個帳號，它在預測客戶流失的可能性方面並沒有價值；而 客戶的居住州份 對我們的模型並沒有產生太大影響。

![](/static/lab1/feature-1.png)

因此，我們將會剔除 `Phone` 和 `State` 這兩個欄位。

![](/static/lab1/feature-2.png)

讓我們再一次運行預覽，選擇 **Update**。

![](/static/lab1/feature-3.png)

如下圖所示，這次的模型準確率提高了 0.1%。

![](/static/lab1/feature-4.png)

我們的預覽模型估計準確率為 95.9%，而影響最大的欄位為 **Night Mins, Night Calls, Night Charge**。這讓我們能夠深入了解哪些欄位對我們模型的性能影響最大。 我們在進行特徵選擇時需要格外小心，因為如果單個特徵對模型的結果影響極大，則它是 目標洩漏 的重要指標，並且該特徵將示能在預測時使用。在這次的例子中，各個欄位都顯示出相似的影響，因此我們可以繼續構建模型。

對於此次實驗，我們計劃使用所有可以使用的欄位。您可以稍後再回到此步驟，嘗試選取不同欄位的組合，以檢視對模型訓練的影響。

探索了本節後，是時候訓練模型了！ Canvas 提供了兩個構建選項：

   * **Standard build** – 通過 AutoML 的優化流程來構建最佳模型，以速度換取最大精確度。
   * **Quick build** – 與標準構建相比，構建模型的時間較短，以潛在準確性來換取速度。

![](/static/shared/canvas-quick-vs-standard.png)

在建立完整的模型前，最好我們對於訓練模型的表現，有一個基本想法。一個 **Quick Model** 訓練模型和超參數組合的配對較少，優先考慮速度勝過準確性，尤其當我們希望驗證 訓練模型 能夠產生價值。請注意，**Quick Build** 不適合大於 50k 筆數的模型。現在，讓我們繼續點擊 **Quick build**。

![](/static/lab1/model-building.png)

現在，我們大概需要等待 2 - 15 分鐘的時間讓 **Quick build** 完成模型的訓練。由於數據集很小，因此可能不需要等到2分鐘。

完成後，Canvas 將自動移動到 **Analyze** 選項卡，向我們展示快速訓練的結果：模型預測正確的客戶流失率為 95.9%。 這看起來不錯，但作為分析師，我們希望更深入地研究，看看我們是否可以信任這個模型並來根據它做出決策。在 **Scoring** 選項卡中，您可以看到表示正確與錯誤預測值的分佈圖，這使我們能夠更深入地了解我們的模型。

讓我們專注於第一個選項卡 **Overview**。這是向我們展示欄位影響 *Column impact*，或這些特徵，在預測目標時的重要性。在我們的案例中，**Night Calls** 這個欄位在預測客戶是否會流失方面具有最顯著的影響。

![](/static/lab1/model-status-1.png)

> **Warning**
> 不用擔心以下截屏中顯示的數字是否與您的數字不同。機器學習在模型訓練過程中會引入一些隨機性，這將導致訓練產生出不同的結果。

Canvas 會自動將數據集分為 訓練數據集 和 測試數據集。 訓練數據集是 Canvas 用於構建模型的數據，而 測試數據集 則是用於測試模型在處理新數據時是否表現良好。 以下屏幕截圖中的 Sankey圖 顯示了模型在 測試數據集 上的表現。假如您想了解更多詳細資料，您可以查看 ["Evaluating Your Model's Performance in Amazon SageMaker Canvas" section](https://docs.aws.amazon.com/sagemaker/latest/dg/canvas-evaluate-model.html) 和 [SHAP Baselines for Explainability](https://docs.aws.amazon.com/sagemaker/latest/dg/clarify-feature-attribute-shap-baselines.html)。

讓我們轉到 **Overview** 選項卡，查看每一個欄位的影響。這些信息可以幫助營銷團隊進行分析，從而採取行動來減少客戶流失。例如，我們可以看到 `CustServ Calls` 數量的高或低都會增加客戶流失的可能性，而營銷團隊便可以根據這些發現以採取措施防止客戶流失，例如可以在網站上創建詳細的 FAQ 常見問題解答 以減少客戶服務電話，並針對常見問題為客戶提供教育活動，以保持他們的參與度。

![](/static/lab1/model-overview-1.png)

業務分析師也可以透過分析 混淆矩陣（Confusion Matrix）以獲取更詳細的見解，例如我們希望更好地了解模型做出錯誤預測的可能性。我們可以點擊 **Advanced metrics** 部分更深入了解模型的性能及更詳細的信息。

我們會看到一個 混淆矩陣，它以視覺化格式來顯示模型的性能，在這個例子中，正類（positive class）為 True（表示 **客戶會流失** 的預測結果）：

   真陽性 True Positive (TP) – 正確預測為 `真` 的 `真結果` 數量

   真陰性 True Negative (TN) – 正確預測為 `假` 的 `假結果` 數量

   假陽性 False Positive (FP) – 被錯誤預測為 `真` 的 `假結果` 數量

   假陰性 False Negative (FN) – 被錯誤預測為 `假` 的 `真結果` 數量

我們不僅可以使用這個矩陣圖來確定我們的模型有多準確，還可以確定模型何時出錯、錯誤的頻率 以及 錯誤的程度。

由於這些指標看起來不錯，只有非常低的 假陽性（如果模型預測客戶會流失，而他們實際上不會）和 假陰性（如果模型預測客戶不會流失，而他們實際上會），所以我們可以相信這個模型的預測結果。

![](/static/lab1/advanced-metrics-1.png)

假如我們的模型看起來非常準確，我們可以直接在 **Predict** 頁面上運行實時預測，可以是批量預測，也可以是單個預測。

現在，您有兩個選擇：

1. 您可以使用此模型運行一些預測，請選擇頁面底部的 **Predict** 按鈕；
1. 您可以使用 **Standard Build** 流程訓練此模型的新版本。這將花費更長的時間（大約 4 - 6 小時），但會產生更準確的結果。

在這個實驗中，我們將選擇選項 1。假如您有更多的空餘時間，也可以選擇運行選項 2。

> **Warning**
> 請注意，如果您要從 SageMaker Canvas 共享模型到 SageMaker Studio，您必須使用 **Standard Build** 來訓練模型。**Predictions** 並不需要**Standard Build**，但是它的性能和準確性會比經過完全訓練的模型低。

<br>

## 使用模型生成預測 Predictions

現在完成模型的訓練，我們可以做一些預測。在 **Analyze** 頁面底部選擇 **Predict**，或選擇 **Predict** 選項卡。

現在，點擊 **Select dataset**。

![](/static/lab1/batch-predict.png)

選擇 `churn.csv`。接下來，選擇頁面底部的 **Generate predictions** 來生成預測。

![](/static/lab1/batch-predict-ds.png)

Canvas 將使用這個數據集來生成我們的預測。雖然在實際環境下我們在訓練和測試時並不會使用相同的數據集，但為了簡單起見，我們在這次實驗中將使用相同的數據集。

幾秒鐘後，預測完成。您可以點擊 **View** 以查看結果。

![](/static/lab1/batch-predict-results.png)

您也可以點擊 download 按鈕下載 CSV 文件。 SageMaker Canvas 將會為每行數據返回一個 預測值 以及 預測正確的概率。

![](/static/lab1/batch-predict-results-1.png)

另外，您還可以選擇 **Single prediction** 來進行單次預測。這個適合模擬場景的應用（*what-if* scenarios），以及測試不同的欄位會如何影響模型的預測結果，例如：如果增加夜間費用會否帶來影響？

![](/static/lab1/singleprediction.png)

----

**恭喜！** 您現在已經完成了實驗1。作為下一步，您可以：

1. 再次運行這個實驗，但建立 Standard Model以查看其模型的表現；
2. 選擇運行另一個實驗