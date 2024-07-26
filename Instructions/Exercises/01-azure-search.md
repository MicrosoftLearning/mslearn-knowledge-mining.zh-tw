---
lab:
  title: 建立 Azure AI 搜尋服務解決方案
  module: Module 12 - Creating a Knowledge Mining Solution
---

# 建立 Azure AI 搜尋服務解決方案

所有組織都依賴資訊來做出決策、回答問題，以及有效率地運作。 大部分組織的問題並非缺少資訊，而是從大量文件、資料庫與儲存資訊的其他來源，尋找及擷取資訊的挑戰。

例如，假設 *Margie's Travel* 是一間旅遊機構，專門組織全球各地城市的旅程。 該公司經過一段時間後，累積了大量的資訊文件，例如摺頁冊，以及客戶提交的旅館評論。 此資料是旅遊專員與客戶在規劃旅程時的寶貴深入資料來源，但大量的資料可能會讓您難以找到相關資訊來回答具體的客戶問題。

為解決此挑戰，Margie's Travel 可使用 Azure AI 搜尋服務實作解決方案，其中文件均已使用 AI 技能編制索引並進行擴充，以便您更容易對其進行搜尋。

## 建立 Azure 資源

您將為 Margie's Travel 建立解決方案需要 Azure 訂用帳戶中的下列資源：

- **Azure AI 搜尋服務**資源，此資源用於管理索引編製和查詢。
- **Azure AI 服務**資源，可提供 AI 服務，讓您的搜尋解決方案可用來使用 AI 產生的深入解析來擴充資料來源中的資料。
- 具 Blob 容器的**儲存體帳戶**，此帳戶會儲存要搜尋的文件。

> **重要**：Azure AI 搜尋服務和 Azure AI 服務資源必須位於相同的位置！

### 建立 Azure AI 搜尋服務資源

