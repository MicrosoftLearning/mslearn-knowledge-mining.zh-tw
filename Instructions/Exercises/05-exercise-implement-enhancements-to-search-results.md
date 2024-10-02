---
lab:
  title: 實作搜尋結果的增強功能
---

# 實作搜尋結果的增強功能

您有假日預約應用程式使用的現有搜尋服務。 您已瞭解搜尋結果的相關性會影響您取得的預約數目。 您最近也已在葡萄牙新增旅館，因此想要提供葡萄牙文作為支援的語言。

在此練習中，您將新增評分設定檔，以改善搜尋結果的相關性。 然後，您將使用 Azure AI 服務來新增所有旅館的葡萄牙文描述。

> **注意** 若要完成此練習，您需要 Microsoft Azure 訂用帳戶。 如果尚未有訂用帳戶，則可在 [https://azure.com/free](https://azure.com/free?azure-portal=true) 註冊免費試用版。

## 建立 Azure 資源

您將建立 Azure AI 搜尋服務並匯入範例旅館資料。

1. 登入 [Azure 入口網站](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true)。
1. 選取 [+ 建立資源]。
1. 搜尋「搜尋」****，然後選取 [Azure AI 搜尋服務]****。
1. 選取 **建立**。
1. 選取 [資源群組] 底下的 [新建]****，將其命名為 **learn-advanced-search**。
1. 在 **[服務名稱]** 中，輸入 **advanced-search-service-12345**。 名稱必須是全域唯一，因此請將亂數新增至名稱結尾。
1. 選取您附近的支援 [區域]。
1. 使用 **[定價層]** 的預設值。
1. 選取 [**檢閱 + 建立**]。
1. 選取 **建立**。
1. 等候部署資源，然後選取 **[移至資源]**。

### 將範例資料匯入搜尋服務

匯入樣本資料。

1. 在 [概觀]**** 窗格上，選取 [匯入資料]****。

    ![顯示匯入資料選單的螢幕擷取畫面。](../media/05-media/import-data-new.png)
1. 在 [匯入資料]**** 窗格上的 [資料來源]**** 下拉式清單中，選取 [範例]****。
1. 選取 **[hotels-sample]**。

1. 在 [新增認知技能 (選用)]**** 索引標籤上，展開 [附加 AI 服務]****，然後選取 [建立新的 AI 服務資源]****。

    ![顯示選取、新增 Azure AI 服務的螢幕擷取畫面。](../media/05-media/add-cognitive-services-new.png)

### 建立 Azure AI 服務以支援翻譯

1. 在新索引標籤中，登入 Azure 入口網站。
1. 在 **[資源群組]** 中，選取 **learn-advanced-search**。
1. 在 **[區域]** 中，選取您為搜尋服務選擇的相同區域。
1. 在 [名稱]**** 中，輸入 **learn-cognitive-translator-12345** 或任何您偏好的名稱。 名稱必須是全域唯一，因此請將亂數新增至名稱結尾。
1. 在 **[定價層]** 中，選取 **[標準 S0]**。
1. 勾選 **[勾選此方塊即表示我確認我已閱讀並了解下列所有條款]**。
1. 選取 [**檢閱 + 建立**]。
1. 選取 **建立**。
1. 建立資源之後，請關閉索引標籤。

### 新增翻譯擴充

1. 在 **[新增認知技能 (選擇性)]** 索引標籤上，選取 [重新整理]。
1. 選取新的服務，**learn-cognitive-translator-12345**。
1. 展開 [新增擴充]**** 區段。
    ![顯示新增葡萄牙文翻譯的螢幕擷取畫面。](../media/05-media/add-translation-enrichment-new.png)
1. 選取 **[翻譯文字]**，將 **[目標語言]** 變更為 **[葡萄牙文]**，然後將 **[欄位名稱]** 變更為 **[Description_pt]**。
1. 選取 [下一步: 自訂目標索引]****。

### 變更欄位以儲存翻譯的文字

1. 在 [自訂目標索引]**** 索引標籤上，捲動至欄位清單底部，並針對 [Description_pt]**** 欄位，將 [分析器]**** 變更為 [葡萄牙文 (葡萄牙) - Microsoft]****。
1. 選取**下一步：建立索引子**。
1. 選取 [提交]****。

    系統會建立索引、執行索引子，並匯入包含範例旅館資料的 50 份文件。
1. 在 **[概觀]** 窗格中，選取 **[索引]**，然後選取 **[hotels-sample-index]**。
1. 選取 **[搜尋]** 以查看索引中所有文件的 JSON。
1. 在結果中搜尋 **Description_pt** (您可以使用 **CTRL + F** 來搜尋)，並請注意這不是英文描述的葡萄牙文翻譯，而是如下所示：

    ```json
    "Description_pt": "45",
    ```

Azure 入口網站假設文件中的第一個欄位必須翻譯。 因此，它目前使用翻譯技能來翻譯 `HotelId`。

### 更新技能集以翻譯文件中的正確欄位

1. 在頁面頂端，選取搜尋服務，**advanced-search-service-12345 | 索引**連結。
1. 在左窗格的 [搜尋管理] 底下選取 [技能]****，然後選取 [hotels-sample-skillset]****。
1. 編輯 JSON 文件，將第 11 行變更為:

    ```json
    "context": "/document/Description",
    ```

1. 將第 12 行的預設值從語言變更為英文:

    ```json
    "defaultFromLanguageCode": "en",
    ```

1. 將第 18 行上的來源欄位變更為:

    ```json
    "source": "/document/Description"
    ```

1. 選取 [儲存]。
1. 在頁面頂端，選取搜尋服務，**advanced-search-service-12345 | 技能**連結。
1. 在 **[概觀]** 窗格中，選取 **[索引子]**，然後選取 **[hotels-sample-indexer]**。
1. 選取 [索引子定義 (JSON)]。****
1. 將第 21 行上的來源欄位名稱變更為:

    ```json
    "sourceFieldName": "/document/Description/Description_pt",
    ```

1. 選取 [儲存]。
1. 選取 [重設]****，然後選取 [是]****。
1. 選取 [執行]****，然後選取 [是]****。

### 測試更新的索引

1. 在頁面頂端，選取搜尋服務，**advanced-search-service-12345 | 索引子**連結。
1. 在 **[概觀]** 窗格中，選取 **[索引]**，然後選取 **[hotels-sample-index]**。
1. 選取 **[搜尋]** 以查看索引中所有文件的 JSON。
1. 在結果中搜尋 **[Description_pt]**，並請注意，現在有葡萄牙文描述。

    ```json
    "Description_pt": "O maior resort durante todo o ano da área oferecendo mais de tudo para suas férias – pelo melhor valor!  O que você pode desfrutar enquanto estiver no resort, além das praias de areia de 1,5 km do lago? Confira nossas atividades com certeza para excitar tanto os jovens quanto os jovens hóspedes do coração. Temos tudo, incluindo ser chamado de \"Propriedade do Ano\" e um \"Top Ten Resort\" pelas principais publicações.",
    ```

1. 現在，您將搜尋具有湖景的旅館。 我們首先會使用僅傳回 `HotelName`、`Description`、`Category` 和 `Tags` 的簡單搜尋。 在 **[查詢] 字串** 中，輸入此搜尋:

    `lake + view&$select=HotelName,Description,Category,Tags&$count=true`

    查看結果，並嘗試尋找符合 `lake` 和 `view` 搜尋字詞的欄位。 請注意，此旅館及其位置:

    ```json
    {
      "@search.score": 0.9433406,
      "HotelName": "Lady Of The Lake B & B",
      "Description": "Nature is Home on the beach.  Save up to 30 percent. Valid Now through the end of the year. Restrictions and blackout may apply.",
      "Category": "Luxury",
      "Tags": [
        "laundry service",
        "concierge",
        "view"
      ]
    },
    ```

此旅館符合 `HotelName` 欄位中的字詞湖，並在 `Tags` 欄位中符合景觀。 您想要將 `Description` 欄位中增強與旅館名稱相符的字詞。 在理想情況下，此旅館應該在結果中最後一個。

## 新增評分設定檔以改善搜尋結果

1. 選取 **[評分設定檔]** 索引標籤。
1. 選取 **[+ 新增評分設定檔]**。
1. 在 **[設定檔名稱]** 中，輸入 **[boost-description-categories]**。
1. 在 [權數]**** 底下，新增下列欄位和權數：

    ![要新增至評分設定檔的權數螢幕擷取畫面。](../media/05-media/add-weights-new.png)
1. 在 **[欄位名稱]** 中，選取 **[描述]**。
1. 在 [權數]**** 中，輸入 **5**。
1. 在 **[欄位名稱]** 中，選取 **[類別]**。
1. 在 [權數]**** 中，輸入 **3**。
1. 在 **[欄位名稱]** 中，選取 **[標籤]**。
1. 在 [權數]**** 中，輸入 **2**。
1. 選取 [儲存]。
1. 在頂端選取**儲存**。

### 測試更新的索引

1. 返回 [hotels-sample-index]**** 頁面的 [搜尋總管]**** 索引標籤。
1. 在 **[查詢] 字串** 中，輸入與先前相同的搜尋:

    `lake + view&$select=HotelName,Description,Category,Tags&$count=true`

    檢查搜尋結果。

    ```json
    {
      "@search.score": 3.5707965,
      "HotelName": "Lady Of The Lake B & B",
      "Description": "Nature is Home on the beach.  Save up to 30 percent. Valid Now through the end of the year. Restrictions and blackout may apply.",
      "Category": "Luxury",
      "Tags": [
        "laundry service",
        "concierge",
        "view"
      ]
    }
    ```

    搜尋分數已從 **0.9433406** 增加到 **3.5707965**。 不過，所有其他旅館都有較高的計算分數。 此旅館現在排在結果最後。

## 清理

您已完成練習，請刪除所有不再需要的資源。

1. 在 Azure 入口網站中選取 [資源群組]。
1. 選取不再需要的資源群組，然後選取 [刪除資源群組]****。
