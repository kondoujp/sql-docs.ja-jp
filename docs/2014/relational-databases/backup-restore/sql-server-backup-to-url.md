---
title: SQL Server Backup to URL | Microsoft Docs
ms.custom: ''
ms.date: 01/25/2016
ms.prod: sql-server-2014
ms.reviewer: ''
ms.technology: backup-restore
ms.topic: conceptual
ms.assetid: 11be89e9-ff2a-4a94-ab5d-27d8edf9167d
author: MikeRayMSFT
ms.author: mikeray
manager: craigg
ms.openlocfilehash: c1ecaf46ebf96ab5b8d06cb5eefb69ae50ff882e
ms.sourcegitcommit: 3b1f873f02af8f4e89facc7b25f8993f535061c9
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/30/2019
ms.locfileid: "70175840"
---
# <a name="sql-server-backup-to-url"></a>SQL Server Backup to URL
  このトピックでは、Azure Blob ストレージサービスをバックアップ先として使用するために必要な概念、要件、およびコンポーネントについて説明します。 バックアップと復元の機能は、ディスクまたはテープを使用する場合とよく似ていますが、いくつか相違点もあります。 相違点、重要な例外、コード例についても、このトピックで説明しています。  
  
## <a name="requirements-components-and-concepts"></a>要件、コンポーネント、および概念  
 **このセクションの内容:**  
  
