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

1. [將資源部署至 Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2Fmslearn-knowledge-mining%2Fmain%2FLabfiles%2F07-exercise-add-to-index-use-push-api%2Fazuredeploy.json) - 選取此連結以建立您的 Azure AI 資源。
    ![部署資源至 Azure 時，顯示選項的螢幕擷取畫面。](../media/07-media/deploy-azure-resources.png)
1. 在 [資源群組] 中，選取 [新建]，將群組命名為 **cog-search-language-exe**。********
1. 在 [區域]**** 中，選取靠近您的[支援區域](/azure/ai-services/language-service/custom-text-classification/service-limits#regional-availability)。
1. **資源前置詞**必須是全域唯一的，請輸入隨機數字和小寫字元前置詞，例如 **acs118245**。
1. 在 [位置]**** 中，選取您在上述步驟選擇的相同區域。
1. 選取 [**檢閱 + 建立**]。
1. 選取 **建立**。
1. 當部署完成時，選取 [移至資源群組]**** 以查看您建立的所有資源。

    ![顯示所有已部署 Azure 資源的螢幕擷取畫面。](../media/07-media/azure-resources-created.png)

## 複製 Azure AI 搜尋服務 REST API 資訊

1. 在資源清單中，選取您建立的搜尋服務。 在上述範例中，即 **acs118245-search-service**。
1. 將搜尋服務名稱複製到文字檔中。

    ![搜尋服務金鑰區段的螢幕擷取畫面。](../media/07-media/search-api-keys-exercise-version.png)
1. 在左側，選取 [金鑰]****，然後將**主要系統管理金鑰**複製到相同的文字檔中。

## 請在 Cloud Shell 中複製存放庫

您將可使用 Azure 入口網站中的 Cloud Shell，開發程式碼。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

> **提示**：如果您最近已複製 **mslearn-knowledge-mining** 存放庫，可以跳過這項工作。 否則，請遵循下列步驟將其複製到您的開發環境。

1. 使用頁面上方搜尋欄右側的 [\>_]**** 按鈕，即可到 Azure 入口網站上，建立新的 Cloud Shell，選取 [PowerShell]****** 環境。 Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    > **提示**：當您將命令貼到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 在 PowerShell 窗格中，輸入以下命令來複製此練習的 GitHub 存放庫：

    ```
    rm -r mslearn-knowledge-mining -f
    git clone https://github.com/microsoftlearning/mslearn-knowledge-mining mslearn-knowledge-mining
    ```

1. 複製存放庫之後，瀏覽至包含應用程式碼檔案的資料夾：  

    ```
   cd mslearn-knowledge-mining/Labfiles/07-exercise-add-to-index-use-push-api/OptimizeDataIndexing
    ```

## 設定您的應用程式

1. 您只要使用`ls`命令，就可以檢視 OptimizeDataIndexing** 資料夾的內容**。 請注意，其中還包含`appsettings.json`組態設定的檔案。

1. 輸入以下命令，編輯已提供的設定檔：

    ```
   code appsettings.json
    ```

    程式碼編輯器中會開啟檔案。

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
   
1. 取代預留位置後，使用 **CTRL+S** 命令儲存變更，然後使用 **CTRL+Q** 命令關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。
1. 在終端中，輸入 `dotnet run`，然後按 **Enter**。

    ![顯示應用程式在 VS Code 中執行但出現例外狀況的螢幕擷取畫面。](../media/07-media/debug-application.png)

    輸出內容會顯示以下案例中，最佳效能的批次大小大約是 900 份文件，最高傳輸速率為（MB/秒）。
   
    >**備註**：傳輸速率的數值可能和螢幕擷取畫面中顯示的數值不太一樣。 然而，表現最佳的批量大小應該都差不多。 

## 編輯程式碼以實作執行緒和輪詢並重試策略

程式碼已加上註解，可讓應用程式變更為使用執行緒將文件上傳至搜尋索引。

1. 輸入以下命令，即可開啟用戶端應用程式的程式碼檔案：

    ```
   code Program.cs
    ```

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

1. 儲存您的變更。
1. 選取您的終端，然後按下任何按鍵以結束執行中的程序 (如果您尚未結束)。
1. 在終端中執行 `dotnet run`。

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

## 清理

您已完成練習，請刪除所有不再需要的資源。 請從複製到您電腦的程式碼開始。 接著，請刪除 Azure 資源。

1. 在 **Azure 入口網站**，中選取 [資源群組]。
1. 選取您為此練習建立的資源群組。
1. 選取 [刪除資源群組]****。 
1. 確認刪除，然後選取 [刪除]****。
1. 選取您不需要的資源，然後選取 [刪除]****。
