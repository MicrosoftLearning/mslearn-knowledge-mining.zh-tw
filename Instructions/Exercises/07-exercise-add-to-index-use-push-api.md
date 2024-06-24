---
lab:
  title: 使用推送 API 新增至索引
---

# 使用推送 API 新增至索引

您想要探索如何使用 C# 程式碼建立 Azure AI 搜尋服務索引，並將文件上傳至該索引。

在此練習中，您會複製現有的 C# 解決方案並執行，以找出上傳文件的最佳批次大小。 然後使用此批次大小，並使用執行緒方法有效地上傳文件。

> **注意** 若要完成此練習，您需要 Microsoft Azure 訂用帳戶。 如果尚未有訂用帳戶，則可在 [https://azure.com/free](https://azure.com/free?azure-portal=true) 註冊免費試用版。

## 設定 Azure 資源

若要節省時間，請選取此 Azure Resource Manager 範本，建立在稍後練習中所需的資源：

1. [將資源部署至 Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2Fmslearn-knowledge-mining%2Fmain%2FLabfiles%2F07-exercise-add-to-index-use-push-api%20lab-files%2Fazuredeploy.json) - 選取此連結以建立您的 Azure AI 資源。
    ![部署資源至 Azure 時，顯示選項的螢幕擷取畫面。](../media/07-media/deploy-azure-resources.png)
1. 在 [資源群組] 中，選取 [新建]，將群組命名為 **cog-search-language-exe**。********
1. 在 [區域]**** 中，選取靠近您的[支援區域](/azure/ai-services/language-service/custom-text-classification/service-limits#regional-availability)。
1. **資源前置詞**必須是全域唯一的，請輸入隨機數字和小寫字元前置詞，例如 **acs118245**。
1. 在 [位置]**** 中，選取您在上述步驟選擇的相同區域。
1. 選取 [**檢閱 + 建立**]。
1. 選取 **建立**。
1. 當部署完成時，選取 [移至資源群組]**** 以查看您建立的所有資源。

    ![顯示所有已部署 Azure 資源的螢幕擷取畫面。](../media/07-media/azure-resources-created.png)

### 複製 Azure AI 搜尋服務 REST API 資訊

1. 在資源清單中，選取您建立的搜尋服務。 在上述範例中，即 **acs118245-search-service**。
1. 將搜尋服務名稱複製到文字檔中。

    ![搜尋服務金鑰區段的螢幕擷取畫面。](../media/07-media/search-api-keys-exercise-version.png)
1. 在左側，選取 [金鑰]****，然後將**主要系統管理金鑰**複製到相同的文字檔中。

### 下載範例程式碼

選取 Azure 入口網站頂端的 [Cloud Shell] 按鈕，以開啟您的 Azure Cloud Shell。
> **注意** 如果您收到建立 Azure 儲存體帳戶的提示，請選取 [建立儲存體]****。

1. 啟動完成後，請在 Cloud Shell 中執行下列操作，以複製下列範例程式碼存放庫：

    ```powershell
    git clone https://github.com/Azure-Samples/azure-search-dotnet-scale.git samples
    ```

1. 執行下列操作，以變更為新建立的目錄：

    ```powershell
    cd samples
    ```

1. 然後執行：

    ```powershell
    code ./optimize-data-indexing/v11
    ```

1. 這會在 Cloud Shell 內的 `/optimize-data-indexing/v11` 資料夾中開啟程式碼編輯器。

    ![顯示設定通知的 VS Code 螢幕擷取畫面。](../media/07-media/setup-visual-studio-code-solution.png)
1. 在左側導覽中，展開 **OptimizeDataIndexing** 資料夾，然後選取 **appsettings.json** 檔案。

    ![顯示 appsettings.json 檔案內容的螢幕擷取畫面。](../media/07-media/update-app-settings.png)
1. 貼上您的搜尋服務名稱和主要管理金鑰。

    ```json
    {
      "SearchServiceUri": "https://acs118245-search-service.search.windows.net",
      "SearchServiceAdminApiKey": "YOUR_SEARCH_SERVICE_KEY",
      "SearchIndexName": "optimize-indexing"
    }
    ```

    設定檔案看起來應該會像上面這樣。
1. 按 **CTRL + S** 儲存變更。
1. 選取 **OptimizeDataIndexing.csproj** 檔案。 <!-- Added this and the next two steps in case we can't update the file in the repo that holds these (seems to be separate from the other labs)-->
1. 在第 5 行上，將 `<TargetFramework>netcoreapp3.1</TargetFramework>` 變更為 `<TargetFramework>net7.0</TargetFramework>`。 <!--- can be removed if no longer needed based on the above-->
1. 按 **CTRL + S** 儲存變更。<!--- can be removed if no longer needed based on the above-->
1. 在終端中，輸入 `cd ./optimize-data-indexing/v11/OptimizeDataIndexing`，然後按 **Enter** 以變更為正確的目錄。
1. 請選取 **Program.cs** 檔案。 然後，在終端中，輸入 `dotnet run`，然後按 **Enter**。

    ![顯示應用程式在 VS Code 中執行但出現例外狀況的螢幕擷取畫面。](../media/07-media/debug-application.png)
輸出內容會顯示在此情況下，最佳效能的批次大小是 900 份文件。 每秒達到 3.688 MB 時。

### 編輯程式碼以實作執行緒和輪詢並重試策略

程式碼已加上註解，可讓應用程式變更為使用執行緒將文件上傳至搜尋索引。

1. 請確定您已選取 **Program.cs**。

    ![顯示 Program.cs 檔案的 VS Code 螢幕擷取畫面。](../media/07-media/edit-program-code.png)
1. 將第 38 行和 39 行註解如下：

    ```csharp
    //Console.WriteLine("{0}", "Finding optimal batch size...\n");
    //await TestBatchSizesAsync(searchClient, numTries: 3);
    ```

1. 取消註解第 41 到 49 行。

    ```csharp
    long numDocuments = 100000;
    DataGenerator dg = new DataGenerator();
    List<Hotel> hotels = dg.GetHotels(numDocuments, "large");

    Console.WriteLine("{0}", "Uploading using exponential backoff...\n");
    await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8);

    Console.WriteLine("{0}", "Validating all data was indexed...\n");
    await ValidateIndexAsync(indexClient, indexName, numDocuments);
    ```

    控制批次大小和執行緒數目的程式碼為 `await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8)`。 批次大小為 1000，且執行緒為 8。

    ![顯示所有已編輯程式碼的螢幕擷取畫面。](../media/07-media/thread-code-ready.png)
    您的程式碼看起來應會如上所示。

1. 若要儲存變更，請按 **Ctrl**+**S**。
1. 選取您的終端，然後按下任何按鍵以結束執行中的程序 (如果您尚未結束)。
1. 在終端中執行 `dotnet run`。

    ![顯示主控台中已完成訊息的螢幕擷取畫面。](../media/07-media/upload-hundred-thousand-documents.png)
    應用程式會啟動八個執行緒，然後在每個執行緒完成將新訊息寫入主控台的作業時：

    ```powershell
    Finished a thread, kicking off another...
    Sending a batch of 1000 docs starting with doc 57000...
    ```

    上傳 100,000 份文件之後，應用程式會撰寫摘要 (這可能需要一些時間才能完成)：

    ```powershell
    Ended at: 9/1/2023 3:25:36 PM
    
    Upload time total: 00:01:18:0220862
    Upload time per batch: 780.2209 ms
    Upload time per document: 0.7802 ms
    
    Validating all data was indexed...
    
    Waiting for service statistics to update...
    
    Document Count is 100000
    
    Waiting for service statistics to update...
    
    Index Statistics: Document Count is 100000
    Index Statistics: Storage Size is 71453102
    
    ``````

探索 `TestBatchSizesAsync` 程序中的程式碼，以查看程式碼如何測試批次大小效能。

探索 `IndexDataAsync` 程序中的程式碼，以查看程式碼如何管理執行緒。

探索 `ExponentialBackoffAsync` 中的程式碼，以查看程式碼如何實作指數輪詢重試策略。

您可以搜尋並確認文件是否已新增至 Azure 入口網站中的索引。

![顯示搜尋索引有 100000 份文件的螢幕擷取畫面。](../media/07-media/check-search-service-index.png)

### 刪除練習資源

您已完成練習，請刪除所有不再需要的資源。 請從複製到您電腦的程式碼開始。 接著，請刪除 Azure 資源。

1. 在 **Azure 入口網站**，中選取 [資源群組]。
1. 選取您為此練習建立的資源群組。
1. 選取 [刪除資源群組]****。 
1. 確認刪除，然後選取 [刪除]****。
1. 選取您不需要的資源，然後選取 [刪除]****。
