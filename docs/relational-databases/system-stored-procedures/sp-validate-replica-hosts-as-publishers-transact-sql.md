---
title: sp_validate_replica_hosts_as_publishers (Transact-sql) |Microsoft Docs
ms.custom: ''
ms.date: 03/14/2017
ms.prod: sql
ms.prod_service: database-engine
ms.reviewer: ''
ms.technology: replication
ms.topic: language-reference
f1_keywords:
- sp_validate_replica_hosts_as_publishers_TSQL
- sp_validate_replica_hosts_as_publishers
helpviewer_keywords:
- sp_validate_replica_hosts_as_publishers
ms.assetid: 45001fc9-2dbd-463c-af1d-aa8982d8c813
author: MashaMSFT
ms.author: mathoma
ms.openlocfilehash: 8df9c4fcc88f568c920f0a5959338f195d79d925
ms.sourcegitcommit: c426c7ef99ffaa9e91a93ef653cd6bf3bfd42132
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/10/2019
ms.locfileid: "72252109"
---
# <a name="sp_validate_replica_hosts_as_publishers-transact-sql"></a>sp_validate_replica_hosts_as_publishers (Transact-SQL)
[!INCLUDE[tsql-appliesto-ss2012-xxxx-xxxx-xxx-md](../../includes/tsql-appliesto-ss2012-xxxx-xxxx-xxx-md.md)]

  **sp_validate_replica_hosts_as_publishers**は、現在のプライマリレプリカだけではなく、すべてのセカンダリレプリカの検証を可能にする**sp_validate_redirected_publisher**の拡張機能です。 **sp_validate_replicat_hosts_as_publisher**は、Always On レプリケーショントポロジ全体を検証します。 **sp_validate_replica_hosts_as_publishers**は、ダブルホップセキュリティエラー (21892) を回避するために、リモートデスクトップセッションを使用してディストリビューターで直接実行する必要があります。  
  
 ![トピック リンク アイコン](../../database-engine/configure-windows/media/topic-link.gif "トピック リンク アイコン") [Transact-SQL 構文表記規則](../../t-sql/language-elements/transact-sql-syntax-conventions-transact-sql.md)  
  
## <a name="syntax"></a>構文  
  
```  
  
sp_validate_replica_hosts_as_publishers   
    [ @original_publisher = ] 'original_publisher',  
    [ @publisher_db = ] 'database_name',   
    [ @redirected_publisher = ] 'new_publisher' output  
```  
  
## <a name="arguments"></a>引数  
`[ @original_publisher = ] 'original_publisher'` は、データベースを最初にパブリッシュした @no__t のインスタンスの名前です。 *original_publisher*は**sysname**,、既定値はありません。  
  
`[ @publisher_db = ] 'publisher_db'` パブリッシュするデータベースの名前。 *publisher_db* は **sysname** 、既定値はありません。  
  
`[ @redirected_publisher = ] 'redirected_publisher'` 元のパブリッシャー/パブリッシュされたデータベースのペアに対して**sp_redirect_publisher**が呼び出されたときの、リダイレクトの対象です。 *redirected_publisher*は**sysname**,、既定値はありません。  
  
## <a name="return-code-values"></a>リターン コードの値  
 **0** (成功) または**1** (失敗)  
  
## <a name="result-sets"></a>結果セット  
 [なし] :  
  
## <a name="remarks"></a>コメント  
 パブリッシャーとパブリッシングデータベースのエントリが存在しない場合、 **sp_validate_redirected_publisher**は出力パラメーター *\@redirected_publisher*に null を返します。 それ以外の場合は、成功と失敗の両方で、関連するリダイレクトされたパブリッシャーが返されます。  
  
 検証が成功した場合、 **sp_validate_redirected_publisher**は成功を示す値を返します。  
  
 検証が失敗した場合は、適切なエラーが発生します。  **sp_validate_redirected_publisher**は、最初に発生した問題だけでなく、すべての問題を発生させるためのベストエフォートを行います。  
  
> [!NOTE]  
>  セカンダリ レプリカのホストで読み取りアクセスが許可されていない場合や、読み取りを目的としたアクセスを指定する必要がある場合、**sp_validate_replica_hosts_as_publishers** による検証は失敗し、次のエラー メッセージが表示されます。  
>   
>  メッセージ 21899、レベル 11、状態 1、プロシージャ **sp_hadr_verify_subscribers_at_publisher**、行 109  
>   
>  元のパブリッシャー 'MyOriginalPublisher' のサブスクライバーの sysserver エントリがあるかどうかを判断するために、リダイレクトされたパブリッシャー 'MyReplicaHostName' で実行したクエリが、エラー '976'、エラー メッセージ 'エラー 976、レベル 14、状態 1、メッセージ: 対象になるデータベース 'MyPublishedDB' は可用性グループに参加しているため、現在クエリでアクセスできません。 データ移動が中断されているか、可用性レプリカの読み取りアクセスが有効になっていません。 このデータベースや可用性グループの他のデータベースへの読み取り専用アクセスを許可するには、グループの 1 つ以上のセカンダリ可用性レプリカへの読み取りアクセスを有効にします。  詳細については、 **オンライン ブックの** ALTER AVAILABILITY GROUP [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] ステートメントのトピックをご覧ください。  
>   
>  レプリカ ホスト 'MyReplicaHostName' について、1 つまたは複数のパブリッシャー検証エラーが発生しました。  
  
## <a name="permissions"></a>アクセス許可  
 呼び出し元は、 **sysadmin**固定サーバーロールのメンバー、ディストリビューションデータベースの**db_owner**固定データベースロールのメンバーであるか、パブリッシャーデータベースに関連付けられている定義済みパブリケーションのパブリケーションアクセスリストのメンバーである必要があります。  
  
## <a name="see-also"></a>関連項目  
 [レプリケーション ストアド プロシージャ &#40;Transact-SQL&#41;](../../relational-databases/system-stored-procedures/replication-stored-procedures-transact-sql.md)   
 [sp_get_redirected_publisher &#40;transact-sql&#41;](../../relational-databases/system-stored-procedures/sp-get-redirected-publisher-transact-sql.md)   
 [sp_redirect_publisher &#40;transact-sql&#41;](../../relational-databases/system-stored-procedures/sp-redirect-publisher-transact-sql.md)   
 [sp_validate_redirected_publisher &#40;transact-sql&#41;](../../relational-databases/system-stored-procedures/sp-validate-redirected-publisher-transact-sql.md)  
  
  
