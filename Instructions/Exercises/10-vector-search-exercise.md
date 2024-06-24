---
lab:
  title: 使用 REST API 執行向量搜尋查詢
---

# 使用 REST API 執行向量搜尋查詢

在此練習中，您將設定專案、建立索引、上傳文件，以及執行查詢。

您需要下列項目才能成功執行此練習：

- [Postman](https://www.postman.com/downloads/) 應用程式
- Azure 訂閱
- Azure AI 搜尋服務
- Postman 範例集合位於此存放庫中 - *Vector-Search-Quickstart.postman_collection v1.0 json*。

> **注意** 如有需要，您可以在[這裡](https://learn.microsoft.com/en-us/azure/search/search-get-started-rest)找到 Postman 應用程式 的詳細資訊。

## 設定您的專案

請先執行下列步驟來設定您的專案：

1. 記下 Azure AI 搜尋服務的 **URL** 和**金鑰**。

    ![服務名稱和金鑰的位置圖例。](../media/vector-search/search keys.png)

1. 下載 [Postman 範例集合](https://github.com/MicrosoftLearning/mslearn-knowledge-mining/blob/main/Labfiles/10-vector-search/Vector%20Search.postman_collection%20v1.0.json)。
1. 選取 [匯入]**** 按鈕並將集合資料夾拖放到方塊中，以開啟 Postman 並匯入集合。

    ![[匯入] 對話方塊的影像](../media/vector-search/import.png)

1. 選取 [分支]**** 按鈕以建立集合的分支，並新增唯一的名稱。
1. 以滑鼠右鍵按一下您的集合名稱，然後選取 [編輯]****。
1. 選取 [變數]**** 索引標籤，並使用 Azure AI 搜尋服務的搜尋服務和索引名稱輸入下列值：

    ![顯示變數設定範例的圖表](../media/vector-search/variables.png)

1. 選取 [儲存]**** 按鈕以儲存變更。

您已準備好將要求傳送至 Azure AI 搜尋服務。

## 建立索引

接下來，在 Postman 中建立您的索引：

1. 從側邊功能表中選取 [PUT 建立/更新索引]****。
1. 使用您稍早記下的 **search-service-name**、**index-name** 和 **api-version** 來更新 URL。
1. 選取 [本文]**** 索引標籤以查看回應。
1. 使用 URL 中的索引名稱值來設定 **index-name**，然後選取 [傳送]****。

您應該會看到類型為 **200** 的狀態代碼，表示成功的要求。

## 上傳文件

上傳文件要求中包含 108 個文件，每個文件都有一組完整的內嵌 [titleVector]**** 和 [contentVector]**** 欄位。

1. 從側邊功能表中選取 [POST 上傳文件]****。
1. 如同先前一樣使用 **search-service-name**、**index-name** 和 **api-version** 來更新 URL。
1. 選取 [本文]**** 索引標籤以查看回應，然後選取 [傳送]****。

您應該會看到類型為 **200** 的狀態代碼，以顯示您的要求成功。

## 執行查詢

1. 現在請嘗試在側邊功能表上執行下列查詢。 若要這樣做，請務必每次與先前一樣更新 URL，並選取 [傳送]**** 來傳送要求：

    - 單一向量搜尋
    - 使用篩選的單一向量搜尋
    - 簡單混合式搜尋
    - 使用篩選的簡單混合式搜尋
    - 跨欄位搜尋
    - 多重查詢搜尋

1. 選取 [本文]**** 索引標籤以查看回應並檢視結果。

針對成功的要求，您應該會看到類型為 **200** 的狀態代碼。