1. 在網頁瀏覽器上開啟位於 `https://portal.azure.com` 的 Azure 入口網站，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 選取 [&#65291;建立資源]**** 按鈕，搜尋*搜尋*，並使用下列設定建立 [Azure AI 搜尋服務]**** 資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*建立新資源群組 (如果您使用受限制的訂用帳戶，則可能沒有建立新資源群組的權限 - 請使用所提供的資源群組)*
    - **服務名稱**：*輸入唯一名稱*
    - **位置**：*選取位置 - 請注意您的 Azure AI 搜尋服務和 Azure AI 服務資源必須位於相同的位置*
    - **定價層**：基本

3. 等候部署完成，然後移至資源。
4. 檢閱 Azure 入口網站中 Azure AI 搜尋服務資源刀鋒視窗上的 [概觀]**** 頁面。 在這裡，您可以使用視覺化介面來建立、測試、管理及監視搜尋解決方案的各種元件；包括資料來源、索引、索引子和技能集。

### 建立 Azure AI 服務資源

如果您的訂用帳戶中還沒有 **Azure AI 服務**資源，則必須加以佈建。 您的搜尋解決方案會使用 AI 產生的深入解析來擴充資料存放區中的資料。

1. 退回 Azure 入口網站首頁，然後選取 **&#65291；建立資源**按鈕，搜尋 *Azure AI 服務*，並使用下列設定建立 **Azure AI 服務多服務帳戶**資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*與您 Azure AI 搜尋服務資源相同的資源群組*
    - **區域**：*與您 Azure AI 搜尋服務資源相同的位置*
    - **名稱**：輸入唯一名稱**
    - **定價層**:標準 S0
2. 選取必要的核取方塊並建立資源。
3. 等候部署完成，然後檢視部署的詳細資料。

### 建立儲存體帳戶

1. 返回 Azure 入口網站的首頁，然後選取 [+ 建立資源]**** 按鈕，搜尋*儲存體帳戶*，並使用下列設建立 [儲存體帳戶]**** 資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：**與您 Azure AI 搜尋服務和 Azure AI 服務資源相同的資源群組*
    - **儲存體帳戶名稱**：*輸入唯一名稱*
    - **區域**：*選擇任何可用的區域*
    - **效能**：標準
    - **複寫**：本地備援儲存體 (LRS)
    - 在 [進階]**** 索引標籤上，核取 [允許在個別容器上啟用匿名存取]** 旁的方塊
2. 等候部署完成，然後移至資源。
3. 在 [概觀]**** 頁面上，請記住**訂用帳戶識別碼**，這可用於識別佈建儲存體帳戶的訂用帳戶。
4. 在 [存取金鑰]**** 頁面上，請注意以針對您的儲存體帳戶產生兩個金鑰。 然後選取 [顯示金鑰]**** 以檢視金鑰。

    > **秘訣**：將 [儲存體帳戶]**** 刀鋒視窗保持開啟，在下一個程序中您將需要訂用帳戶識別碼和其中一個金鑰。

## 準備在 Visual Studio Code 中開發應用程式

您將使用 Visual Studio Code 開發搜尋應用程式。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

> **秘訣**：如果您已複製 **mslearn-knowledge-mining** 存放庫，請在 Visual Studio Code 中開啟它。 否則，請遵循下列步驟將其複製到您的開發環境。

1. 啟動 Visual Studio Code。
1. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
1. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
1. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。

## 將文件上傳至 Azure 儲存體

現在您已具備必要的資源，可以將某些文件上傳至 Azure 儲存體帳戶。

1. 在 Visual Studio Code [檔案總管]**** 窗格中，展開 [Labfiles\01-azure-search]**** 資料夾，然後選取 [UploadDocs.cmd]****。
2. 編輯批次檔，將您先前建立之儲存體帳戶的適當訂用帳戶識別碼、Azure 儲存體帳戶名稱和 Azure 儲存體帳戶金鑰值取代 **YOUR_SUBSCRIPTION_ID**、**YOUR_AZURE_STORAGE_ACCOUNT_NAME** 和 **YOUR_AZURE_STORAGE_KEY** 預留位置。
3. 儲存您的變更，然後以滑鼠右鍵按一下 [01-azure-search]**** 資料夾，然後開啟整合式終端。
4. 輸入下列命令，以使用 Azure CLI 登入您的 Azure 訂用帳戶。

    ```powershell
    az login
    ```

    將會開啟網頁瀏覽器索引標籤，並提示您登入 Azure。 如此操作後，接著關閉瀏覽器索引標籤並返回 Visual Studio Code。

5. 輸入下列命令以執行批次檔。 這會在您的儲存體帳戶中建立 Blob 容器，並將 **data** 資料夾中的文件上傳至其中。

    ```powershell
    .\UploadDocs.cmd
    ```

## 將文件編制索引

現在您已將文件準備就緒，可以透過對其編制索引來建立搜尋解決方案。

1. 在 Azure 入口網站中，瀏覽至您的 Azure AI 搜尋服務資源。 然後，在其 [概觀]**** 頁面上，選取 [匯入資料]****。
2. 在 [連接到您的資料]**** 頁面上，在 [資料來源]**** 清單上選取 [Azure Blob 儲存體]****。 然後使用下列值完成資料存放區詳細資料：
    - **資料來源**：Azure Blob 儲存體
    - **資料來源名稱**：margies-data
    - [要擷取的資料]****：內容和中繼資料
    - [剖析模式]****；預設
    - **連接字串**：*選取 [請選擇現有的連線]****。然後選取儲存體帳戶，最後再選取 UploadDocs.cmd 指令碼建立的 **margies** 容器。*
    - [受控識別驗證]****：無
    - **容器名稱**：margies
    - [Blob 資料夾]****：*將此保留為空白*
    - **描述**：Margie's Travel 網站的手冊和評論。
3. 繼續進行下個步驟 (*新增認知技能*)。
4. 在 [附加 Azure AI 服務]**** 區段中，選取您的 Azure AI 服務資源。
5. 在 [新增擴充]**** 區段中：
    - 將 [技能集名稱]**** 變更為 **margies-skillset**。
    - 請選取 [啟用 OCR 並將所有文字合併到 merged_content 欄位中]**** 選項。
    - 請確保 [來源資料欄位]**** 已設定為 [merged_content]****。
    - 將 [擴充資料細微性層級]**** 保留為 [來源欄位]****，這會設定要編制索引文件的整個內容；但請注意，您可以變更此選項，以更細微層級擷取資訊，例如頁面或句子。
    - 選取下列擴充欄位：

        | 認知技能 | 參數 | 欄位名稱 |
        | --------------- | ---------- | ---------- |
        | 擷取位置名稱 | | 位置 |
        | 擷取關鍵片語 | | 關鍵片語 |
        | 偵測語言 | | language |
        | 從影像產生標籤 | | imageTags |
        | 從影像產生標題 | | imageCaption |

6. 請重複檢查您的選項 (稍後將難以變更)。 然後繼續進行下一個步驟 (*自訂目標索引*)。
7. 將**索引名稱**變更為 **margies-index**。
8. 請確定 [金鑰]**** 設定為 **metadata_storage_path** ，並將 [建議名稱]**** 保留空白，而 [搜尋]**** 模式則保留預設值。
9. 對索引欄位進行下列變更，讓所有其他欄位保持其預設設定 (**重要**：您可能需要向右捲動以查看整個資料表)：

    | 欄位名稱 | 可擷取 | 可篩選 | 可排序 | 可面向化 | 可搜尋 |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | metadata_author | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | 位置 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | 關鍵片語 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | language | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |

10. 請仔細檢查您的選取項目，並特別注意確保已針對每個欄位選取正確的 [可擷取]****、[可篩選]****、[可排序]****、[可 Facet]**** 和 [可搜尋]**** 選項 (稍後很難進行變更)。 然後繼續進行下一個步驟 (*建立所引子*)。
11. 將**索引子名稱**變更為 **margies-index**。
12. 維持將 [排程]**** 設為 [一次]****。
13. 展開 [進階]**** 選項，並確定已選取 [Base-64 編碼索引鍵]**** 選項 (通常編碼索引鍵可讓索引更有效率)。
14. 選取 [提交]**** 以建立資料來源、技能集、索引和索引子。 索引子會自動執行並執行索引管線，並執行下列動作：
    1. 從資料來源擷取文件中繼資料欄位和內容
    2. 執行認知技能的技能集，以產生額外的擴充欄位
    3. 將擷取的欄位對應至索引。
15. 在左側檢視**索引子**網頁，應該會顯示新建立的 **margies-indexer**。 等候幾分鐘，然後按一下 &orarr;[重新整理]****，直到 [狀態]**** 表示成功。

## 搜尋索引

現在您已擁有索引，您可以加以搜尋。

1. 在 Azure AI 搜尋服務資源的 [概觀]**** 頁面頂端，選取 [搜尋檔案總管]****。
2. 在 [搜尋檔案總管] 的 [查詢字串]**** 方塊中輸入 `*` (單一星號)，然後選取 [搜尋]****。

    此查詢會以 JSON 格式擷取索引中的所有文件。 檢查結果並記下每個文件的欄位，其中包含您選取認知技能所擷取的文件內容、中繼資料和擴充資料。

3. 在 [檢視]**** 功能表中，選取 [JSON 檢視]****，並注意搜尋的 JSON 要求會顯示如下：

    ```json
    {
      "search": "*"
    }
    ```

1. 修改 JSON 要求以包含 **count** 參數，如下所示：

    ```json
    {
      "search": "*",
      "count": true
    }
    ```

1. 提交修改後的搜尋。 這次結果會包含 **@odata.count** 結果頂端的欄位，指出搜尋所傳回的文件數目。

4. 請嘗試下列查詢：

    ```json
    {
      "search": "*",
      "count": true,
      "select": "metadata_storage_name,metadata_author,locations"
    }
    ```

    這次結果只包含文件內容中所提及的檔案、作者和任何位置。 檔案名稱和作者位於從來源文件擷取的 **metadata_storage_name** 和 **metadata_author** 欄位。 **位置**欄位是由認知技能所產生。

5. 現在請嘗試下列查詢字串：

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name,keyphrases"
    }
    ```

    此搜尋會在任何可搜尋的欄位中尋找提及「紐約」的文件，並傳回文件中的檔案名稱和關鍵字組。

6. 讓我們再嘗試另一個查詢：

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name",
      "filter": "metadata_author eq 'Reviewer'"
    }
    ```

    此查詢會傳回*評論者*所撰寫任何文件的檔案名稱，該文件提及「紐約」。

