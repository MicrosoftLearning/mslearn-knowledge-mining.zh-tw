---
lab:
  title: 使用 Azure AI 搜尋服務建立知識存放區
  module: Module 12 - Creating a Knowledge Mining Solution
---

# 使用 Azure AI 搜尋服務建立知識存放區

Azure AI 搜尋服務使用 AI 技能的擴充管線，從文件擷取 AI 產生的欄位，並將其包含在搜尋索引中。 雖然索引常視為是索引編製流程中的主要輸出，但它所包含的擴充資料也能以其他方式來使用。 例如：

- 因為索引基本上是 JSON 物件的集合，每個物件都代表一個索引記錄。因此將物件匯出為 JSON 檔案，藉此利用如 Azure Data Factory 等的工具整合到資料協調流程中，便是個實用的做法。
- 建議您將索引記錄標準化為以資料表組成的關聯式結構描述，藉此使用如 Microsoft Power BI 等的工具進行分析和報告。
- 建議您將在編製索引流程時從文件中擷取出的內嵌影像儲存為檔案。

在本次練習中，您將會針對 Margie's Travel** 進行知識存放區的實作。這是一家虛構的旅行社，他們使用摺頁冊和旅館評論中的資訊來協助客戶進行旅遊規劃。

## 準備在 Visual Studio Code 中開發應用程式

您將使用 Visual Studio Code 開發搜尋應用程式。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

> **秘訣**：如果您已複製 **mslearn-knowledge-mining** 存放庫，請在 Visual Studio Code 中開啟它。 否則，請遵循下列步驟將其複製到您的開發環境。

1. 啟動 Visual Studio Code。
1. 開啟選擇區 (SHIFT+CTRL+P) 並執行 **Git：複製 ** 命令，將 `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` 存放庫複製到本機資料夾 (哪個資料夾無關緊要)。
1. 複製存放庫後，請在 Visual Studio Code 中開啟此資料夾。
1. 等候其他檔案安裝以支援存放庫中的 C# 程式碼專案。

    > **注意**：如果系統提示您新增必要的資產來組建和偵錯，請選取 [現在不要]****。

## 建立 Azure 資源

> **注意**：如果您先前已完成**[建立 Azure AI 搜尋服務解決方案](01-azure-search.md)** 練習，而且這些 Azure 資源仍在您的訂用帳戶中，您可以略過本節，然後從**建立搜尋解決方案**一節開始。 否則，請遵循下列步驟來佈建必要的 Azure 資源。

1. 在網頁瀏覽器上開啟位於 `https://portal.azure.com` 的 Azure 入口網站，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
2. 檢視您訂用帳戶中的**資源群組**。
3. 如果您使用受限制的訂用帳戶提供資源群組，請選取 [資源群組] 以檢視其屬性。 否則，請用您選擇的名稱建立新的資源群組，接著在建立後前往該群組。
4. 在資源群組的 [概觀] **** 頁面上，記下 [訂用帳戶識別碼] **** 和 [位置]****。 後續步驟中，您將需要這些值以及資源群組的名稱。
5. 在 Visual Studio Code 中，展開 **Labfiles/03-knowledge-store** 資料夾，然後選取 **setup.cmd**。 您將使用此批次指令碼來執行 Azure 命令列介面 (CLI) 命令，以建立所需的 Azure 資源。
6. 以滑鼠右鍵按一下 **03-knowledge-store** 資料夾，然後選取 [在整合式終端機中開啟]****。
7. 在終端窗格中，輸入下列命令來建立 Azure 訂用帳戶的已驗證連線。

    ```powershell
    az login --output none
    ```

8. 出現提示時，登入您的 Azure 訂用帳戶。 然後返回 Visual Studio Code，並等候登入流程完成。
9. 執行下列命令以列出 Azure 位置。

    ```powershell
    az account list-locations -o table
    ```

