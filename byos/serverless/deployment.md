# <a name="serverless-openhack"></a>サーバーレス OpenHack

## <a name="setting-up-permissions"></a>アクセス許可の設定

続行する前に、Azure サブスクリプションで OpenHack を実行するために必要なアクセス許可を把握しておいてください。

### <a name="initial-setup"></a>**初期セットアップ**

OpenHack の準備をするためにセットアップとデプロイを実行するには、Azure サブスクリプションの所有者ロールに割り当てられている必要があります。

これを検証するには、[Azure portal](https://portal.azure.com) に移動します。 **[すべてのサービス]**  ->  **[サブスクリプション]**  ->  **[アクセス制御 (IAM)]** をクリックします。

**[アクセスの確認]** テキスト ボックスにメール アドレスを入力して、セットアップを実行しているユーザーの現在のアクセス許可を確認します。  

![[アクセスの確認] ダイアログ](images/check-access.png "[アクセスの確認] ダイアログには、メール アドレスを入力するためのテキスト ボックスが表示されます。")

### <a name="performing-the-openhack"></a>**OpenHack の実行**

OpenHack の各参加者には、そのチームに固有のリソース グループに対する **所有者** ロールが割り当てられます。 これについては、このドキュメントのデプロイに関するセクションで後ほど説明します。

## <a name="common-azure-resources"></a>一般的な Azure リソース

OpenHack でデプロイおよび使用される一般的な Azure リソースの一覧を次に示します。  

これらのサービスが Azure Policy によってブロックされていないことを確認してください。  これは OpenHack であるため、参加者が使用できるサービスはこのリストに限定されていません。そのため、参加者が使用したいと考えているサービスがポリシーによって無効にされている場合、厳密に制御されたサービス カタログを含むサブスクリプションでは問題が発生する可能性があります。  

| Azure リソース           | リソース プロバイダー |
| ------------------------ | --------------------------------------- |  
| Azure Logic Apps         | Microsoft.Logic                         |  
| Azure Functions          | Microsoft.Web                           |  
| Azure App Service        | Microsoft.Web                           |  
| Azure Storage に関するページ            | Microsoft.Storage                       |  
| Azure Cosmos DB          | Microsoft.DocumentDb                    |  
| イベント ハブ                | Microsoft.EventHub                      |  
| Event Grid               | Microsoft.EventGrid                     |  
| Azure Cognitive Services | Microsoft.CognitiveServices             |  
| Virtual Network          | Microsoft.Network                       |  
| Azure Stream Analytics   | Microsoft.StreamAnalytics               |  
| 仮想マシン          | Microsoft.VMWare                        |  
| Azure DevOps             | Microsoft.VSOnline                      |  
| Service Bus              | Microsoft.ServiceBus                    |  

> 注: リソース プロバイダーの登録は `https://portal.azure.com/_yourtenantname_.onmicrosoft.com/resource/subscriptions/_yoursubscriptionid_/resourceproviders` にあります

## <a name="attendee-computers"></a>参加者のコンピューター

参加者は、OpenHack を実行しているワークステーションにソフトウェアをインストールする必要があります。 ソフトウェアのインストールを実行するための適切なアクセス許可があることを確認してください。  

## <a name="deployment-instructions"></a>デプロイの手順  

デプロイの場合は、各チームに適切なリソース グループを設定するための ARM テンプレートを実行する PowerShell スクリプトを実行します。  次に、手動でチーム メンバーをリソース グループに所有者として追加します。 このスクリプトは、**PowerShell 7 以降** または **PowerShell ISE** (いずれか好きな方) で実行できます。

### <a name="powershell"></a>PowerShell  

1. **PowerShell 7** ウィンドウを開き、次のコマンドを実行します。メッセージが表示されたら、 **[Yes to All]\(すべてはい\)** をクリックします。

   ```PowerShell
   Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
   ```

2. 次のコマンドを実行して、最新の PowerShell Azure モジュールがインストールされていることを確認します。

    ```PowerShell
    Install-Module -Name Az -AllowClobber -Scope CurrentUser
    ```

3. 更新プログラムをインストールした場合は、PowerShell 7 ウィンドウを **閉じて** から、**再度開きます**。 これにより、**AZ** モジュールの最新バージョンが使用されるようになります。

4. 次を実行して、Azure アカウントにサインインします。

    ```PowerShell
    Connect-AzAccount
    ```  

5. 複数のサブスクリプションがある場合は、次の手順の前に、必ず適切なサブスクリプションを選択します。 `Get-AzSubscription` を使用してサブスクリプションを一覧表示した後、以下に示すように `Select-AzSubscription` コマンドを使用して、デプロイの対象となるサブスクリプションを設定します。

    サブスクリプションを一覧表示する:  

    ```powershell
    Get-AzSubscription
    ```  

    使用するサブスクリプションを選択する:

    ```powershell
    Select-AzSubscription -Subscription <The selected Subscription Id>
    ```  

6. ディレクトリを `serverless\deploy\` フォルダーに変更します。  必要に応じて、スクリプトを確認および変更します。  

    **省略可能**: チーム数またはリージョンの読み取りをオーバーライドするようにスクリプトを変更します。

    ```PowerShell
    $teamCount = Read-Host "How many teams are hacking?";
    #could be
    $teamCount = 1;
    #and 
    $region = Read-Host "What Region Resources be deployed to (i.e. centralus, southcentralus, japaneast, etc)?";
    #could be
    $region = "centralus"
    ```  

7. スクリプトを実行します。

    次のコマンドを使用します。

    ```powershell  
    .\deployAll.ps1
    ```  

    スクリプトを実行するには  

    > *注: `azuredeploy.json` が見つからないというエラーが表示された場合は、**PowerShell** のコンソール ウィンドウの現在のディレクトリが `serverless\deploy` フォルダーであることを確認してください。*  

   * **ServerlessOpenHackRGXX-[location]** というリソース グループを作成します。ここでは、XX は 2 桁のチーム番号、location はスクリプト内で入力または指定した場所になります。  たとえば、**southcentralus** のチーム2 の場合、予想されるリソース グループは `ServerlessOpenHackRG02-southcentralus` です  
   * それぞれのコンテナーがある 2 つのストレージ アカウント [sohsalesxxxxxxxxx] と [sohvmdiagxxxxxxxxx]
   * VPN [soh-vnet]
   * ネットワーク インターフェイス [soh-jumpbox-nic]
   * ネットワーク セキュリティ グループ [soh-jumpbox-nsg]
   * パブリック ID [soh-jumpbox-pip]
   * VM ディスク [soh-jumpbox_OsDisk_1_xxxxxxxxxxxxx]
   * VM [soh-jumpbox]

### <a name="manual-step"></a>手動操作

デプロイ後、チームの適切なリソース グループに、所有者アクセス権を持つ適切なユーザーを手動で追加すると、ユーザーは、そのリソース グループにリソースを作成してデプロイできるようになります。

## <a name="deployment-artifacts--validation"></a>デプロイ成果物または検証

デプロイが完了すると、各チームのリソース グループに次のリソースが表示されます。###

* チーム XX リソース グループ (つまり `ServerlessOpenHackRG01-centralus`)
* VM - [soh-jumpbox]
* NIC - [soh-jumpbox-nic]
* NSG - [soh-jumpbox-nsg]
* パブリック ID - [soh-jumpbox-pip]
* ディスク - [soh-jumpbox_OsDisk_1_xxxxxxxx]
* VPN - [soh-vnet]
* Storage Sales [sohsalesxxxxxxxxxx]  
    - コンテナー [receipts]  
    - コンテナー [receipts-high-value]  
* Storage VMDiagnostics [sohvmdiagxxxxxxxxx]  
    - コンテナー [bootdiagnostics-sohjumpbo-(guidish)]

## <a name="more-detail-on-the-usage-of-the-services"></a>サービスの使用方法の詳細  

チームの進捗に応じて:  

* チームがストレージを作成して操作できることを確認する
* チームがロジック アプリを作成できることを確認する
* チームが Standard または Free の関数アプリ サービスを使用して関数を作成し、最終的に Premium に更新できるかどうかを確認する
* チームが APIM を作成できることを確認する
* チームがイベント ハブを作成し、トリガーを Event Grid と統合できることを確認する
* チームがサービス バスを作成し、適切なメッセージングに使用できることを確認する
* チームが Stream Analytics を使用して情報をフィルター処理できることを確認する
* チームが Cognitive Services をプロビジョニングできることを確認する

## <a name="services-and-application-overview"></a>サービスとアプリケーションの概要  

この OpenHack では、次のサービスとアプリケーションが使用されます。

### <a name="logic-apps"></a>Logic Apps  

メモ:  

* ユーザーは少なくとも 2 つ (おそらく 4 つまたは 5 つ) を作成します  
* イベントへの応答と、サード パーティの Outlook またはビジネス サーバーに対してメールをプッシュする処理  
* 関数またはその他の応答イベントにメッセージをルーティングする  
* レポート用のデータを収集する、など  

### <a name="cosmos-db-andor-azure-tables"></a>Cosmos DB または Azure テーブル  

メモ:  

* Cosmos DB は後の課題で非常に容易になるため、Cosmos DB の使用をお勧めします
* テーブルを作成し、Cosmos DB に対するイベントに応答できるようになる必要があります

### <a name="apim"></a>APIM

メモ:  

* 参加者は API Management ゲートウェイを作成して内部 API と外部 API をグループ化します  
* APIM を使用して、アクセスのためにさまざまなサブスクリプションを作成します

### <a name="azure-functions"></a>Azure Functions

メモ:

* 複数の関数が作成またはデプロイされます
* http、イベント グリッド、またはストレージがトリガーされると、最終的に VPN エンドポイントに要求をプッシュできる必要があります

### <a name="app-service"></a>App Service

メモ:  

* 関数アプリで Azure Functions を活用します
* Basic または Free プランで開始します
* 場合によっては、ネットワーク統合でトラフィックのルーティングまたは機能のロックダウンを行うために Premium プランが必要になります

### <a name="cognitive-services"></a>Cognitive Services  

メモ:

* メッセージまたはレビューの意図を解析するための Cognitive Services  
* Text Analytics API を使用した、イベントやキューとの統合  

### <a name="messaging-and-events"></a>メッセージングとイベント

メモ:  

* メッセージ処理用の Event Grid またはイベント ハブ  
* メッセージ キュー用の Service Bus  

### <a name="storage"></a>ストレージ  

メモ:

* サード パーティ製アプリによってストレージにプッシュされる多数の csv ファイルを受け取るコンテナーが備わっているパブリック BLOB
* ロジックアプリまたは Azure 関数を介したストレージ作成イベントに応答する機能

### <a name="iaas"></a>IaaS  

メモ:  

* ネットワーク  
* チームは、サードパーティ VPN にプッシュするものを構築する必要があります

* VM
* チームがサードパーティのサービスから VPN エンドポイントを表示するためにジャンプボックス VM が作成されます。

### <a name="stream-analytics"></a>Stream Analytics  

メモ:  

* レポート用のデータを収集するために使用されます  

### <a name="deploymentpipelinessource-repository"></a>デプロイ、Pipelines、またはソース リポジトリ  

チームは Azure DevOps または GitHub で CI/CD パイプラインを設定する必要があります

### <a name="development-tools"></a>開発ツール  

作業を完了するには、次のうち 1 つ以上を使用する必要があります

* Visual Studio - チームは Visual Studio でコードを記述できる必要があります
* Java または Maven  
* VS Code または Python と Node またはその他の開発
* postman
