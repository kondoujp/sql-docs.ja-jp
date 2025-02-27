---
title: sp_update_proxy (Transact-sql) |Microsoft Docs
ms.custom: ''
ms.date: 03/14/2017
ms.prod: sql
ms.prod_service: database-engine
ms.reviewer: ''
ms.technology: system-objects
ms.topic: language-reference
f1_keywords:
- sp_update_proxy
- sp_update_proxy_TSQL
dev_langs:
- TSQL
helpviewer_keywords:
- ALTER PROXY statement
- sp_update_proxy
ms.assetid: 864fd0e6-9d61-4f07-92ef-145318d2f881
author: VanMSFT
ms.author: vanto
ms.openlocfilehash: ec6c40abd080c86722565762fab3b4f9d30bd0c0
ms.sourcegitcommit: 43c3d8939f6f7b0ddc493d8e7a643eb7db634535
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/14/2019
ms.locfileid: "72305307"
---
# <a name="sp_update_proxy-transact-sql"></a>sp_update_proxy (Transact-sql)
[!INCLUDE[tsql-appliesto-ss2008-xxxx-xxxx-xxx-md](../../includes/tsql-appliesto-ss2008-xxxx-xxxx-xxx-md.md)]

  既存のプロキシのプロパティを変更します。  
  
 ![トピック リンク アイコン](../../database-engine/configure-windows/media/topic-link.gif "トピック リンク アイコン") [Transact-SQL 構文表記規則](../../t-sql/language-elements/transact-sql-syntax-conventions-transact-sql.md)  
  
## <a name="syntax"></a>構文  
  
```  
  
sp_update_proxy   
    [ @proxy_id = ] id,  
    [ @proxy_name = ] 'proxy_name',  
    [ @credential_name = ] 'credential_name' ,  
    [ @credential_id = ] credential_id ,  
    [ @new_name = ] 'new_name' ,  
    [ @enabled = ] is_enabled ,  
    [ @description = ] 'description'  
```  
  
## <a name="arguments"></a>引数  
`[ @proxy_id = ] id` は、変更するプロキシのプロキシ識別番号を指定します。 *Proxy_id*は**int**,、既定値は NULL です。  
  
`[ @proxy_name = ] 'proxy_name'` 変更するプロキシの名前。 *Proxy_name*の部分は**sysname**で、既定値は NULL です。  
  
`[ @credential_name = ] 'credential_name'` プロキシの新しい資格情報の名前。 *Credential_name*の部分は**sysname**で、既定値は NULL です。 *Credential_name*または*credential_id*のいずれかを指定できます。  
  
`[ @credential_id = ] credential_id` プロキシの新しい資格情報の識別番号。 *Credential_id*は**int**,、既定値は NULL です。 *Credential_name*または*credential_id*のいずれかを指定できます。  
  
`[ @new_name = ] 'new_name'` プロキシの新しい名前。 *新しい名前*の部分は**sysname**で、既定値は NULL です。 指定した場合、プロシージャはプロキシの名前を*新しい名前*に変更します。 この引数が NULL の場合、プロキシの名前は変更されません。  
  
`[ @enabled = ] is_enabled` は、プロキシが有効になっているかどうかを示します。 *Is_enabled*フラグは**tinyint**,、既定値は NULL です。 *Is_enabled*が**0**の場合、プロキシは有効ではなく、ジョブステップでは使用できません。 この引数が NULL の場合、プロキシの状態は変更されません。  
  
`[ @description = ] 'description'` プロキシの新しい説明。 *説明*は**nvarchar (512)** ,、既定値は NULL です。 この引数が NULL の場合、プロキシの説明は変更されません。  
  
## <a name="return-code-values"></a>リターン コードの値  
 **0** (成功) または**1** (失敗)  
  
## <a name="remarks"></a>コメント  
 **@No__t-1proxy_name**または **\@proxy_id**を指定する必要があります。 両方の引数を指定する場合は、両方とも同じプロキシを参照する必要があります。異なるプロキシを参照する場合、ストアド プロシージャは失敗します。  
  
 プロキシの資格情報を変更するには、 **@no__t 1credential_name**または **@no__t の**いずれかを指定する必要があります。 両方の引数を指定する場合は、両方とも同じ資格情報を参照する必要があります。異なる資格情報を参照する場合、ストアド プロシージャは失敗します。  
  
 このプロシージャでプロキシが変更されますが、プロキシへのアクセスは変更されません。 プロキシへのアクセスを変更するには、 **sp_grant_login_to_proxy**と**sp_revoke_login_from_proxy**を使用します。  
  
## <a name="permissions"></a>アクセス許可  
 このプロシージャを実行できるのは、 **sysadmin**固定セキュリティロールのメンバーだけです。  
  
## <a name="examples"></a>使用例  
 次の例では、プロキシ `Catalog application proxy` の enabled の値を `0` に設定します。  
  
```  
USE msdb ;  
GO  
  
EXEC dbo.sp_update_proxy  
    @proxy_name = 'Catalog application proxy',  
    @enabled = 0;  
GO  
```  
  
## <a name="see-also"></a>関連項目  
 [ストアドプロシージャ&#40;の SQL Server エージェント transact-sql&#41;](../../relational-databases/system-stored-procedures/sql-server-agent-stored-procedures-transact-sql.md)   
 [SQL Server エージェントセキュリティ](../../ssms/agent/implement-sql-server-agent-security.md)@no__t を実装する-1  
 [sp_add_proxy &#40;transact-sql&#41;](../../relational-databases/system-stored-procedures/sp-add-proxy-transact-sql.md)   
 [sp_delete_proxy &#40;transact-sql&#41;](../../relational-databases/system-stored-procedures/sp-delete-proxy-transact-sql.md)   
 [sp_grant_login_to_proxy &#40;transact-sql&#41;](../../relational-databases/system-stored-procedures/sp-grant-login-to-proxy-transact-sql.md)   
 [sp_revoke_login_from_proxy &#40;transact-sql&#41;](../../relational-databases/system-stored-procedures/sp-revoke-login-from-proxy-transact-sql.md)  
  
  
