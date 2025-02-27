---
title: SQL Server 2019 リリース ノート | Microsoft Docs
ms.date: 10/07/2019
ms.prod: sql
ms.reviewer: ''
ms.technology: release-landing
ms.topic: article
ms.assetid: 13942af8-5a40-4cef-80f5-918386767a47
author: MikeRayMSFT
ms.author: mikeray
monikerRange: = sql-server-ver15 || = sqlallproducts-allversions
ms.openlocfilehash: 9b6895abfa0b09459911eba03b52837379f2d162
ms.sourcegitcommit: 512acc178ec33b1f0403b5b3fd90e44dbf234327
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/08/2019
ms.locfileid: "72041191"
---
# <a name="sql-server-2019-preview-release-notes"></a>SQL Server 2019 プレビュー リリース ノート
[!INCLUDE[tsql-appliesto-ssver15-xxxx-xxxx-xxx](../includes/tsql-appliesto-ssver15-xxxx-xxxx-xxx.md)]

この記事では、[!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] の制限事項と既知の問題について説明します。 関連情報については、次を参照してください。

>[SQL Server 2019 の新機能](../sql-server/what-s-new-in-sql-server-ver15.md)

>[!NOTE]
>内容は [!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] リリース候補に関するものです。 リリース候補はプレリリース版ソフトウェアです。 情報は変更される可能性があります。 サポートのシナリオについては、「[サポート](#support)」を参照してください。

## <a name="includesql-server-2019includessssqlv15-mdmd-release-candidate-rc"></a>[!INCLUDE[sql-server-2019](../includes/sssqlv15-md.md)] リリース候補 (RC)

[!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] RC は、[!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] の最新のパブリック リリースです。

[!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] RC は、評価版としてのみ使用可能です。 他のエディションは提供されません。

リリース候補ソフトウェアのサポートとライセンスに関するすべての詳細情報については、インストール メディアの `license_Eval.rtf` に記載されています。

[!INCLUDE[ctp-support-exclusion](../includes/ctp-support-exclusion.md)]

## <a name="documentation"></a>ドキュメント

- **問題およびユーザーへの影響**:SQL Server 2019 (15.x) のドキュメントは制限されており、コンテンツは [!INCLUDE[ssSQL17](../includes/sssql17-md.md)] ドキュメント セットに付属しています。 記事の中で SQL Server 2019 (15.x) に固有の内容は、**適用対象**で示されています。

- **問題およびユーザーへの影響**: [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)]ドキュメントはバージョンによってフィルター処理できます。 各ドキュメント ページの左上にあるコントロールを使用し、要件に応じてフィルター処理してください。

- **問題およびユーザーへの影響**:SQL Server 2019 (15.x) には、オフライン コンテンツがありません。

## <a name="build-number"></a>ビルド番号

Windows、Linux、およびコンテナーでの SQL Server 2019 RC のビルド番号は `15.0.1900.25` です。  ビッグ データ クラスターで使用される SQL Server 2019 RC のビルド番号は `15.0.1900.47` です。

## <a name="hardware-and-software-requirements"></a>ハードウェアとソフトウェアの要件

