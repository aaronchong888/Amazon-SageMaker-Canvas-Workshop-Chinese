---
title : "AWS 講師指導"
weight : 11
---

::alert[請按照講師及工作坊職員的指示來獲取和登錄是次活動提供的 AWS 帳戶。 請勿使用您的個人或企業 AWS 帳戶來運行實驗，否則實驗當中所需要的預建資源將無法使用。]{type=warning}

- [獲取 Event Engine AWS 帳戶](#event-engine-aws)
- [訪問 Amazon SageMaker Canvas](#amazon-sagemaker-canvas)
- [新增 IAM 信任政策](#iam)
- [新增 S3 存儲桶 CORS 規則](#s3-cors)


## 獲取 Event Engine AWS 帳戶

以瀏覽器訪問網址：https://dashboard.eventengine.run/login  您將被重定向到以下的頁面：

![](/static/prerequisites/image43.png)

輸入由講師提供的 event hash：

![](/static/prerequisites/image44.png)

選擇 **Email One-Time Password (OTP)**：

![](/static/prerequisites/image45.png)

您將被重定向到以下頁面：

![](/static/prerequisites/image46.png)

輸入您的電子郵件地址，然後選擇 **Send passcode**：

![](/static/prerequisites/image47.png)

您將被重定向到以下頁面：

![](/static/prerequisites/image48.png)

檢查您的電子郵箱，複製並貼上一次性密碼，然後選擇 **Sign in**：

![](/static/prerequisites/image49.png)

您將被重定向至 **Team Dashboard** 頁面。選擇 **AWS Console**：

![](/static/prerequisites/image50.png)

在下個頁面選擇 **Open AWS Console**：

![](/static/prerequisites/image51.png)

您將被重定向至 **AWS Console**。


## 訪問 Amazon SageMaker Canvas

Amazon SageMaker Canvas 透過使用視覺化點選式介面為商業分析師提供產生準確機器學習 (ML) 預測的能力 (無需任何程式碼或機器學習經驗)，藉此延伸機器學習的存取。

您的 AWS 帳戶和資源已由講師預建，請按照以下步驟訪問 Amazon SageMaker 環境：

1.  打開 AWS 控制台並切換到講師指定的 AWS 區域。

::alert[圖中示例為愛爾蘭區域，但您也可以選擇其他提供 SageMaker Canvas 的區域。請依照講師的指示選擇正確的區域。]{type=warning}

提供 SageMaker Canvas 的 AWS 區域列表 [連結](https://docs.aws.amazon.com/sagemaker/latest/dg/canvas.html)。

![](/static/prerequisites/image22.png)

2.  搜尋 **Amazon SageMaker**：

![](/static/prerequisites/image23.png)

3.  在 **Get Started** 底下，點擊橙色的按鈕 **SageMaker Studio**：

![](/static/prerequisites/image41.png)

4.	SageMaker Studio 環境已經準備就緒，選擇 **Open Canvas** (在預先配置的 **sagemakeruser** 用戶名右側)：

![](/static/prerequisites/image42.png)

5.	首次訪問 SageMaker Canvas 時，可能需要 1-2 分鐘才能載入頁面：

![](/static/prerequisites/image30.png)

6.	最後，您將被重定向到以下的頁面：

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
