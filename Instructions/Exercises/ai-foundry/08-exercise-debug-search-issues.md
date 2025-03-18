---
lab:
  title: 偵錯搜尋問題
---

# 偵錯搜尋問題

您建立好自己的搜尋解決方案後，發現索引子上出現一些警告。

在本練習中，您會建立 Azure AI 搜尋服務解決方案、匯入樣本資料，然後解析索引子上的警告。

> **注意**：若要完成此練習，您需要 Microsoft Azure 訂用帳戶。 如果尚未有訂用帳戶，則可在 [https://azure.com/free](https://azure.com/free?azure-portal=true) 註冊免費試用版。

## 建立自己的搜尋解決方案

開始使用偵錯工作階段之前，請先建立 Azure AI 搜尋服務。

1. [將資源部署至 Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2Fmslearn-knowledge-mining%2Fmain%2FLabfiles%2F08-debug-search%2Fazuredeploy.json) - 如果您位於託管的 VM 中，請複製此連結並貼到 VM 瀏覽器。 否則，選取此連結，將所有需要的資源部署至 Azure 入口網站。

    ![欄位已輸入的 ARM 部署範本螢幕擷取畫面。](../media/08-media/arm-template-deployment.png)

1. 在**資源群組**底下，選取您提供的資源群組，或選取**新建**並輸入 **debug-search-exercise**。
1. 選取與您最接近的**區域**，或使用預設區域。
1. 針對**資源前置詞**，輸入 **debugsearch**，並新增隨機的數字或字元組合，以確保儲存體名稱是唯一的。
1. 針對 [位置]，選取與上方所用相同的區域。
1. 在窗格底部，選取 [審核 + 建立]****。
1. 等候資源部署完成，然後選取 [前往資源群組]****。

## 匯入範例資料並設定資源

資源建立完成之後，即可匯入來源資料。

1. 在列出的資源中，瀏覽至儲存體帳戶。 移至左側窗格中的**設定**，將**允許 Blob 匿名存取**設定為**啟用**，然後選取**儲存**。
1. 瀏覽至您的資源群組，然後選取搜尋服務。
1. 在 [概觀]**** 窗格上，選取 [匯入資料]****。

      ![顯示選取匯入資料選單的螢幕擷取畫面。](../media/08-media/import-data.png)

1. 在匯入資料窗格的 [資料來源] 中，選取 [樣本]****。

      ![顯示欄位已填入的螢幕擷取畫面。](../media/08-media/import-data-selection-screen-small.png)

1. 在樣本清單中選取 [hotels-sample]****。
1. 選取 [下一步：新增認知技能 (選用)]****。
1. 展開 [新增擴充]**** 區段。

    ![新增擴充選項的螢幕擷取畫面。](../media/08-media/add-enrichments.png)

1. 選取 [文字認知技能]****。
1. 選取 [下一步：自訂目標索引]****。
1. 保留預設值，然後選取 [下一步：建立索引子]****。
1. 選取 [提交]****。

## 使用偵錯工作階段解析索引子上的警告

索引子即將開始內嵌 50 份文件。 但檢查索引子的狀態後，您卻發現有警告。

![顯示索引子上 150 則警告的螢幕擷取畫面。](../media/08-media/indexer-warnings.png)

1. 選取左窗格中的 [偵錯工作階段]****。
1. 選取 [+ 新增偵錯工作階段]****。
1. 提供工作階段的名稱，並為**索引子範本**選取 **hotel-sample-indexer**。
1. 從**儲存體帳戶**欄位中，選取您的儲存體帳戶。 這會自動建立儲存體容器，讓您保留偵錯資料。
1. 讓使用受控識別進行驗證保持未選取狀態。
1. 選取**儲存**。
1. 建立之後，偵錯工作階段會自動在搜尋服務中的資料上執行。 它應該會在完成時發生錯誤/警告。

    相依性關係圖顯示每份文件都有三項技能方面的錯誤。
    ![顯示擴充文件有三個錯誤的螢幕擷取畫面。](../media/08-media/debug-session-errors.png)

    > **注意**：您可能會看到連線到儲存體帳戶和設定受控識別的錯誤。 如果您在啟用匿名 Blob 存取之後嘗試過快偵錯，就會發生這種情況，而且執行偵錯工作階段仍可正常運作。 幾分鐘后重新整理瀏覽器視窗應該會移除警告。

1. 在相依性關係圖中，選取其中一個發生錯誤的技能節點。
1. 在技能詳細資料窗格上選取 [錯誤/警告 (1)]****。

    詳細資料如下︰

    *無效的語言代碼「（未知）」。支援的語言：af、am、ar、as、az、bg、bn、bs、ca、cs、cy、da、de、el、en、es、et、eu、fa、fi、fr、ga、gl、gu、he、hi、hr、hu、hy、id、it、ja、ka、kk、km、kn、ko、ku、ky、lo、lt、lv、mg、mk、ml、mn、mr、ms、my、ne、nl、no、or、pa、pl、ps、pt-BR、pt-PT、ro、ru、sk、sl、so、sq、sr、ss、sv、sw、ta、te、th、tr、ug、uk、ur、uz、vi、zh-Hans、zh-Hant。如需其他詳細資訊，請參閱 https://aka.ms/language-service/language-support。*

    請回顧相依性關係圖，語言偵測技能輸出至三項技能時顯示錯誤。 如果您查看有錯誤的技能設定，您會看到導致錯誤的技能輸入為 `languageCode`。

1. 在相依性關係圖中選取 [語言偵測]****。

    ![顯示語言偵測技能之技能設定的螢幕擷取畫面。](../media/08-media/language-detection-skill-settings.png)
    請查看技能設定 JSON，您會發現用來推算語言的欄位為 `HotelId`。

    由於技能無法解析使用識別碼的語言，因此本欄位造成了錯誤。

## 解析索引子上的警告

1. 選取 [輸入] 底下的 [來源]****，並將欄位變更為 `/document/Description`。
1. 選取**儲存**。
1. 選取**執行**。 索引子上應該不會再顯示任何錯誤或警告。 現在請更新技能。

    ![顯示完整執行且沒有錯誤的螢幕擷取畫面。](../media/08-media/debug-session-complete.png)
   
1. 選取**認可變更**，將此工作階段中所做的變更推送至您的索引子。
1. 選取 [確定]。 您現在可以刪除您的工作階段。

現在您需要確定您的技能集已連結至 Azure AI 服務資源，否則您會達到基本配額且索引子會逾時。 

1. 若要執行此操作，請在左窗格中選取 [技能集]****，然後選取您的 [hotels-sample-skillset]****。

    ![顯示技能集清單的螢幕擷取畫面。](../media/08-media/update-skillset.png)
1. 選取 **[連線 AI 服務]**，然後選取清單中的 AI 服務資源。

    ![顯示要連結至技能集的 Azure AI 服務資源螢幕擷取畫面。](../media/08-media/skillset-attach-service.png)
1. 選取 [儲存]。

1. 現在請執行索引子，以固定 AI 擴充來更新文件。 若要執行此操作，請在左窗格中選取 [索引子]****，選取 [hotels-sample-indexer]****，然後選取 [執行]****。  完成執行後，您應該不會再看到任何警告。

    ![顯示問題已解決的螢幕擷取畫面。](../media/08-media/warnings-fixed-indexer.png)

## 清理

 完成練習之後，如果您已完成探索 Azure AI 搜尋服務，請刪除您在練習中建立的 Azure 資源。 最簡單的方式是刪除 **debug-search-exercise** 資源群組。