-   [セキュリティ](#security)  
  
-   [主なコンポーネントと概念の概要](#intorkeyconcepts)  
  
-   [Azure Blob Storage サービス](#Blob)  
  
-   [SQL Server のコンポーネント](#sqlserver)  
  
-   [制限事項](#limitations)  
  
-   [BACKUP/RESTORE ステートメントのサポート](#Support)  
  
-   [SQL Server Management Studio でのバックアップ タスクの使用](sql-server-backup-to-url.md#BackupTaskSSMS)  
  
-   [メンテナンス プラン ウィザードを使用した SQL Server Backup to URL](sql-server-backup-to-url.md#MaintenanceWiz)  
  
-   [SQL Server Management Studio を使用した Azure storage からの復元](sql-server-backup-to-url.md#RestoreSSMS)  
  
###  <a name="security"></a> セキュリティ  
 次に、Azure Blob ストレージサービスとの間でバックアップまたは復元を行う際のセキュリティに関する考慮事項と要件を示します。  
  
-   Azure Blob ストレージサービスのコンテナーを作成するときは、アクセス権を**private**に設定することをお勧めします。 アクセスを private に設定すると、Azure アカウントに対する認証に必要な情報を提供できるユーザーまたはアカウントへのアクセスが制限されます。  
  
    > [!IMPORTANT]  
    >  [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)]Azure アカウント名とアクセスキー認証が[!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)]資格情報に格納されている必要があります。 この情報は、バックアップ操作または復元操作の実行時に Azure アカウントに対して認証を行うために使用されます。  
  
-   BACKUP コマンドまたは RESTORE コマンドの発行に使用するユーザー アカウントは、 **資格情報の変更** 権限を持つ **db_backup operator** データベース ロールに属している必要があります。  
  
###  <a name="intorkeyconcepts"></a> 主なコンポーネントと概念の概要  
 次の2つのセクションでは、azure blob ストレージサービス[!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)]と、azure blob ストレージサービスとの間でバックアップまたは復元を行うときに使用するコンポーネントについて説明します。 Azure Blob ストレージサービスとの間でバックアップまたは復元を実行するには、コンポーネントと、コンポーネント間のやり取りを理解しておくことが重要です。  
  
 このプロセスの最初の手順として、Azure アカウントを作成します。 [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)]は、 **Azure ストレージアカウント名**とその**アクセスキー**値を使用して、ストレージサービスに対して認証および blob の読み書きを行います。 [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] 資格情報はこの認証情報を格納するため、バックアップ操作または復元操作中に使用されます。 ストレージアカウントを作成し、単純な復元を実行する完全なチュートリアルについては、「 [Azure Storage サービスを使用した SQL Server のバックアップと復元のチュートリアル](https://go.microsoft.com/fwlink/?LinkId=271615)」を参照してください。  
  
 ![ストレージアカウントを sql 資格情報にマップしています](../../tutorials/media/backuptocloud-storage-credential-mapping.gif "ストレージアカウントを sql 資格情報にマップしています")  
  
###  <a name="Blob"></a>Azure Blob Storage サービス  
 **ストレージ アカウント:** ストレージ アカウントは、すべてのストレージ サービスの開始点となります。 Azure Blob Storage サービスにアクセスするには、まず Azure ストレージアカウントを作成します。 **ストレージアカウント名**とその**アクセスキー**プロパティは、Azure Blob Storage サービスとそのコンポーネントに対して認証を行うために必要です。  
  
 **コンテナー:** コンテナーは一連の Blob をグループ化したもので、無制限の数の Blob を格納できます。 Azure Blob service に[!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)]バックアップを書き込むには、少なくともルートコンテナーが作成されている必要があります。  
  
 **BLOB:** 任意の種類とサイズのファイルです。 Azure Blob ストレージサービスに格納できる blob には、ブロック blob とページ blob の2種類があります。 [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] バックアップでは、BLOB の種類としてページ BLOB を使用します。 Blob は、次の URL 形式を使用し\<てアドレス指定できます: https://ストレージアカウント >、blob\<. core.\<windows. net/container >/blob >  
  
 ![Azure BLOB ストレージ](../../database-engine/media/backuptocloud-blobarchitecture.gif "Azure BLOB ストレージ")  
  
 Azure Blob ストレージサービスの詳細については、「 [クイック スタート:.NET 用 Azure Blob Storage クライアント ライブラリ](http://www.windowsazure.com/develop/net/how-to-guides/blob-storage/) 」を参照してください。  
  
 ページ BLOB の詳細については、「 [ブロック BLOB およびページ BLOB について](https://msdn.microsoft.com/library/windowsazure/ee691964.aspx)」を参照してください。  
  
###  <a name="sqlserver"></a> [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] コンポーネント  
 **URL:** 一意なバックアップ ファイルの Uniform Resource Identifier (URI) を示します。 URL は、 [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] バックアップ ファイルの場所と名前を指定するために使用されます。 この実装では、有効な URL は、Azure ストレージアカウント内のページ Blob を指す URL のみです。 URL は、コンテナーだけでなく、実際の BLOB を指している必要があります。 BLOB が存在しない場合は作成されます。 既存の Blob が指定されている場合、"WITH FORMAT" オプションが指定されていない限り、BACKUP は失敗します。  
  
> [!WARNING]  
>  バックアップファイルをコピーして Azure Blob ストレージサービスにアップロードする場合は、ストレージオプションとしてページ Blob を使用します。 ブロック BLOB からの復元はサポートされていません。 ブロック BLOB から RESTORE ステートメントを実行すると、エラーが発生して失敗します。  
  
 URL 値の例を次に示します。 http [s\<]://ACCOUNTNAME.Blob.core.windows.net/\<CONTAINER >/FILENAME .bak >。 HTTPS は必須ではありませんが、推奨されています。  
  
 **資格情報:** [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] 資格情報は、SQL Server の外部にあるリソースへの接続に必要な認証情報を保存するために使用されるオブジェクトです。  ここで[!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)]は、バックアップと復元のプロセスで、Azure Blob ストレージサービスに対する認証に資格情報を使用します。 資格情報には、ストレージ アカウントの名前とその **アクセス キー** 値が格納されます。 作成した資格情報は、BACKUP/RESTORE ステートメントの実行時に WITH CREDENTIAL オプションで指定する必要があります。 ストレージアカウントの**アクセスキー**を表示、コピー、または再生成する方法の詳細については、「[ストレージアカウントのアクセスキー](https://msdn.microsoft.com/library/windowsazure/hh531566.aspx)」を参照してください。  
  
 資格情報を[!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)]作成する方法の詳細な手順については、このトピックの「[資格情報の作成の](#credential)例」を参照してください。  
  
 資格情報の全般的な情報については、「 [資格情報](../security/authentication-access/credentials-database-engine.md)」をご覧ください。  
  
 資格情報が使用されるその他の例については、「 [SQL Server エージェント プロキシの作成](../../ssms/agent/create-a-sql-server-agent-proxy.md)」参照してください。  
  
###  <a name="limitations"></a> 制限事項  
  
-   Premium Storage へのバックアップはサポートされていません。  
  
-   サポートされるバックアップの最大サイズは 1 TB です。  
  
-   TSQL、SMO、または PowerShell コマンドレットを使用してバックアップ ステートメントや復元ステートメントを実行できます。 SQL Server Management Studio のバックアップまたは復元ウィザードを使用して Azure Blob ストレージサービスとの間でバックアップまたは復元を行うことは、現在有効になっていません。  
  
-   論理デバイス名の作成はサポートされていません。 そのため、sp_dumpdevice または SQL Server Management Studio を使用してバックアップ デバイスとして URL を追加することはできません。  
  
-   既存のバックアップ BLOB への追加はサポートされていません。 既存の BLOB へのバックアップは WITH FORMAT オプションを使用した場合にのみ上書きできます。  
  
-   1 回のバックアップ操作で複数の BLOB にバックアップすることはサポートされていません。 たとえば、次のコードではエラーが返されます。  
  
    ```  
    BACKUP DATABASE AdventureWorks2012   
    TO URL = 'https://mystorageaccount.blob.core.windows.net/mycontainer/AdventureWorks2012_1.bak'   
       URL = 'https://mystorageaccount.blob.core.windows.net/mycontainer/AdventureWorks2012_2.bak'   
          WITH CREDENTIAL = 'mycredential'   
         ,STATS = 5;  
    GO  
  
    ```  
  
-   `BACKUP` ステートメントでブロック サイズを指定することはサポートされていません。  
  
-   `MAXTRANSFERSIZE` を指定することはサポートされていません。  
  
-   バックアップセットのオプション (`RETAINDAYS` および `EXPIREDATE`) を指定することはサポートされていません。  
  
-   [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] では、バックアップ デバイス名に最大 259 文字の制限があります。 BACKUP TO URL では、URL の指定に使用する必須要素に 'https://.blob.core.windows.net//.bak ' の 36 文字が使用されるため、アカウント、コンテナー、および BLOB の名前は残りの 223 文字で構成します。  
  
###  <a name="Support"></a> BACKUP/RESTORE ステートメントのサポート  
  
|||||  
|-|-|-|-|  
|BACKUP/RESTORE ステートメント|Supported|例外|コメント|  
|BACKUP|???|BLOCKSIZE および MAXTRANSFERSIZE はサポートされていません。|WITH CREDENTIAL を指定する必要があります|  
|RESTORE|???||WITH CREDENTIAL を指定する必要があります|  
|RESTORE FILELISTONLY|???||WITH CREDENTIAL を指定する必要があります|  
|RESTORE HEADERONLY|???||WITH CREDENTIAL を指定する必要があります|  
|RESTORE LABELONLY|???||WITH CREDENTIAL を指定する必要があります|  
|RESTORE VERIFYONLY|???||WITH CREDENTIAL を指定する必要があります|  
|RESTORE REWINDONLY|???|||  
  
 BACKUP ステートメントの構文と一般的な情報については、「[BACKUP &#40;Transact-SQL&#41;](/sql/t-sql/statements/backup-transact-sql)」をご覧ください。  
  
 RESTORE ステートメントの構文と一般的な情報については、「[RESTORE &#40;Transact-SQL&#41;](/sql/t-sql/statements/restore-statements-transact-sql)」をご覧ください。  
  
### <a name="support-for-backup-arguments"></a>BACKUP の引数のサポート  
  
|||||  
|-|-|-|-|  
|引数|Supported|例外|コメント|  
|DATABASE|???|||  
|LOG|???|||  
||  
|TO (URL)|???|ディスクまたはテープの場合とは異なり、URL では論理名の指定と作成はサポートされていません。|この引数は、バックアップ ファイルの URL パスの指定に使用されます。|  
|MIRROR TO|???|||  
|**WITH オプション:**||||  
|CREDENTIAL|???||WITH CREDENTIAL は、BACKUP TO URL オプションを使用して Azure Blob ストレージサービスにバックアップする場合にのみサポートされます。|  
|DIFFERENTIAL|???|||  
|COPY_ONLY|???|||  
|COMPRESSION&#124;NO_COMPRESSION|???|||  
|DESCRIPTION|???|||  
|NAME|???|||  
|EXPIREDATE &#124; RETAINDAYS|???|||  
|NOINIT &#124; INIT|???||このオプションは使用しても無視されます。<br /><br /> BLOB に追加することはできません。 バックアップを上書きするには、FORMAT 引数を使用します。|  
|NOSKIP &#124; SKIP|???|||  
|NOFORMAT &#124; FORMAT|???||このオプションは使用しても無視されます。<br /><br /> WITH FORMAT を指定した場合を除き、既存の BLOB に対して実行されるバックアップは失敗します。 WITH FORMAT を指定すると、既存の BLOB が上書きされます。|  
|MEDIADESCRIPTION|???|||  
|MEDIANAME|???|||  
|BLOCKSIZE|???|||  
|BUFFERCOUNT|???|||  
|MAXTRANSFERSIZE|???|||  
|NO_CHECKSUM &#124; CHECKSUM|???|||  
|STOP_ON_ERROR &#124; CONTINUE_AFTER_ERROR|???|||  
|STATS|???|||  
|REWIND &#124; NOREWIND|???|||  
|UNLOAD &#124; NOUNLOAD|???|||  
|NORECOVERY &#124; STANDBY|???|||  
|NO_TRUNCATE|???|||  
  
 BACKUP の引数の詳細については、「[BACKUP &#40;Transact-SQL&#41;](/sql/t-sql/statements/backup-transact-sql)」をご覧ください。  
  
### <a name="support-for-restore-arguments"></a>RESTORE の引数のサポート  
  
|||||  
|-|-|-|-|  
|引数|Supported|例外|コメント|  
|DATABASE|???|||  
|LOG|???|||  
|FROM (URL)|???||FROM URL 引数は、バックアップ ファイルの URL パスの指定に使用されます。|  
|**WITH Options:**||||  
|CREDENTIAL|???||WITH CREDENTIAL は、RESTORE FROM URL オプションを使用して Azure Blob Storage サービスから復元する場合にのみサポートされます。|  
|PARTIAL|???|||  
|RECOVERY &#124; NORECOVERY &#124; STANDBY|???|||  
|LOADHISTORY|???|||  
|MOVE|???|||  
|[REPLACE]|???|||  
|RESTART|???|||  
|RESTRICTED_USER|???|||  
|FILE|???|||  
|PASSWORD|???|||  
|MEDIANAME|???|||  
|MEDIAPASSWORD|???|||  
|BLOCKSIZE|???|||  
|BUFFERCOUNT|???|||  
|MAXTRANSFERSIZE|???|||  
|CHECKSUM &#124; NO_CHECKSUM|???|||  
|STOP_ON_ERROR &#124; CONTINUE_AFTER_ERROR|???|||  
|FILESTREAM|???|||  
|STATS|???|||  
|REWIND &#124; NOREWIND|???|||  
|UNLOAD &#124; NOUNLOAD|???|||  
|KEEP_REPLICATION|???|||  
|KEEP_CDC|???|||  
|ENABLE_BROKER &#124; ERROR_BROKER_CONVERSATIONS &#124; NEW_BROKER|???|||  
|STOPAT &#124; STOPATMARK &#124; STOPBEFOREMARK|???|||  
  
 RESTORE の引数の詳細については、「[RESTORE の引数 &#40;Transact-SQL&#41;](/sql/t-sql/statements/restore-statements-arguments-transact-sql)」をご覧ください。  
  
##  <a name="BackupTaskSSMS"></a>SQL Server Management Studio でのバックアップタスクの使用  
 SQL Server Management Studio のバックアップタスクが拡張され、ターゲットオプションの1つとして URL が含められるようになりました。また、SQL 資格情報などの Azure storage へのバックアップに必要なその他のサポートオブジェクトも追加されました。  
  
 次の手順では、Azure storage へのバックアップを許可するためにデータベースのバックアップタスクに加えられた変更について説明します。  
  
1.  SQL Server Management Studio を起動し、SQL Server インスタンスに接続します。  バックアップするデータベースを選択し、**タスク** を右クリックして、**バックアップ** を選択します。データベースのバックアップ ダイアログボックスが開きます。  
  
2.  全般 ページの  **URL** オプションを使用して、Azure storage へのバックアップを作成します。 このオプションを選択すると、このページの他のオプションも有効になります。  
  
    1.  **ファイル名:** バックアップ ファイルの名前。  
  
    2.  **SQL 資格情報:** 既存の SQL Server 資格情報を指定することも、SQL 資格情報 ボックスの横にある **作成** をクリックして新しい資格情報を作成することもできます。  
  
        > [!IMPORTANT]  
        >  **[作成]** をクリックすると開くダイアログでは、サブスクリプションの管理証明書または公開プロファイルが求められます。 SQL Server では現在、公開プロファイルのバージョン 2.0 がサポートされています。 公開プロファイルのサポート対象バージョンをダウンロードするには、「 [公開プロファイルのバージョン 2.0 のダウンロード](https://go.microsoft.com/fwlink/?LinkId=396421)」をご覧ください。  
        >   
        >  管理証明書または公開プロファイルにアクセスできない場合は、Transact-SQL または SQL Server Management Studio を使用してストレージ アカウント名とアクセス キーの情報を指定し、SQL 資格情報を作成することができます。 Transact-sql を使用して資格情報を作成するには、「[資格情報の作成](#credential)」セクションのサンプルコードを参照してください。 または SQL Server Management Studio を使用して、データベース エンジン インスタンスから、 **[セキュリティ]** を右クリックし、 **[新規作成]** 、 **[資格情報]** の順にクリックします。 **[ID]** にストレージ アカウント名、 **[パスワード]** にアクセス キーを指定します。  
  
    3.  **[Azure Storage コンテナー]:** バックアップファイルを格納する Azure storage コンテナーの名前。  
  
    4.  **[URL プレフィックス]** これは、前の手順で説明したフィールドで指定された情報を使用して自動的に作成されます。 この値を手動で編集する場合は、前に入力した他の情報と一致することを確認してください。 たとえばストレージの URL を変更する場合は、[SQL 資格情報] も、同じストレージ アカウントに対する認証を行うように設定されていることを確認します。  
  
 バックアップ先として URL を選択すると、 **[メディア オプション]** ページの一部のオプションが無効になります。  [データベースのバックアップ] ダイアログの詳細については、次のトピックを参照してください。  
  
 [データベースのバックアップ &#40;全般ページ&#41;](../../integration-services/general-page-of-integration-services-designers-options.md)  
  
 [データベースのバックアップ &#40;メディア オプション ページ&#41;](back-up-database-media-options-page.md)  
  
 [データベースのバックアップ &#40;バックアップ オプション ページ&#41;](back-up-database-backup-options-page.md)  
  
 [資格情報の作成- Azure ストレージに対する認証](create-credential-authenticate-to-azure-storage.md)  
  
##  <a name="MaintenanceWiz"></a> メンテナンス プラン ウィザードを使用した SQL Server Backup to URL  
 前に説明したバックアップタスクと同様に、SQL Server Management Studio のメンテナンスプランウィザードが拡張され、ターゲットオプションの1つとして**URL**が含まれるようになりました。また、SQL のような Azure storage へのバックアップに必要な他のサポートオブジェクトも追加されました。証明. 詳細については、「 **Using Maintenance Plan Wizard** 」の「 [バックアップ タスクを定義する](../maintenance-plans/use-the-maintenance-plan-wizard.md#SSMSProcedure)」を参照してください。  
  
##  <a name="RestoreSSMS"></a>SQL Server Management Studio を使用した Azure storage からの復元  
 データベースを復元する場合、復元元のデバイスとして **[URL]** が用意されています。 次の手順では、Azure storage からの復元を可能にする復元タスクの変更について説明します。  
  
1.  SQL Server Management Studio の復元タスクの **[全般]** ページで **[デバイス]** を選択すると、 **[バックアップ デバイスの選択]** ダイアログ ボックスが表示されます。このダイアログ ボックスでは、バックアップ メディアの種類として **[URL]** を選択できます。  
  
2.  **[URL]** を選択し、 **[追加]** をクリックすると、 **[Azure ストレージへの接続]** ダイアログが開きます。 Azure storage に対して認証する SQL 資格情報を指定します。  
  
3.  SQL Server、指定した SQL 資格情報を使用して Azure storage に接続し、 **[azure でのバックアップファイルの検索]** ダイアログボックスを開きます。 このページには、ストレージに存在するバックアップ ファイルが表示されます。 復元に使用するファイルを選択して **[OK]** をクリックします。 これにより、 **[バックアップ デバイスの選択]** ダイアログ ボックスが再度表示されます。このダイアログ ボックスで **[OK]** をクリックすると、メインの **[復元]** ダイアログ ボックスに戻って復元を完了できます。  詳細については、次のトピックを参照してください。  
  
     [[データベースの復元] &#40;[全般] ページ&#41;](restore-database-general-page.md)  
  
     [[データベースの復元] &#40;[ファイル] ページ&#41;](restore-database-files-page.md)  
  
     [[データベースの復元] &#40;[オプション] ページ&#41;](restore-database-options-page.md)  
  
##  <a name="Examples"></a> コード例  
 ここでは、次の例について説明します。  
  
-   [Shared Access Signature の作成](#credential)  
  
-   [データベース全体をバックアップする](#complete)  
  
-   [データベースおよびログをバックアップする](#databaselog)  
  
-   [プライマリファイルグループの完全ファイルバックアップを作成する](#filebackup)  
  
-   [プライマリファイルグループのファイルの差分バックアップの作成](#differential)  
  
-   [データベースを復元しファイルを移動する](#restoredbwithmove)  
  
-   [STOPAT を使って特定の時点の状態に復元する](#PITR)  
  
###  <a name="credential"></a> Shared Access Signature の作成  
 次の例では、Azure Storage 認証情報を格納する資格情報を作成します。  
  
1.  **Tsql**  
  
    ```  
    IF NOT EXISTS  
    (SELECT * FROM sys.credentials   
    WHERE credential_identity = 'mycredential')  
    CREATE CREDENTIAL mycredential WITH IDENTITY = 'mystorageaccount'  
    ,SECRET = '<storage access key>' ;  
  
    ```  
  
2.  **C#**  
  
    ```  
    // Connect to default sql server instance on local machine  
    Server server = new Server(".");  
    string identity = "mystorageaccount";  
    string secret = "<storage access key>";  
  
    // Create a Credential  
    string credentialName = "mycredential";  
    Credential credential = new Credential(server, credentialName);  
    credential.Create(identity, secret);  
    ```  
  
3.  **PowerShell**  
  
    ```  
    # create variables  
    $storageAccount = "mystorageaccount"  
    $storageKey = "<storage access key>"  
    $secureString = convertto-securestring $storageKey  -asplaintext -force  
    $credentialName = "mycredential"  
  
    $srvPath = "SQLSERVER:\SQL\COMPUTERNAME\INSTANCENAME"  
    # for default instance, the $srvpath variable would be "SQLSERVER:\SQL\COMPUTERNAME\DEFAULT"  
  
    # Create a credential  
     New-SqlCredential -Name $credentialName -Path $srvpath -Identity $storageAccount -Secret $secureString  
  
    ```  
  
###  <a name="complete"></a>データベース全体のバックアップ  
 次の例では、AdventureWorks2012 データベースを Azure Blob ストレージサービスにバックアップします。  
  
1.  **Tsql**  
  
    ```  
    BACKUP DATABASE AdventureWorks2012   
    TO URL = 'https://mystorageaccount.blob.core.windows.net/mycontainer/AdventureWorks2012.bak'   
          WITH CREDENTIAL = 'mycredential'   
         ,COMPRESSION  
         ,STATS = 5;  
    GO  
  
    ```  
  
1.  **C#**  
  
    ```  
    // Connect to default sql server instance on local machine  
    Server server = new Server(".");  
    string identity = "mystorageaccount";  
  
    string credentialName = "mycredential";  
    string dbName = "AdventureWorks2012";  
    string blobContainerName = "mycontainer";  
  
    // Generate Unique Url  
    string url = String.Format(@"https://{0}.blob.core.windows.net/{1}/{2}-{3}.bak",  
            identity,  
            blobContainerName,  
            dbName,  
            DateTime.Now.ToString("s").Replace(":", "-"));  
  
    // Backup to Url  
    Backup backup = new Backup();  
    backup.CredentialName = credentialName;  
    backup.Database = dbName;  
    backup.CompressionOption = BackupCompressionOptions.On;  
    backup.Devices.AddDevice(url, DeviceType.Url);  
    backup.SqlBackup(server);  
    ```  
  
2.  **PowerShell**  
  
    ```  
    # create variables  
    $backupUrlContainer = "https://mystorageaccount.blob.core.windows.net/mycontainer/"  
    $credentialName = "mycredential"  
    $srvPath = "SQLSERVER:\SQL\COMPUTERNAME\INSTANCENAME"   
    # for default instance, the $srvpath varilable would be "SQLSERVER:\SQL\COMPUTERNAME\DEFAULT"  
  
    # navigate to SQL Server Instance  
    CD $srvPath   
    $backupFile = $backupUrlContainer + "AdventureWorks2012" +  ".bak"  
    Backup-SqlDatabase -Database AdventureWorks2012 -backupFile $backupFile  -SqlCredential $credentialName -CompressionOption On  
  
    ```  
  
###  <a name="databaselog"></a>データベースとログのバックアップ  
 次の例では、AdventureWorks2012 サンプル データベースをバックアップします。このデータベースでは、既定で単純復旧モデルが使用されています。 ここではまず、ログをバックアップするため、完全復旧モデルを使用するよう AdventureWorks2012 データベースを変更します。 この例では、Azure Blob へのデータベースの完全バックアップを作成し、更新操作の期間後に、ログをバックアップします。 この例では、日付と時刻のタイム スタンプを含むバックアップ ファイル名を作成します。  
  
1.  **Tsql**  
  
    ```  
    -- To permit log backups, before the full database backup, modify the database   
    -- to use the full recovery model.  
    USE master;  
    GO  
    ALTER DATABASE AdventureWorks2012  
       SET RECOVERY FULL;  
    GO  
  
    -- Back up the full AdventureWorks2012 database.  
           -- First create a file name for the backup file with DateTime stamp  
  
    DECLARE @Full_Filename AS VARCHAR (300);  
    SET @Full_Filename = 'https://mystorageaccount.blob.core.windows.net/mycontainer/AdventureWorks2012_Full_'+   
    REPLACE (REPLACE (REPLACE (CONVERT (VARCHAR (40), GETDATE (), 120), '-','_'),':', '_'),' ', '_') + '.bak';   
    --Back up Adventureworks2012 database  
  
    BACKUP DATABASE AdventureWorks2012  
    TO URL =  @Full_Filename  
    WITH CREDENTIAL = 'mycredential';  
    ,COMPRESSION  
    GO  
    -- Back up the AdventureWorks2012 log.  
    DECLARE @Log_Filename AS VARCHAR (300);  
    SET @Log_Filename = 'https://mystorageaccount.blob.core.windows.net/mycontainer/AdventureWorks2012_Log_'+   
    REPLACE (REPLACE (REPLACE (CONVERT (VARCHAR (40), GETDATE (), 120), '-','_'),':', '_'),' ', '_') + '.trn';  
    BACKUP LOG AdventureWorks2012  
     TO URL = @Log_Filename  
    WITH CREDENTIAL = 'mycredential'  
    ,COMPRESSION;  
    GO  
    ```  
  
2.  **C#**  
  
    ```  
    // Connect to default sql server instance on local machine  
    Server server = new Server(".");  
    string identity = "mystorageaccount";  
  
    string credentialName = "mycredential";  
    string dbName = "AdventureWorks2012";  
    string blobContainerName = "mycontainer";  
  
    // Generate Unique Url for data backup  
    string urlDataBackup = String.Format(@"https://{0}.blob.core.windows.net/{1}/{2}_Data-{3}.bak",  
            identity,  
            blobContainerName,  
            dbName,  
            DateTime.Now.ToString("s").Replace(":", "-"));  
  
    // Backup Database to Url  
    Backup backupData = new Backup();  
    backupData.CredentialName = credentialName;  
    backupData.Database = dbName;  
    backup.CompressionOption = BackupCompressionOptions.On;  
    backupData.Devices.AddDevice(urlDataBackup, DeviceType.Url);  
    backupData.SqlBackup(server);  
  
    // Generate Unique Url for data backup  
    string urlLogBackup = String.Format(@"https://{0}.blob.core.windows.net/{1}/{2}_Log-{3}.bak",  
            identity,  
            blobContainerName,  
            dbName,  
            DateTime.Now.ToString("s").Replace(":", "-"));  
  
    // Backup Database Log to Url  
    Backup backupLog = new Backup();  
    backupLog.CredentialName = credentialName;  
    backupLog.Database = dbName;  
    backup.CompressionOption = BackupCompressionOptions.On;  
    backupLog.Devices.AddDevice(urlLogBackup, DeviceType.Url);  
    backupLog.Action = BackupActionType.Log;  
    backupLog.SqlBackup(server);  
    ```  
  
3.  **PowerShell**  
  
    ```  
  
    #create variables  
    $backupUrlContainer = "https://mystorageaccount.blob.core.windows.net/mycontainer/"  
    $credentialName = "mycredential"  
    $srvPath = "SQLSERVER:\SQL\COMPUTERNAME\INSTANCENAME"  
    # for default instance, the $srvpath variable would be "SQLSERVER:\SQL\COMPUTERNAME\DEFAULT"  
  
    # navigate to theSQL Server Instance  
  
    CD $srvPath   
    #Create a unique file name for the full database backup  
    $backupFile = $backupUrlContainer + "AdventureWorks2012_" + (Get-Date).ToString("s").Replace("-","_").Replace(":", "_").Replace(" ","_").Replace("/", "_") +  ".bak"  
  
    #Backup Database to URL  
  
    Backup-SqlDatabase -Database AdventureWorks2012 -backupFile $backupFile  -SqlCredential $credentialName -CompressionOption On -BackupAction Database    
  
    #Create a unique file name for log backup  
  
    $backupFile = $backupUrlContainer + "AdventureWorks2012_" + (Get-Date).ToString("s").Replace("-","_").Replace(":", "_").Replace(" ","_").Replace("/", "_") +  ".trn"  
  
    #Backup Log to URL  
  
    Backup-SqlDatabase -Database AdventureWorks2012 -backupFile $backupFile  -SqlCredential $credentialName -CompressionOption On -BackupAction Log  
  
    ```  
  
###  <a name="filebackup"></a>プライマリファイルグループの完全ファイルバックアップを作成する  
 次の例では、プライマリ ファイル グループのファイル全体のバックアップを作成します。  
  
1.  **Tsql**  
  
    ```  
    --Back up the files in Primary:  
    BACKUP DATABASE AdventureWorks2012  
       FILEGROUP = 'Primary'  
       TO URL = 'https://mystorageaccount.blob.core.windows.net/mycontainer/AdventureWorks2012files.bck'  
       WITH CREDENTIAL = 'mycredential'  
       ,COMPRESSION;  
    GO  
    ```  
  
2.  **C#**  
  
    ```  
    // Connect to default sql server instance on local machine  
    Server server = new Server(".");  
    string identity = "mystorageaccount";  
  
    string credentialName = "mycredential";  
    string dbName = "AdventureWorks2012";  
    string blobContainerName = "mycontainer";  
  
    // Generate Unique Url  
    string url = String.Format(@"https://{0}.blob.core.windows.net/{1}/{2}-{3}.bck",  
            identity,  
            blobContainerName,  
            dbName,  
            DateTime.Now.ToString("s").Replace(":", "-"));  
  
    // Backup to Url  
    Backup backup = new Backup();  
    backup.CredentialName = credentialName;  
    backup.Database = dbName;  
    backup.Action = BackupActionType.Files;  
    backup.DatabaseFileGroups.Add("PRIMARY");  
    backup.CompressionOption = BackupCompressionOptions.On;  
    backup.Devices.AddDevice(url, DeviceType.Url);  
    backup.SqlBackup(server);  
  
    ```  
  
3.  **PowerShell**  
  
    ```  
  
    #create variables  
    $backupUrlContainer = "https://mystorageaccount.blob.core.windows.net/mycontainer/"  
    $credentialName = "mycredential"  
    $srvPath = "SQLSERVER:\SQL\COMPUTERNAME\INSTANCENAME"  
    # for default instance, the $srvpath variable would be "SQLSERVER:\SQL\COMPUTERNAME\DEFAULT"  
  
    # navigate to the SQL Server Instance  
  
    CD $srvPath   
    #Create a unique file name for the file backup  
    $backupFile = $backupUrlContainer + "AdventureWorks2012_" + (Get-Date).ToString("s").Replace("-","_").Replace(":", "_").Replace(" ","_").Replace("/", "_") +  ".bck"  
  
    #Backup Primary File Group to URL  
  
    Backup-SqlDatabase -Database AdventureWorks2012 -backupFile $backupFile  -SqlCredential $credentialName -CompressionOption On -BackupAction Files -DatabaseFileGroup Primary  
  
    ```  
  
###  <a name="differential"></a>プライマリファイルグループのファイルの差分バックアップを作成する  
 次の例では、プライマリ ファイル グループのファイルの差分バックアップを作成します。  
  
1.  **Tsql**  
  
    ```  
    --Back up the files in Primary:  
    BACKUP DATABASE AdventureWorks2012  
       FILEGROUP = 'Primary'  
       TO URL = 'https://mystorageaccount.blob.core.windows.net/mycontainer/AdventureWorks2012filesdiff.bck'  
       WITH   
          CREDENTIAL = 'mycredential'  
          ,COMPRESSION  
      ,DIFFERENTIAL;  
    GO  
  
    ```  
  
2.  **C#**  
  
    ```  
    // Connect to default sql server instance on local machine  
    Server server = new Server(".");  
    string identity = "mystorageaccount";  
  
    string credentialName = "mycredential";  
    string dbName = "AdventureWorks2012";  
    string blobContainerName = "mycontainer";  
  
    // Generate Unique Url  
    string url = String.Format(@"https://{0}.blob.core.windows.net/{1}/{2}-{3}.bak",  
            identity,  
            blobContainerName,  
            dbName,  
            DateTime.Now.ToString("s").Replace(":", "-"));  
  
    // Backup to Url  
    Backup backup = new Backup();  
    backup.CredentialName = credentialName;  
    backup.Database = dbName;  
    backup.Action = BackupActionType.Files;  
    backup.DatabaseFileGroups.Add("PRIMARY");  
    backup.Incremental = true;  
    backup.CompressionOption = BackupCompressionOptions.On;  
    backup.Devices.AddDevice(url, DeviceType.Url);  
    backup.SqlBackup(server);  
  
    ```  
  
3.  **PowerShell**  
  
    ```  
  
    #create variables  
    $backupUrlContainer = "https://mystorageaccount.blob.core.windows.net/mycontainer/"  
    $credentialName = "mycredential"  
    $srvPath = "SQLSERVER:\SQL\COMUTERNAME\INSTANCENAME"  
    # for default instance, the $srvpath variable would be "SQLSERVER:\SQL\COMPUTERNAME\DEFAULT"  
  
    # navigate to SQL Server Instance  
  
    CD $srvPath   
  
    #create a unique file name for the full backup  
    $backupdbFile = $backupUrlContainer + "AdventureWorks2012_" + (Get-Date).ToString("s").Replace("-","_").Replace(":", "_").Replace(" ","_").Replace("/", "_") +  ".bak"  
  
    #Create a differential backup of the primary filegroup  
  
    Backup-SqlDatabase -Database AdventureWorks2012 -backupFile $backupFile  -SqlCredential $credentialName -CompressionOption On -BackupAction Files -DatabaseFileGroup Primary -Incremental  
  
    ```  
  
###  <a name="restoredbwithmove"></a>データベースを復元してファイルを移動する  
 データベースの完全バックアップを復元し、復元したデータベースを C:\Program Files\Microsoft SQL Server\MSSQL12.MSSQLSERVER\MSSQL\Data ディレクトリに移動するには、次の手順を実行します。  
  
1.  **Tsql**  
  
    ```  
    -- Backup the tail of the log first  
  
    DECLARE @Log_Filename AS VARCHAR (300);  
    SET @Log_Filename = 'https://mystorageaccount.blob.core.windows.net/mycontainer/AdventureWorks2012_Log_'+   
    REPLACE (REPLACE (REPLACE (CONVERT (VARCHAR (40), GETDATE (), 120), '-','_'),':', '_'),' ', '_') + '.trn';  
    BACKUP LOG AdventureWorks2012  
     TO URL = @Log_Filename  
    WITH CREDENTIAL = 'mycredential'  
    ,NORECOVERY;  
    GO  
  
    RESTORE DATABASE AdventureWorks2012 FROM URL = 'https://mystorageaccount.blob.core.windows.net/mycontainer/AdventureWorks2012.bak'  
    WITH CREDENTIAL = 'mycredential'  
    ,MOVE 'AdventureWorks2012_data' to 'C:\Program Files\Microsoft SQL Server\myinstance\MSSQL\DATA\AdventureWorks2012.mdf'  
    ,MOVE 'AdventureWorks2012_log' to 'C:\Program Files\Microsoft SQL Server\myinstance\MSSQL\DATA\AdventureWorks2012.ldf'  
    ,STATS = 5  
  
    ```  
  
2.  **C#**  
  
    ```  
    // Connect to default sql server instance on local machine  
    Server server = new Server(".");  
    string identity = "mystorageaccount";  
  
    string credentialName = "mycredential";  
    string dbName = "AdventureWorks2012";  
    string blobContainerName = "mycontainer";  
  
    // Generate Unique Url  
    string urlBackupData = String.Format(@"https://{0}.blob.core.windows.net/{1}/{2}-Data{3}.bak",  
            identity,  
            blobContainerName,  
            dbName,  
            DateTime.Now.ToString("s").Replace(":", "-"));  
  
    // Backup to Url  
    Backup backup = new Backup();  
    backup.CredentialName = credentialName;  
    backup.Database = dbName;  
    backup.Devices.AddDevice(urlBackupData, DeviceType.Url);  
    backup.SqlBackup(server);  
  
    // Generate Unique Url for tail log backup  
    string urlTailLogBackup = String.Format(@"https://{0}.blob.core.windows.net/{1}/{2}-TailLog{3}.bak",  
            identity,  
            blobContainerName,  
            dbName,  
            DateTime.Now.ToString("s").Replace(":", "-"));  
  
    // Backup Tail Log to Url  
    Backup backupTailLog = new Backup();  
    backupTailLog.CredentialName = credentialName;  
    backupTailLog.Database = dbName;  
    backupTailLog.Action = BackupActionType.Log;  
    backupTailLog.NoRecovery = true;  
    backupTailLog.Devices.AddDevice(urlTailLogBackup, DeviceType.Url);  
    backupTailLog.SqlBackup(server);  
  
    // Restore a database and move files  
    string newDataFilePath = server.MasterDBLogPath  + @"\" + dbName + DateTime.Now.ToString("s").Replace(":", "-") + ".mdf";  
    string newLogFilePath = server.MasterDBLogPath  + @"\" + dbName + DateTime.Now.ToString("s").Replace(":", "-") + ".ldf";  
  
    Restore restore = new Restore();  
    restore.CredentialName = credentialName;  
    restore.Database = dbName;  
    restore.ReplaceDatabase = true;  
    restore.Devices.AddDevice(urlBackupData, DeviceType.Url);  
    restore.RelocateFiles.Add(new RelocateFile(dbName, newDataFilePath));  
    restore.RelocateFiles.Add(new RelocateFile(dbName+ "_Log", newLogFilePath));  
    restore.SqlRestore(server);  
  
    ```  
  
3.  **PowerShell**  
  
    ```  
  
    #create variables  
    $backupUrlContainer = "https://mystorageaccount.blob.core.windows.net/mycontainer/"  
    $credentialName = "mycredential"  
    $srvPath = "SQLSERVER:\SQL\COMPUTERNAME\INSTNACENAME"  
    # for default instance, the $srvpath variable would be "SQLSERVER:\SQL\COMPUTERNAME\DEFAULT"  
  
    # navigate to SQL Server Instance   
  
    CD $srvPath   
  
    #create a unique file name for the full backup  
    $backupdbFile = $backupUrlContainer + "AdventureWorks2012_" + (Get-Date).ToString("s").Replace("-","_").Replace(":", "_").Replace(" ","_").Replace("/", "_") +  ".bak"  
  
    # Full database backup to URL  
    Backup-SqlDatabase -Database AdventureWorks2012 -backupFile $backupdbFile  -SqlCredential $credentialName -CompressionOption On      
  
    #Create a unique file name for the tail log backup  
    $backuplogFile = $backupUrlContainer + "AdventureWorks2012_" + (Get-Date).ToString("s").Replace("-","_").Replace(":", "_").Replace(" ","_").Replace("/", "_") +  ".trn"  
  
    #Backup tail log to URL  
  
    Backup-SqlDatabase -Database AdventureWorks2012 -backupFile $backupFile  -SqlCredential $credentialName  -BackupAction Log -NoRecovery    
  
    # Restore Database and move files  
  
    $newDataFilePath = New-Object Microsoft.SqlServer.Management.Smo.RelocateFile ("AdventureWorks_Data","C:\Program Files\Microsoft SQL Server\myinstance\MSSQL\DATA\AdventureWorks2012.mdf")  
    $newLogFilePath = New-Object Microsoft.SqlServer.Management.Smo.RelocateFile("AdventureWorks_Log","C:\Program Files\Microsoft SQL Server\myinstance\MSSQL\DATA\AdventureWorks2012.ldf")  
  
    Restore-SqlDatabase -Database AdventureWorks2012 -SqlCredential $credentialName -BackupFile $backupdbFile -RelocateFile @($newDataFilePath,$newLogFilePath)  
  
    ```  
  
###  <a name="PITR"></a> STOPAT を使って特定の時点の状態に復元する  
 次の例では、データベースを特定の時点の状態に復元し、復元操作を示します。  
  
1.  **Tsql**  
  
    ```  
    RESTORE DATABASE AdventureWorks FROM URL = 'https://mystorageaccount.blob.core.windows.net/mycontainer/AdventureWorks2012.bak'   
    WITH   
     CREDENTIAL = 'mycredential'  
    ,MOVE 'AdventureWorks2012_data' to 'C:\Program Files\Microsoft SQL Server\myinstance\MSSQL\DATA\AdventureWorks2012.mdf'  
    ,Move 'AdventureWorks2012_log' to 'C:\Program Files\Microsoft SQL Server\myinstance\MSSQL\DATA\AdventureWorks2012.ldf'  
    ,NORECOVERY  
    --,REPLACE  
    ,STATS = 5;  
    GO   
  
    RESTORE LOG AdventureWorks FROM URL = 'https://mystorageaccount.blob.core.windows.net/mycontainer/AdventureWorks2012.trn'   
    WITH CREDENTIAL = 'mycredential'  
    ,RECOVERY   
    ,STOPAT = 'Oct 23, 2012 5:00 PM'   
    GO  
    ```  
  
2.  **C#**  
  
    ```  
    // Connect to default sql server instance on local machine  
    Server server = new Server(".");  
    string identity = "mystorageaccount";  
  
    string credentialName = "mycredential";  
    string dbName = "AdventureWorks2012";  
    string blobContainerName = "mycontainer";  
  
    // Generate Unique Url  
    string urlBackupData = String.Format(@"https://{0}.blob.core.windows.net/{1}/{2}-Data{3}.bak",  
            identity,  
            blobContainerName,  
            dbName,  
            DateTime.Now.ToString("s").Replace(":", "-"));  
  
    // Backup to Url  
    Backup backup = new Backup();  
    backup.CredentialName = credentialName;  
    backup.Database = dbName;  
    backup.Devices.AddDevice(urlBackupData, DeviceType.Url);  
    backup.SqlBackup(server);  
  
    // Generate Unique Url for Tail Log backup  
    string urlTailLogBackup = String.Format(@"https://{0}.blob.core.windows.net/{1}/{2}-TailLog{3}.bak",  
            identity,  
            blobContainerName,  
            dbName,  
            DateTime.Now.ToString("s").Replace(":", "-"));  
  
    // Backup Tail Log to Url  
    Backup backupTailLog = new Backup();  
    backupTailLog.CredentialName = credentialName;  
    backupTailLog.Database = dbName;  
    backupTailLog.Action = BackupActionType.Log;  
    backupTailLog.NoRecovery = true;  
    backupTailLog.Devices.AddDevice(urlTailLogBackup, DeviceType.Url);  
    backupTailLog.SqlBackup(server);  
  
    // Restore a database and move files  
    string newDataFilePath = server.MasterDBLogPath + @"\" + dbName + DateTime.Now.ToString("s").Replace(":", "-") + ".mdf";  
    string newLogFilePath = server.MasterDBLogPath + @"\" + dbName + DateTime.Now.ToString("s").Replace(":", "-") + ".ldf";  
  
    Restore restore = new Restore();  
    restore.CredentialName = credentialName;  
    restore.Database = dbName;  
    restore.ReplaceDatabase = true;  
    restore.NoRecovery = true;  
    restore.Devices.AddDevice(urlBackupData, DeviceType.Url);  
    restore.RelocateFiles.Add(new RelocateFile(dbName, newDataFilePath));  
    restore.RelocateFiles.Add(new RelocateFile(dbName + "_Log", newLogFilePath));  
    restore.SqlRestore(server);  
  
    // Restore transaction Log with stop at   
    Restore restoreLog = new Restore();  
    restoreLog.CredentialName = credentialName;  
    restoreLog.Database = dbName;  
    restoreLog.Action = RestoreActionType.Log;  
    restoreLog.Devices.AddDevice(urlBackupData, DeviceType.Url);  
    restoreLog.ToPointInTime = DateTime.Now.ToString();   
    restoreLog.SqlRestore(server);  
  
    ```  
  
3.  **PowerShell**  
  
    ```  
  
    #create variables  
    $backupUrlContainer = "https://mystorageaccount.blob.core.windows.net/mycontainer/"  
    $credentialName = "mycredential"  
    $srvPath = "SQLSERVER:\SQL\COMPUTERNAME\INSTANCENAME"  
    # for default instance, the $srvpath variable would be "SQLSERVER:\SQL\COMPUTERNAME\DEFAULT"  
  
    # Navigate to SQL Server Instance Directory  
  
    CD $srvPath   
  
    #create a unique file name for the full backup  
    $backupdbFile = $backupUrlContainer + "AdventureWorks2012_" + (Get-Date).ToString("s").Replace("-","_").Replace(":", "_").Replace(" ","_").Replace("/", "_") +  ".bak"  
  
    # Full database backup to URL  
    Backup-SqlDatabase -Database AdventureWorks2012 -backupFile $backupdbFile  -SqlCredential $credentialName -CompressionOption On     
  
    #Create a unique file name for the tail log backup  
    $backuplogFile = $backupUrlContainer + "AdventureWorks2012_" + (Get-Date).ToString("s").Replace("-","_").Replace(":", "_").Replace(" ","_").Replace("/", "_") +  ".trn"  
  
    #Backup tail log to URL  
  
    Backup-SqlDatabase -Database AdventureWorks2012 -backupFile $backupFile  -SqlCredential $credentialName  -BackupAction Log -NoRecovery     
  
    # Restore Database and move files  
  
    $newDataFilePath = New-Object Microsoft.SqlServer.Management.Smo.RelocateFile ("AdventureWorks_Data","C:\Program Files\Microsoft SQL Server\myinstance\MSSQL\DATA\AdventureWorks2012.mdf")  
    $newLogFilePath = New-Object Microsoft.SqlServer.Management.Smo.RelocateFile("AdventureWorks_Log","C:\Program Files\Microsoft SQL Server\myinstance\MSSQL\DATA\AdventureWorks2012.ldf")  
  
    Restore-SqlDatabase -Database AdventureWorks2012 -SqlCredential $credentialName -BackupFile $backupdbFile -RelocateFile @($newDataFilePath,$newLogFilePath) -NoRecovery    
  
    # Restore Transaction log with Stop At:  
    Restore-SqlDatabase -Database AdventureWorks2012 -SqlCredential $credentialName -BackupFile $backuplogFile  -ToPointInTime (Get-Date).ToString()  
  
    ```  
  
## <a name="see-also"></a>関連項目  
 [SQL Server Backup to URL に関するベスト プラクティスとトラブルシューティング](sql-server-backup-to-url-best-practices-and-troubleshooting.md)   
 [システム データベースのバックアップと復元 &#40;SQL Server&#41;](back-up-and-restore-of-system-databases-sql-server.md)   
 [チュートリアル: Azure Blob Storage サービスへのバックアップと復元の SQL Server](../tutorial-sql-server-backup-and-restore-to-azure-blob-storage-service.md)  
  
  