## 探索和修改搜尋元件的定義

搜尋解決方案的元件是以 JSON 定義為基礎，您可以在Azure 入口網站中進行檢視和編輯。

雖然您可以使用入口網站來建立和修改搜尋解決方案，但通常會想要在 JSON 中定義搜尋物件，並使用 Azure AI 服務 REST 介面來進行建立和修改。。

### 取得 Azure AI 搜尋服務資源的端點和金鑰

1. 在Azure 入口網站中，返回 Azure AI 搜尋服務資源的 [概觀]**** 頁面；然後在頁面頂端的區段中，尋找資源的 **URL**(看起來像是 **https://resource_name.search.windows.net**)，然後將其複製到剪貼簿。
2. 在 Visual Studio Code [檔案總管] 窗格中，展開 [01-azure-search]**** 資料夾及其 [modify-search]**** 子資料夾，然後選取 [modify-search.cmd]**** 加以開啟。 您將使用此指令檔來執行將 JSON 提交至 Azure AI 服務 REST 介面的 *cURL* 命令。
3. 在 **modify-search.cmd** 中，將 **YOUR_SEARCH_URL** 預留位置取代為您複製到剪貼簿的 URL。
4. 在Azure 入口網站中，檢視 Azure AI 搜尋服務資源的 [金鑰]**** 頁面，並將 [主要管理金鑰]**** 複製到剪貼簿。
5. 在 Visual Studio Code 中將 **YOUR_ADMIN_KEY** 預留位置取代為您複製到剪貼簿的金鑰。
6. 將變更儲存至 **modify-search.cmd** (但不要執行！)

### 檢閱和修改技能集

1. 在 Visual Studio Code 的 **modify-search** 資料夾中，開啟 **skillset.json**。 這會顯示 **margies-skillset** 的 JSON 定義。
2. 在技能定義頂端，請注意 **cognitiveServices** 物件，該物件是用來將 Azure AI 服務資源連線到技能。
3. 在 Azure 入口網站中，開啟您的 Azure AI 服務資源 (<u>並非</u>您的 Azure AI 搜尋服務資源！) 並檢視其 [金鑰]**** 頁面。 然後將 [金鑰 1]**** 複製到剪貼簿。
4. 在 Visual Studio Code 的 **skillset.json** 中，將 **YOUR_COGNITIVE_SERVICES_KEY** 預留位置取代為您複製到剪貼簿的 Azure AI 服務金鑰。
5. 捲動 JSON 檔案，指出該檔案包含您在 Azure 入口網站中使用 Azure AI 搜尋服務使用者介面所建立技能的定義。 在技能清單底部，已使用下列定義新增額外的技能：

    ```json
    {
        "@odata.type": "#Microsoft.Skills.Text.V3.SentimentSkill",
        "defaultLanguageCode": "en",
        "name": "get-sentiment",
        "description": "New skill to evaluate sentiment",
        "context": "/document",
        "inputs": [
            {
                "name": "text",
                "source": "/document/merged_content"
            },
            {
                "name": "languageCode",
                "source": "/document/language"
            }
        ],
        "outputs": [
            {
                "name": "sentiment",
                "targetName": "sentimentLabel"
            }
        ]
    }
    ```

    新的技能會命名為 **get-sentiment**，而針對文件中的每一個**文件**層級，此項目會評估所編制索引文件 **merged_content** 欄位中所找到的文字 (其中包含來源內容，以及從內容中影像擷取的任何文字)。 此項目會使用文件的擷取**語言** (預設值為英文)，並評估內容的情緒標籤。 情緒標籤的值可以是「正面」、「負面」、「中性」或「混合」。 此標籤接著會輸出為名為 **sentimentLabel** 的新欄位。

6. 儲存您對 **skillset.json** 所做的變更。

### 檢閱和修改索引

1. 在 Visual Studio Code 的 **modify-search** 資料夾中，開啟 **index.json**。 這會顯示 **margies-index** 的 JSON 定義。
2. 捲動索引並檢視欄位定義。 有些欄位是以來源文件的中繼資料和內容為基礎，而其他欄位則是技能集中的技能結果。
3. 在您在 Azure 入口網站中定義的欄位清單結尾，請注意，已新增兩個額外的欄位：

    ```json
    {
        "name": "sentiment",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "sortable": true
    },
    {
        "name": "url",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "searchable": false,
        "sortable": false
    }
    ```

4. **情緒**欄位將用來新增技能集的 **get-sentiment** 技能輸出。 **URL** 欄位根據從資料來源擷取的 **metadata_storage_path** 值，將每個已編制索引文件的 URL 新增至索引。 請注意，索引已經包含 **metadata_storage_path** 欄位，但會作為索引鍵並以 Base-64 編碼使用，使其有效率作為索引鍵使用，但如想將其作為欄位使用該實際 URL 值，則用戶端應用程式須將其解碼。 新增未編碼值的第二個欄位可解決此問題。

### 檢閱和修改索引子

1. 在 Visual Studio Code 的 **modify-search** 資料夾中，開啟 **index.json**。 如此會將 **margies-indexer** 的 JSON 定義，即從文件內容和中繼資料擷取的欄位 (在 **fieldMappings** 區段) 以及技能集中的技能所擷取的值 (在 **outputFieldMappings** 區段) 顯示於索引中的欄位。
2. 在 **fieldMappings** 清單中，請注意將 **metadata_storage_path** 值對應至 Base-64 編碼的索引鍵欄位。 當您指派 **metadata_storage_path** 作為索引鍵，並選取該選項以編碼 Azure 入口網站的索引鍵時，就會建立此項目。 此外，新的對應會明確將相同的值對應至 **URL** 欄位，但不含 Base-64 編碼：

    ```json
    {
        "sourceFieldName" : "metadata_storage_path",
        "targetFieldName" : "url"
    }    
    ```

    來源文件中所有其他中繼資料和內容欄位都會隱含地對應至索引中相同名稱的欄位。

3. 檢閱 **ouputFieldMappings ** 區段，此區段會將技能集中的技能輸出對應至索引欄位。 其中大部分都反映您在使用者介面中所做的選擇，但已新增下列對應，以將情緒技能所擷取的 **sentimentLabel** 值對應至您新增至索引的**情緒**欄位：

    ```json
    {
        "sourceFieldName": "/document/sentimentLabel",
        "targetFieldName": "sentiment"
    }
    ```

### 使用 REST API 更新搜尋解決方案

1. 以滑鼠右鍵按一下 **modify-search** 資料夾，然後開啟整合式終端機。
2. 在 **modify-search** 資料夾的終端機窗格中，輸入下列命令以執行 **modify-search.cmd** 指令碼，此項目會將 JSON 定義提交至 REST 介面並起始索引。

    ```powershell
    ./modify-search
    ```

3. 當指令碼完成時，請返回 Azure 入口網站中 Azure AI 搜尋服務資源的 [概觀]**** 頁面，然後檢視 [索引子]**** 頁面。 會定期選取 [重新整理]**** 以追蹤編制索引作業的進度。 這可能需要一分鐘左右才能完成。

    *針對少部分文件，可能會顯示警告提示該文件過大無法評估情緒。情緒分析通常是在頁面或句子層級執行，而並非針對完整文件；但在此案例中，大部分的文件 (特別是旅館評論) 都足以評估有用的文件層級情感分數。*

### 查詢修改過的索引

1. 在 Azure AI 搜尋服務資源的窗格頂端，選取 [搜尋檔案總管]****。
2. 在 [搜尋總管] 的 [查詢字串]**** 方塊中，提交下列 JSON 查詢：

    ```json
    {
      "search": "London",
      "select": "url,sentiment,keyphrases",
      "filter": "metadata_author eq 'Reviewer' and sentiment eq 'positive'"
    }
    ```

    此查詢會擷取由*評論者*撰寫，並提及*倫敦*所有文件的 **url**、**sentiment** 和 **keyphrases**，該評論有正面**情緒**標籤 (換句話說，提及倫敦的正面評論)

3. 關閉 [搜尋檔案總管]**** 頁面來返回 [概觀]**** 頁面。

## 建立搜尋用戶端應用程式

既然您有實用的索引，即可透過用戶端應用程式使用該索引。 您可以藉由使用 REST 介面、透過 HTTP 提交要求和接收 JSON 格式的回應來完成此動作；或者您可以將軟體開發工具組 (SDK) 用於慣用的程式設計語言。 在此練習中，我們將使用 SDK。

> **注意**：您可以選擇使用適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

### 取得搜尋資源的端點和金鑰

1. 在 Azure 入口網站中，於 Azure AI 搜尋服務資源的 [概觀]**** 頁面上，記下 **URL** 值，此值應類似 **https://*your_resource_name*.search.windows.net**。 此為您搜尋資源的端點。
2. 請注意，在 [金鑰]**** 頁面上，有兩個**系統管理**金鑰和單一**查詢**金鑰。 *系統管理*金鑰用於建立和管理搜尋資源；*查詢*金鑰會交由僅需執行搜尋查詢的用戶端應用程式使用。

    *您需要用戶端應用程式的端點和查詢金鑰。*

### 準備使用 Azure AI 搜尋 SDK

1. 在 Visual Studio Code 的 [瀏覽器]**** 窗格中，瀏覽至 [01-azure-search]**** 資料夾，然後根據您的語言喜好設定展開 [C-Sharp]**** 或 [Python]**** 資料夾。
2. 以滑鼠右鍵按一下 **margies-travel** 資料夾，然後開啟整合式終端。 然後針對您的語言喜好設定執行適當的命令，以安裝 Azure AI 搜尋服務 SDK 套件：

    **C#**

    ```
    dotnet add package Azure.Search.Documents --version 11.1.1
    ```

    **Python**

    ```
    pip install azure-search-documents==11.0.0
    ```

3. 檢視 **margies-travel** 資料夾的內容，並注意其中包含組態設定的檔案：
    - **C#**：appsettings.json
    - **Python**：.env

    開啟組態檔並更新其所包含的組態值，以反映 Azure AI 搜尋服務資源的**端點**和**查詢金鑰**。 儲存您的變更。

### 探索程式碼以搜尋索引

**margies-travel**資料夾包含 Web 應用程式的程式碼檔案 (Microsoft C# *ASP.NET Razor* Web 應用程式或 Python *Flask*應用程式)，該檔案中會包含搜尋功能。

1. 根據您選擇的程式設計語言，在 Web 應用程式中開啟下列程式碼檔案：
    - **C#**：Pages/Index.cshtml.cs
    - **Python**：app.py
2. 在程式碼檔案頂端附近，尋找備註匯入**搜尋命名空間**，並記下已匯入的命名空間，以搭配 Azure AI 搜尋服務 SDK 使用：
3. 在 **search_query** 函式中，尋找註解**建立搜尋用戶端**，並請注意，程式碼會使用您 Azure AI 搜尋服務資源的端點和查詢金鑰來建立 **SearchClient** 物件：
4. 在 **search_query** 函式中，尋找註解**提交搜尋查詢**，並檢閱程式碼以提交具有下列選項的指定文字搜尋：
    - *搜尋模式*需要搜尋文字中的**所有**個別字。
    - 搜尋找到的文件總數會包含在結果中。
    - 結果會篩選為僅包含符合所提供篩選運算式的文件。
    - 結果會排序為指定的排序順序。
    - **metadata_author** 欄位中的每個離散值都會以 *Facet* 的形式傳回，可用來顯示預先定義的篩選值。
    - 結果中最多會包含三個擷取的 **merged_content** 和 **imageCaption** 欄位，其中已醒目提示搜尋字詞。
    - 結果僅包含指定的欄位。

### 探索程式碼以轉譯搜尋結果

Web 應用程式已經包含用來處理和轉譯搜尋結果的程式碼。

1. 根據您選擇的程式設計語言，在 Web 應用程式中開啟下列程式碼檔案：
    - **C#**：Pages/Index.cshtml
    - **Python**：templates/search.html
2. 檢查程式碼，此程式碼會呈現顯示搜尋結果的頁面。 請觀察下列項目：
    - 頁面的開頭是搜尋表單，使用者可以用來在應用程式的 Python 版本中提交新的搜尋 (此表單定義在 **base.html** 範本)，此項目可在在頁面開頭中參考。
    - 接著會轉譯第二份表單，讓使用者能夠精簡搜尋結果。 此表單的程式碼：
        - 從搜尋結果擷取並顯示文件計數。
        - 擷取 **metadata_author** 欄位的 Facet 值，並將其顯示為選項清單以供篩選。
        - 建立結果排序選項的下拉式清單。
    - 然後程式碼會逐一查看搜尋結果，並轉譯每個結果，如下所示：
        - 將 **metadata_storage_name** (檔案名稱) 欄位顯示為 **url** 欄位中位址的連結。
        - 顯示在 *merged_content* 和 **imageCaption** 欄位中所找到搜尋字詞的**醒目提示項目**，以協助在內容中顯示搜尋字詞。
        - 顯示 **metadata_author**、**metadata_storage_size**、**metadata_storage_last_modified** 和**語言**欄位。
        - 顯示文件的**情緒**標籤。 可以是正面、負面、中性或混合。
        - 顯示前五個**關鍵字** (如有任何項目)。
        - 顯示前五個**位置** (如有任何項目)。
        - 顯示前五個 **imageTags** (如有任何項目)。

### 執行 Web 應用程式

 1. 傳回 **margies-travel** 資料夾的整合式終端，然後輸入下列命令來執行程式：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    flask run
    ```

2. 在成功啟動應用程式時顯示的訊息中，遵循執行中的 Web 應用程式連結 (*http://localhost:5000/* 或 *http://127.0.0.1:5000/*)，在網頁瀏覽器中開啟 Margies Travel 網站。
3. 在 Margie's Travel 網站中，於搜尋方塊中輸入**倫敦旅館**，然後按一下 [搜尋]****。
4. 檢閱搜尋結果。 此項目會包含檔案名稱 (與檔案 URL 的超連結)、強調具有搜尋字詞的檔案內容擷取 (*倫敦*和*旅館*)，以及索引欄位中檔案的其他屬性。
5. 觀察該頁面結果包含部分使用者介面元素，可讓您精簡結果。 包括：
    - 以 *metadata_author* 欄位的 Facet 值為基礎的**篩選**。 這會示範如何使用*可 Facet* 欄位傳回 *Facet* 清單 - 欄位，其中包含一組可顯示為使用者介面中潛在篩選值的離散值。
    - 根據指定的欄位*排序*結果的功能，以及排序方向 (遞增或遞減)。 預設順序是以*相關性*為基礎，此項目根據**評分設定檔**上的 *search.score ()* 值進行計算，用以評估索引欄位中搜尋字詞的頻率和重要性。
6. 選取 [評論者]**** 篩選和 [正面到負面]**** 排序選項，然後選取 [精簡結果]****。
7. 觀察該結果將進行篩選，僅會包含評論並根據情緒標籤進行排序。
8. 在 [搜尋]**** 方塊中，輸入新搜尋**紐約的安靜旅館**，並檢閱結果。
9. 請嘗試下列搜尋字詞：
    - **倫敦的塔** (觀察到此詞彙組在部分文件中識別為*關鍵詞組*)。
    - **摩天大樓** (觀察到此單字不會出現在任何文件的實際內容中，但可在部分文件影像產生的*影像標題*和*影像標籤*中找到)。
    - **莫哈韋沙漠** (觀察到此詞彙組在部分文件中識別為*位置*)。
10. 關閉包含 Margie's Travel 網站的瀏覽器索引標籤，並返回 Visual Studio Code。 然後在 **margies-travel** 資料夾的 Python 終端機中 (終端機上正在執行 dotnet 或 flask 應用程式)，輸入 Ctrl+C 以停止應用程式。

## 刪除練習資源

您已完成練習，請刪除所有不再需要的資源。 刪除 Azure 資源:

1. 在 **Azure 入口網站**，中選取 [資源群組]。
1. 選取不需要的資源群組，然後選取 [刪除資源群組]****。

## 其他相關資訊

若要深入了解 Azure AI 搜尋服務，請參閱 [Azure AI 搜尋服務文件](https://docs.microsoft.com/azure/search/search-what-is-azure-search)。