- **問題およびユーザーへの影響**:ハードウェア要件とソフトウェア要件は現在レビュー段階であり、製品リリース用の最終版ではありません。

  - **ハードウェア**
    - [Windows - プロセッサ、メモリ、およびオペレーティング システムの要件](../sql-server/install/hardware-and-software-requirements-for-installing-sql-server.md#pmosr)
    - [Linux - システム要件](../linux/sql-server-linux-setup.md#system)
  - **ソフトウェア**
    - Windows Server 2016 以降。 その他の要件については、[SQL Server のインストール要件](../sql-server/install/hardware-and-software-requirements-for-installing-sql-server.md)に関する記事をご覧ください
    - Microsoft .NET Framework 4.6.2。 [ダウンロード センター](https://www.microsoft.com/download/details.aspx?id=53344)から入手できます。
    - Linux については、[サポートされている Linux プラットフォーム](../linux/sql-server-linux-setup.md#supportedplatforms)に関する記事をご覧ください

## <a name="sql-server-installation-may-fail-if-ssms-18x-is-installed"></a>SSMS 18.x がインストールされていると SQL Server のインストールが失敗する場合がある

- **問題およびユーザーへの影響**: 次のインストールをこの順序で実行すると、[!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] のインストールが失敗します。
  1. SQL Server Management Studio (SSMS) バージョン 18.0、18.1、18.2、または 18.3 がサーバーにインストールされています。
  1. リムーバブル メディアから [!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] のインストールを試みます。 たとえば、DVD などのインストール メディアです。

- **回避策**:
  1. SSMS 18.3.1 より古いバージョンの SSMS をすべてアンインストールします。
  1. 新しいバージョンの SSMS (18.3.1 以降) をインストールします。 最新のバージョンについては、[SSMS のダウンロード](../ssms/download-sql-server-management-studio-ssms.md)に関するページを参照してください。
  1. [!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] を普通にインストールします。

  >[!NOTE]
  >アンインストールする必要があります。

- **適用対象**: [!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] リリース候補。

## <a name="updated-compiler"></a>更新されたコンパイラ

- **問題およびユーザーへの影響**: [!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] は更新されたコンパイラで構築されています。 コンパイラの更新の結果、CTP 2.1 には浮動小数点数の結果と他の変換シナリオで、前のバージョンとは異なる値が返される場合があるという既知の問題がありました。 CTP 2.2 には、影響のあるシナリオで [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] の前のバージョンと同じ結果が返されるようにする作業が含まれています。 リリース候補リリースでは、もう問題は解決されたと考えています。 [!INCLUDE[ss2017](../includes/sssqlv14-md.md)] と比較した結果に異常がある場合、[[!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] チーム](https://aka.ms/sqlfeedback)にすぐに報告してください。

- **回避策**:なし

- **適用対象**:リリース候補

## <a name="installation-wizard-may-wait-between-eula-pages"></a>インストール ウィザードが使用許諾契約ページの間で待機状態になることがある

- **問題およびユーザーへの影響**:インストール ウィザードでのインストールの間に、R Services の使用許諾契約 (EULA) と Python の EULA の間で、プロセスが長時間待機状態になることがあります。

- **回避策**:インストール ウィザードが動くまで待ちます。 待つ時間は 30 分を超える可能性があります。

- **適用対象**:[!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] CTP 3.0

## <a name="utf-8-collations"></a>UTF-8 対応の照合順序

- **問題およびユーザーへの影響**:UTF-8 対応の照合順序は、他の [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] 機能とは併用できない場合があります。 UTF-8 は、次の [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] 機能が使用されている場合にはサポートされません。

  - インメモリ OLTP
  - PolyBase 用の外部テーブル ([!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] RC 1)
  - Always Encrypted ([!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] RC 1 以前)
  - リンク サーバー ([!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] CTP 3.2 以前)

  > [!Note]
  > Azure Data Studio および SQL Server Data Tools (SSDT) では、UTF-8 対応の照合順序を選択するための UI は現在のところサポートされていません。 最新の [!INCLUDE[ssManStudioFull](../includes/ssmanstudiofull-md.md)] (SSMS) バージョン 18 では、UTF-8 対応の照合順序を UI で選択することができます。
 
- **回避策**:[!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] CTP には回避策はありません。


- **適用対象**:すべての CTP リリース。

## <a name="always-encrypted-with-secure-enclaves"></a>セキュア エンクレーブを使用する Always Encrypted

- **問題およびユーザーへの影響**:高度な計算がパフォーマンス最適化とエラー処理拡張の妨げとなっており、現在は既定で無効になっています。

- **回避策**:高度な計算を有効にするには、`DBCC traceon(127,-1)` を実行します。 詳細については、[高度な計算の有効化](../relational-databases/security/encryption/configure-always-encrypted-enclaves.md#configure-a-secure-enclave)に関する記事をご覧ください。

- **適用対象**:[!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] CTP 3.2、CTP 3.1

## <a name="sql-server-configuration-manager-may-not-start"></a>SQL Server 構成マネージャーが起動しない場合がある

- **問題およびユーザーへの影響**:SQL Server 構成マネージャー (SSCM) は、VCRuntime 140 (VCRUNTIME140.dll) ファイルのないマシンでは起動しません。 SSCM を開始するときに、次のダイアログ ボックスが表示される場合があります。 


  `MMC could not create the snap-in. The snap-in might not have been installed correctly.`

- **回避策**:最新の VC ランタイム 2013 (x86) をインストールします。

  - [詳細](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads)
  - [直接](https://support.microsoft.com/help/4032938/update-for-visual-c-2013-redistributable-package)

- **適用対象**:[!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] CTP 3.1、CTP 3.0、CTP 2.5。

## <a name="always-on-availability-group-kubernetes-operator-not-supported"></a>Always On 可用性グループの Kubernetes 演算子はサポートされていない

- **問題およびユーザーへの影響**:Always On 可用性グループの Kubernetes 演算子は、このリリース候補ではサポートされておらず、RTM では使用できません。 

- **回避策**:なし

- **適用対象**:[!INCLUDE[SQL Server 2019](../includes/sssqlv15-md.md)] リリース候補

## <a name="master-data-service-notification-email-contains-broken-link"></a>マスター データ サービスの通知電子メールに壊れたリンクが含まれる

- **問題およびユーザーへの影響**:マスター データ サービス (MDS) からの通知電子メールには、壊れたリンクが含まれています。 そのリンクでは、次のメッセージのようなエラーが返されるページに移動します。

   `The view 'Index' or its master was not found or no view engine supports the searched locations.`

- **回避策**:MDS ポータルを開き、リソースに手動で移動します。

- **適用対象**:SQL Server 2019 リリース候補。

## <a name="machine-learning-services"></a>Machine Learning Services

SQL Server Machine Learning Services での問題については、「[SQL Server Machine Learning Services の既知の問題](../advanced-analytics/known-issues-for-sql-server-machine-learning-services.md)」を参照してください。

[!INCLUDE[get-help-options-msft-only](../includes/paragraph-content/get-help-options.md)]

![MS_Logo_X-Small](../sql-server/media/ms-logo-x-small.png)
