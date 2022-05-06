---
ms.openlocfilehash: 07a00d0079982d7fe6ec4ba16e3d6d06420b0190
ms.sourcegitcommit: 3e65a56f5041a2106e99b6fe06be5dc7a931198e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "144385232"
---

# <a name="modern-data-warehouse-openhack"></a>最新のデータ ウェアハウス OpenHack

## <a name="setting-up-permissions"></a>アクセス許可の設定  

続行する前に、Azure サブスクリプションで OpenHack を実行するために必要なアクセス許可を把握しておいてください。

出席者は、リソース グループにリソースを作成できる Azure サブスクリプションのアクセス許可を持っている必要があります。 さらに、出席者には、Azure AD でサービス プリンシパルを作成し、Azure AD にアプリケーションを登録するための、サブスクリプションの十分なアクセス許可が必要です。 通常は、リソース グループに対する `Owner` ロールを持つユーザー アカウントだけで十分です。

## <a name="common-azure-resources"></a>一般的な Azure リソース

OpenHack でデプロイおよび使用される一般的な Azure リソースの一覧を次に示します。 

これらのサービスが Azure Policy によってブロックされていないことを確認してください。 これは OpenHack であるため、参加者が使用できるサービスはこのリストに限定されていません。そのため、参加者が使用したいと考えているサービスがポリシーによって無効にされている場合、厳密に制御されたサービス カタログを含むサブスクリプションでは問題が発生する可能性があります。

| Azure リソース           | リソース プロバイダー |
| ------------------------ | --------------------------------------- |
| Azure Cosmos DB          | Microsoft.DocumentDB                    | 
| Azure Data Factory       | Microsoft.DataFactory                   |
| Azure Purview            | Microsoft.Purview                       |
| Azure Synapse            | Microsoft.Synapse                       |
| Azure Databricks         | Microsoft.Databricks                    |
| Azure SQL データベース       | Microsoft.SQL                           |
| Azure Storage に関するページ            | Microsoft.Storage                       |
| Azure Data Lake Store    | Microsoft.DataLakeStore                 |
| Azure Virtual Machines   | Microsoft.Compute                       |

> 注: [リソース プロバイダーの登録] は、 https://portal.azure.com/ _<自分のテナント名>_ .onmicrosoft.com/resource/subscriptions/ _<自分のサブスクリプション ID>_ /resourceproviders で見つかります

## <a name="attendee-computers"></a>参加者のコンピューター

参加者は、OpenHack を実行しているワークステーションにソフトウェアをインストールする必要があります。 ソフトウェアのインストールを実行するための適切なアクセス許可があることを確認してください。

## <a name="deployment-instructions"></a>デプロイの手順  

1. **PowerShell 7** ウィンドウを開き、次のコマンドを実行します。メッセージが表示されたら、 **[Yes to All]\(すべてはい\)** をクリックします。

   ```PowerShell
   Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
   ```

2. 次のようにして、サブスクリプションで **所有者** ロールが割り当てられた Azure アカウントにサインインします。

    ```PowerShell
    Connect-AzAccount
    ```

3. 複数のサブスクリプションがある場合は、次のステップに進む前に、必ず適切なものを選択してください。 `Get-AzSubscription` を使ってその一覧を表示した後、以下のコマンドを使って、使用しているサブスクリプションを設定します。

    サブスクリプションを一覧表示する:  

    ```powershell
    Get-AzSubscription
    ```  

    使用するサブスクリプションを選択する:

    ```powershell
    Select-AzSubscription -Subscription <The selected Subscription Id>
    ```