10. 在輸出中，尋找對應資源群組位置的 **Name** 值 (例如，*美國東部*的對應名稱為 *eastus*)。
11. 在 **setup.cmd** 指令碼中，將 **subscription_id**、**resource_group** 及 **location** 變數宣告修改為您的訂用帳戶識別碼、資源群組名及位置名稱的適當值。 然後儲存您的變更。
12. 在 **03-knowledge-store** 資料夾的終端機中，輸入下列命令以執行指令碼：

    ```powershell
    ./setup
    ```
    > **注意**：搜尋 CLI 模組處於預覽狀態，而且可能會卡在 *- Running ..* 是最困難的操作。 如果此狀況持續超過 2 分鐘，請按 CTRL+C 取消長時間執行的作業，然後在系統詢問您是否要終止指令碼時，選取 [N]****。 該作業之後應該會順利完成。
    >
    > 如果指令碼失敗，請確定您已使用正確的變數名稱儲存指令碼，然後再試一次。

13. 當指令碼完成時，請檢閱其顯示的輸出，並記下有關 您 Azure 資源的下列資訊 (您稍後會需要用到這些值)：
    - 儲存體帳戶名稱
    - 儲存體連接字串
    - Azure AI 服務帳戶
    - Azure AI 服務金鑰
    - 搜尋服務端點
    - 搜尋服務系統管理金鑰
    - 搜尋服務查詢金鑰

14. 在 Azure 入口網站中，重新整理資源群組，並驗證其中包含 Azure 儲存體帳戶、Azure AI 服務資源及 Azure AI 搜尋服務資源。

## 建立搜尋解決方案

現在您已擁有必要的 Azure 資源，您可以建立包含下列元件的搜尋解決方案：

- 參考 Azure 儲存體容器中文件的**資料來源**。
- **技能集**，定義技能的擴充管線，以從文件中擷取 AI 產生的欄位。 技能集也會定義將在*知識存放區*中產生的*投影*。
- 定義可搜尋文件資料列的**索引**。
- **索引子**，從資料來源擷取文件、套用技能集，並填入索引。 編製索引的程序也會保存知識存放區技能集中定義的投影。

在本練習中，您將使用 Azure AI 搜尋服務 REST 介面來提交 JSON 要求，以建立這些元件。

### 準備 REST 作業的 JSON

您將使用 REST 介面來提交 Azure AI 搜尋服務元件的 JSON 定義。

1. 在 Visual Studio Code 的 **03-knowledge-store** 資料夾中，展開 **create-search** 資料夾，然後選取 **data_source.json**。 此檔案包含名為 **margies-knowledge-data** 資料來源的 JSON 定義。
2. 以 Azure 儲存體帳戶的連接字串取代 **YOUR_CONNECTION_STRING** 預留位置，結果應該如下所示：

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *您可以在 Azure 入口網站的 [存取金鑰] **** 頁面上，找到您儲存體帳戶的連接字串。*

3. 儲存並關閉更新後的 JSON 檔案。
4. 在 [create-search] **** 資料夾中，開啟 [skillset.json]****。 此檔案包含名為 **margies-knowledge-skillset** 技能集的 JSON 定義。
5. 在技能定義頂端的 **cognitiveServices** 元素中，將 **YOUR_COGNITIVE_SERVICES_KEY** 預留位置取代為 Azure AI 服務資源的其中一個金鑰。

    *您可以在 Azure 入口網站中 Azure AI 服務資源的 [金鑰與端點]**** 頁面上找到金鑰。*

