---
title : "自定進度"
weight : 11
---

::alert[如果您已經執行了上頁 "AWS 講師指導" 中的步驟，則 **無須** 執行這頁的步驟。]{type=warning}

::alert[您需要使用自己的 AWS 帳戶來執行這頁的步驟，當中可能會產生一些費用。]{type=warning}


- [啟用 Amazon SageMaker Canvas](#amazon-sagemaker-canvas)
- [新增 IAM 信任政策](#iam)
- [新增 S3 存儲桶 CORS 規則](#s3-cors)


## 啟用 Amazon SageMaker Canvas

Amazon SageMaker Canvas 透過使用視覺化點選式介面為商業分析師提供產生準確機器學習 (ML) 預測的能力，而無需使用任何程式碼或機器學習經驗。直觀的用戶界面讓您可以瀏覽和存取在雲端或本地的不同數據源，只需單擊按鈕即可組合數據，訓練準確的模型，並在新數據可用時生成新的預測。

SageMaker Canvas 運用與 Amazon SageMaker 相同的技術來自動整理和合併數據，然後在後台創建數百個不同的模型，從中選擇表現最佳的模型，並生成新的單個或批量預測結果。它能支持多種機器學習類型，例如二元分類、多類分類、數值回歸和時間序列預測，讓您在無需編寫任何代碼的情況下即可解決關鍵業務問題，例如欺詐檢測、客戶流失和庫存優化等的應用實例。

請按照以下步驟以 **Quick Setup** 方式啟用 Amazon SageMaker Canvas：

1.  打開 AWS 控制台並切換到您想要使用的 AWS 區域。

::alert[圖中示例為愛爾蘭區域，但您也可以選擇其他提供 SageMaker Canvas 的區域。[區域列表](https://docs.aws.amazon.com/sagemaker/latest/dg/canvas.html)]

![](/static/prerequisites/image22.png)

2.  搜尋 **Amazon SageMaker**：

![](/static/prerequisites/image23.png)

3.  選擇 **Amazon SageMaker Studio**（左側菜單中的第一個選項）：

![](/static/prerequisites/image40.png)

4.  選擇 **Quick start**：

5.  設定 **Name**（例如：**sagemakeruser**）：

![](/static/prerequisites/image52.png)

6.  在執行角色（Execution Role）下方選擇 **Create a new role**：

![](/static/prerequisites/image53.png)

7. 保持默認設置並選擇 **Create Role**：

![](/static/prerequisites/image54.png)

8. 您將看到角色已成功創建：

![](/static/prerequisites/image55.png)

9. 選擇 **Submit**：

![](/static/prerequisites/image27.png)

10.	SageMaker Studio 環境將保持 **Pending** 狀態幾分鐘：

![](/static/prerequisites/image56.png)

11.	幾分鐘後，SageMaker Studio Domain 便會準備就緒。在 **Launch app** 下方選擇 **Canvas**：
 
![](/static/prerequisites/image57.png)

12.	首次訪問 SageMaker Canvas 時，可能需要 1-2 分鐘才能載入頁面：

![](/static/prerequisites/image30.png)

13.	最後，您將被重定向到以下的頁面：

![](/static/prerequisites/image31.png)

## 新增 IAM 信任政策

如果您計劃運行實驗3 - 銷售預測（時間序列預測）， 請按照[Canvas 文檔](https://docs.aws.amazon.com/sagemaker/latest/dg/canvas-set-up-forecast.html)的步驟來進行以下設置：

1. 轉到先前的 SageMaker Studio domain 頁面，並複制 Studio Domain 詳細信息中的執行角色（Execution Role）名稱：

![](/static/prerequisites/find-execution-role.png)

2. 前往 [IAM Roles Management Console](https://console.aws.amazon.com/iamv2/home?#/roles), 貼上並搜尋上一步的執行角色（Execution Role）名稱：

![](/static/prerequisites/find-execution-role.png)

3. 點擊角色名稱，選擇 **Add permissions**，然後選擇 **Attach policies**：

![](/static/prerequisites/attach-policies.png)

4. 搜尋並選擇 `AmazonForecastFullAccess` 和 `AmazonRedshiftFullAccess`，然後選擇 **Attach policies**。完成後，您的權限政策應如下圖所示：

![](/static/prerequisites/permission-set.png)

5. 選擇 **Trust relationships**，然後點擊 **Edit trust policy** 按鈕：

![](/static/prerequisites/edit-trust-policy.png)

6. 複製並貼上以下的 JSON 政策文件：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": [
            "sagemaker.amazonaws.com",
            "forecast.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

7. 最後，選擇 **Update policy** 便完成。 

## 新增 S3 存儲桶 CORS 規則

SageMaker Canvas 提供了通過 Canvas UI 將本地存儲的數據上傳到 S3 的功能，但是您必須要把相應的 CORS 規則加入到要上傳數據的 S3 存儲桶（如要了解更多，[Canvas 文檔](https://docs.aws.amazon.com/sagemaker/latest/dg/canvas-set-up-local-upload.html) 提供了詳細解釋）。 請按照以下步驟進行設置：

1. 首先，前往 [S3 Console](https://console.aws.amazon.com/s3/) 並確認您的 SageMaker 默認 S3 存儲桶或您想使用的任何其他存儲桶。在這個例子中，我們將選擇名為 `sagemaker-studio-****` 的存儲桶。您亦可以使用名為 `sagemaker-[AWS-REGION]-[ACCOUNT-ID]` 的存儲桶。

![](/static/prerequisites/sagemaker-studio-bucket.png)

2. 選擇您想使用的 S3 存儲桶，點擊 **Permissions** 並移至 **Cross-origins resource sharing (CORS)**，選擇 **Edit**：

![](/static/prerequisites/edit-cors.png)

3. 複製並貼上以下的 CORS 規則：

```json
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "POST"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": []
    }
]
```

4. 最後，選擇 **Save changes** 便完成。 

-----

恭喜！！您已成功完成 SageMaker Canvas 的設置。 現在您可以從本頁左側菜單中選擇您喜歡的實驗。