4. PowerShell セッションの `$sqlpwd` 変数と `$vmpwd` 変数を **セキュリティで保護された文字列** として割り当てます。 どちらにも強力なパスワードを使用してください。 仮想マシンのパスワード要件については[このリンク](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm)に、SQL Server については[このリンク](https://docs.microsoft.com/en-us/sql/relational-databases/security/password-policy?view=sql-server-2017#password-complexity)に移動してください。

    ```powershell
    $PlainPassword = "demo@pass123"
    $SqlAdminLoginPassword = $PlainPassword | ConvertTo-SecureString -AsPlainText -Force
    $VMAdminPassword = $PlainPassword | ConvertTo-SecureString -AsPlainText -Force
    ```  

5. まだ行っていない場合は、リポジトリから `modern-data-warehousing` フォルダーをダウンロードする必要があります。  次のコマンドを使って、リポジトリを現在のディレクトリにクローンできます。

   ```shell
   git clone https://github.com/microsoft/OpenHack.git
   ```
  
6. OpenHack リポジトリのクローンの `modern-data-warehousing` ディレクトリから次を実行して、環境をデプロイします (このプロセスには 10 分から 15 分かかる場合があります)。

    ```powershell
     .\BYOS-deployAll.ps1 -SqlAdminLoginPassword $SqlAdminLoginPassword -VMAdminPassword $VMAdminPassword 
    ```

### <a name="manual-step---assigning-users-to-each-resource-group"></a>手動ステップ - 各リソース グループへのユーザーの割り当て  

デプロイ後、チームの適切なリソース グループに対する所有者アクセス権を持つ適切なユーザーを、手動で追加します。  

ユーザーをロールに割り当てる詳細な手順については、次を参照してください。

[Azure portal を使用して Azure ロールの割り当てを追加または削除する](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-portal)

## <a name="validate"></a>検証  

各チームにリソース グループが存在し、各チームのメンバーにリソース グループに対する所有者アクセス許可が適切に割り当てられています。

チームごとに、前提条件やその他のことを readme ファイルで確認します。

## <a name="more-details-on-the-usage-of-the-services"></a>サービスの使用方法の詳細

プロセスを開始すると、多数のテンプレートがデプロイされます。

### <a name="deploymdwopenhacklabjson"></a>DeployMDWOpenHackLab.json

これを開始しました (おそらく BYOS-deployAll.ps1 スクリプトから)。  

この展開テンプレートにより、Azure Storage 内にある同じ名前のオーケストレーター テンプレートがトリガーされます。  

#### <a name="parameters"></a>パラメーター

`DeployMDWOpenHackLab.parameters.json` を使うと、ほとんどのパラメーターが自動的に提供されることに注意してください。 以下では、デプロイ時に注意が必要なものが、**太字** でマークされています。

- SqlAdminLogin: このデプロイ内の **すべての** SQL DB の SQL 管理者ユーザー名。
- **SqlAdminLoginPassword**: このデプロイ内の **すべての** SQL DB での SqlAdminLogin のパスワード。
- SalesDacPacPath: CloudSales (southridge) DB にインポートされた bacpac への URI。
- StreamingDacPacPath: CloudStreaming (southridge) DB にインポートされた bacpac への URI。
- VMAdminUsername: このデプロイ内の **すべての** VM の管理者ユーザー名
- **VMAdminPassword**: このデプロイ内の **すべての** VM での管理者のパスワード
- RentalsBackupStorageAccountName: dbbackups が格納されているストレージ アカウント
- RentalsBackupFileName: "オンプレミス" の VM の Rentals DB (VanArsdel Ltd.) にインポートされた .bak のファイル名
- RentalsDatabaseName: VM に復元される "オンプレミス" のデータベースの名前
- RentalsCsvFolderName: dbbackups コンテナー内のすべての CSV データを含むフォルダーの名前
- CatalogJsonFileName: Southridge Video のすべてのムービー カタログを含む JSON ファイルの名前
- SQLFictitiousCompanyNamePrefix: "オンプレミス" の SQL VM を使用している架空の会社名 (VanArsdel Ltd.)
- CsvFictitiousCompanyNamePrefix: "オンプレミス" の CSV ファイルを使用している架空の会社名 (Fourth Coffee)
- CloudFictitiousCompanyNamePrefix: クラウドの DB を使用している架空の会社名 (Southridge)
- **location**:ターゲットの Azure リージョン

> 注: すべてのスクリプトにはデプロイの重要な部分に関する既定値が設定されているため、パラメーターを指定せずにスクリプトを実行できます。  必要な場合は (パスワードなど)、パラメーターを使ってオーバーライドできます。  

### <a name="deploycosmosdbjson"></a>DeployCosmosDB.json

Cosmos DB アカウントのプロビジョニングを対象としたテンプレート。

> DeployFileVM の VM 拡張機能によって、Cosmos DB のコレクションが作成されて設定されます。

#### <a name="parameters"></a>パラメーター  

- location:ターゲットの Azure リージョン
- namePrefix: Cosmos DB のアカウントは、`{namePrefix}-catalog-{uniqueStringForResourceGroup}` の形式に従った名前で作成されます

### <a name="deployfilevmjson"></a>DeployFileVM.json

1 つの架空の会社が CSV データを格納する VM の作成を対象とするテンプレート。

> この VM のデプロイには、CSV データを VM にダウンロードするだけでなく、Cosmos DB のムービー カタログの設定も行う拡張機能が含まれています。

#### <a name="parameters"></a>パラメーター

- adminUsername:VM 管理者のユーザー名
- adminPassword:VM 管理者のパスワード
- BackupStorageAccountName: CSV データが含まれるストレージ アカウント
- BackupStorageContainerName: ストレージ アカウント内の CSV データが含まれるコンテナー
- RentalsCsvFolderName: コンテナー内の CSV データが含まれるフォルダー
- catalogJsonFileName: Cosmos にアップロードする southridge ムービー カタログのファイル名
- location:ターゲットの Azure リージョン
- namePrefix: この名前は VM リソースで使用されます (例: `{namePrefix}VM`、`{namePrefix}-PIP` など)。

### <a name="deploysqlazurejson"></a>DeploySQLAzure.json

このテンプレートにより、2 つの Azure SQL DB が 1 つのサーバーにデプロイされ、それぞれに対して bacpac のインポートが実行されます。

#### <a name="parameters"></a>パラメーター

- AdminLogin: SQL の管理者ログイン
- AdminLoginPassword: SQL の管理者パスワード
- SalesDacPacPath: CloudSales bacpac へのパス
- StreamingDacPacPath: CloudStreaming bacpac へのパス
- DacPacContainerSAS: dbbackups コンテナーの SAS
- location:ターゲットの Azure リージョン
- namePrefix: SQL Server の名前は、`{namePrefix-sqlserver-uniqueStringForResourceGroup}` という形式になります

### <a name="deploysqlvmjson"></a>DeploySQLVM.json

これにより、SQL Server を含む VM がデプロイされて、オンプレミスのレンタル DB 用の bak がインポートされます。

#### <a name="parameters"></a>パラメーター

- adminUsername:VM 管理者のユーザー名
- adminPassword:VM 管理者のパスワード
- sqlAuthenticationLogin: SQL のユーザー名
- sqlAuthenticationPassword: SQL のパスワード
- BackupStorageAccountName: bak が含まれるストレージ アカウント
- BackupStorageContainerName: ストレージ アカウント内の bak が含まれるコンテナー
- BackupFileName: bak のファイル名
- DatabaseName: 復元される DB の名前
- location:ターゲットの Azure リージョン
- namePrefix: この名前は VM リソースで使用されます (例: `{namePrefix}VM`、`{namePrefix}-PIP` など)。