6. 在技能集的技能集合結束時，請尋找名為 **define-projection** 的 **Microsoft.Skills.Util.ShaperSkill** 技能。 此技能會定義擴充資料的 JSON 結構，在索引子所處理每個文件的知識存放區上，管線的投影會使用此資料。
7. 在技能集檔案底部，可以看到技能集也包含 **knowledgeStore** 定義，其中包含知識存放區建立所在 Azure 儲存體帳戶的連接字串，以及**投影**的集合。 此技能集包含三個*投影群組*：
    - 第一個群組包含根據技能集成形技能 **knowledge_projection** 輸出的*物件*投影。
    - 第二個群組包含根據擷取自文件影像資料 *normalized_images* 集合的**檔案**投影。
    - 第三個群組包含下列「資料表」** 投影：
        - **KeyPhrases**：包含自動產生的索引鍵資料行，以及對應至成形技能 **knowledge_projection/key_phrases/** 集合輸出的 **keyPhrase** 資料行。
        - **位置**：包含自動產生的索引鍵資料行，以及對應至成形技能 **knowledge_projection/key_phrases/** 集合輸出的 **location** 資料行。
        - **ImageTags**：包含自動產生的索引鍵資料行，以及對應至成形技能 **knowledge_projection/image_tags/** 集合輸出的 **tag** 資料行。
        - **Docs**：包含自動產生的索引鍵資料行，以及尚未指派給資料表成形技能的所有 **knowledge_projection** 輸出值。
8. 以儲存體帳戶的連接字串取代 **storageConnectionString** 值的 **YOUR_CONNECTION_STRING** 預留位置。
9. 儲存並關閉更新後的 JSON 檔案。
10. 在 [create-search] **** 資料夾中，開啟 [index.json]****。 此檔案包含名為 **margies-knowledge-index** 索引的 JSON 定義。
11. 檢閱索引的 JSON，然後直接關閉該檔案而不做任何變更。
12. 在 [create-search] **** 資料夾中，開啟 [indexer.json]****。 此檔案包含名為 **margies-knowledge-indexer** 索引子的 JSON 定義。
13. 檢閱索引子的 JSON，然後直接關閉該檔案而不做任何變更。

### 提交 REST 要求

現在您已備妥定義搜尋解決方案元件的 JSON 物件，接著可以將 JSON 檔案提交至 REST 介面以建立元件。

1. 在 [create-search] **** 資料夾中，開啟 [create-search.cmd]****。 此批次指令碼會使用 cURL 公用程式，將 JSON 定義提交至您 Azure AI 搜尋服務資源的 REST 介面。
2. 將 **YOUR_SEARCH_URL** 和 **YOUR_ADMIN_KEY** 變數預留位置取代為您 Azure AI 搜尋服務資源的 **URL** 及其中一個**系統管理員金鑰**。

    *您可以在 Azure 入口網站中 Azure AI 搜尋服務資源的 [概觀]**** 和 [金鑰]**** 頁面上找到這些值。*

3. 儲存已更新的批次檔案。
4. 以滑鼠右鍵按一下 [create-search] **** 資料夾，然後選取 [在整合式終端中開啟]****。
5. 在 [create-search] **** 資料夾的終端窗格中，輸入下列命令以執行批次指令碼。

    ```powershell
    ./create-search
    ```

6. 當指令碼完成時，在 Azure 入口網站的 Azure AI 搜尋服務資源頁面上，選取 [索引子]**** 頁面，並等候編製索引程序完成。

    *您可以選取 [重新整理] **** 以追蹤編制索引作業的進度。這可能需要一分鐘左右才能完成。*

    > **秘訣**：如果指令碼失敗，請檢查您在 **data_source.json** 和 **skillset.json** 檔案，以及 **create-search.cmd** 檔案中新增的預留位置。 更正存在的錯誤之後，您可能需要使用 Azure 入口網站使用者介面，先刪除搜尋資源中建立的任何元件，再重新執行指令碼。

## 檢視知識存放區

執行使用技能集建立知識存放區的索引子之後，索引編製程序所擷取的擴充資料會保存在知識存放區投影中。

### 檢視物件投影

Margie's Travel 技能集中定義的*物件*投影是由每個已編製索引文件的 JSON 檔案所組成。 這些檔案會儲存在技能集定義所指定 Azure 儲存體帳戶中的 Blob 容器中。

1. 在 Azure 入口網站中，檢視您先前建立的 Azure 儲存體帳戶。
2. 選取 [儲存體瀏覽器]**** 索引標籤 (左側窗格中)，以在 Azure 入口網站的儲存體總管介面中檢視儲存體帳戶。
3. 展開 [Blob 容器]**** 以檢視儲存體帳戶中的容器。 除了儲存來源資料的 **margies** 容器之外，還有兩個新的容器：**margies-images** 和 **margies-knowledge**。 這些容器是由編製索引程序所建立。
4. 選取 **margies-knowledge** 容器。 其中應該會包含每個已編製索引的資料夾。
5. 開啟任何資料夾，然後下載並開啟其中包含的 **knowledge-projection.json** 檔案。 每個 JSON 檔案都包含已編製索引文件檔的表示法，包括技能集所擷取的擴充資料，如下所示。

```json
{
    "file_id":"abcd1234....",
    "file_name":"Margies Travel Company Info.pdf",
    "url":"https://store....blob.core.windows.net/margies/...pdf",
    "language":"en",
    "sentiment":0.83164644241333008,
    "key_phrases":[
        "Margie’s Travel",
        "Margie's Travel",
        "best travel experts",
        "world-leading travel agency",
        "international reach"
        ],
    "locations":[
        "Dubai",
        "Las Vegas",
        "London",
        "New York",
        "San Francisco"
        ],
    "image_tags":[
        "outdoor",
        "tree",
        "plant",
        "palm"
        ]
}
```

建立這類*物件*投影的能力可讓您產生可併入企業資料分析解決方案的擴充資料物件，例如，將 JSON 檔案內嵌至 Azure Data Factory 管線，以便進一步處理或載入資料倉儲。

### 檢視檔案投影

技能集中定義的*檔案*投影會針對在編制索引過程中從文件擷取的每個影像建立 JPEG 檔案。

1. 在 Azure 入口網站的 [儲存體瀏覽器]** 介面中，選取 **margies-images** Blob 容器。 此容器包含每個包含影像的文件資料夾。
2. 開啟任何資料夾並檢視其內容：每個資料夾至少會包含一個 \*.jpg 檔案。
3. 開啟任何影像檔案，以確認是否包含從文件擷取的影像。

這種產生*檔案*投影的方式，可讓編制索引成為從大量文件擷取內嵌影像的有效方法。

### 檢視資料表投影

技能集中定義的*資料表*投影會形成擴充資料的關聯式結構敘述。

1. 在 Azure 入口網站的 [儲存體瀏覽器]** 介面中，展開 [資料表]****。
2. 選取 **docs** 資料表以檢視其資料行。 資料行包含一些標準 Azure 儲存體資料表資料行，若要隱藏這些資料行，請修改 [資料行選項]**** 並只選取下列資料行：
    - **document_id** (編製索引程序自動產生的索引鍵資料行)
    - **file_id** (編碼檔案 URL)
    - **file_name** (從文件中繼資料擷取的檔案名稱)
    - **language** (文件的撰寫語言)
    - **sentiment** 針對文件計算出的情緒分數。
    - **url** Azure 儲存體內文件 Blob 的 URL。
3. 檢視編製索引程序所建立的其他資料表：
    - **ImageTags** (包含個別影像標籤的資料列，其中包含標籤所在文件的 **document_id**)。
    - **KeyPhrases** (包含每個個別關鍵片語的資料列，其中包含片語所在文件的 **document_id**)。
    - **Locations** (包含個別位置的資料列，其中包含位置所在文件的 **document_id**)。

建立*資料表*投影的能力可讓您建置可查詢關係結構描述的分析與報告解決方案；例如，使用 Microsoft Power BI。 自動產生的索引鍵資料行可用來聯結查詢中的資料表，例如，傳回特定文件中提及的所有位置。

## 刪除練習資源

您已完成練習，請刪除所有不再需要的資源。 刪除 Azure 資源:

1. 在 **Azure 入口網站**，中選取 [資源群組]。
1. 選取不需要的資源群組，然後選取 [刪除資源群組]****。

## 其他相關資訊

若要深入了解如何使用 Azure AI 搜尋服務建立知識存放區，請參閱 [Azure AI 搜尋服務文件](https://docs.microsoft.com/azure/search/knowledge-store-concept-intro)。
