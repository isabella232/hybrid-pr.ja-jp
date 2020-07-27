---
title: Azure と Azure Stack Hub に SQL Server 2016 可用性グループをデプロイする
description: Azure と Azure Stack Hub に SQL Server 2016 可用性グループをデプロイする方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 85b859457b9b54a973c5fc23329b927212b60a07
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477084"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a>Azure と Azure Stack Hub に SQL Server 2016 可用性グループをデプロイする

この記事では、2 つの Azure Stack Hub 環境にわたって、非同期 DR (ディザスター リカバリー) サイトと共に、基本的な高可用性 (HA) SQL Server 2016 Enterprise クラスターを自動デプロイする方法を段階的に説明します。 SQL Server 2016 と高可用性の詳細については、「[AlwaysOn 可用性グループ: 高可用性とディザスター リカバリーのソリューション](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016)」を参照してください。

このソリューションでは、以下を実現するためのサンプル環境を構築します。

> [!div class="checklist"]
> - 2 つの Azure Stack Hub にわたってデプロイを調整する。
> - Docker を使用し、Azure API プロファイルで依存関係の問題を最小限に抑える。
> - 基本的な高可用性 SQL Server 2016 Enterprise クラスターをディザスター リカバリー サイトと共にデプロイする。

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub は Azure の拡張機能です。 Azure Stack Hub はオンプレミス環境にクラウド コンピューティングの機敏性とイノベーションをもたらし、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドを可能にします。  
> 
> [ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。 これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。

## <a name="architecture-for-sql-server-2016"></a>SQL Server 2016 のアーキテクチャ

![SQL Server 2016 SQL HA Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a>SQL Server 2016 の前提条件

- 2 つの接続された Azure Stack Hub 統合システム (Azure Stack Hub)。 このデプロイは Azure Stack Development Kit (ASDK) では機能しません。 Azure Stack Hub の詳細については、「[Azure Stack の概要](https://azure.microsoft.com/overview/azure-stack/)」を参照してください。
- 各 Azure Stack Hub のテナント サブスクリプション。
  - **各サブスクリプション ID と Azure Resource Manager エンドポイントを Azure Stack Hub ごとにメモしてください。**
- 各 Azure Stack Hub のテナント サブスクリプションに対する権限が与えられた Azure Active Directory (Azure AD) サービス プリンシパル。 複数の Azure AD テナントに対して Azure Stack Hub がデプロイされている場合、サービス プリンシパルを 2 つ作成しなければならないことがあります。 Azure Stack Hub のサービス プリンシパルを作成する方法については、[Azure Stack Hub リソースへのアクセスをアプリに提供するサービス プリンシパルを作成する](/azure-stack/user/azure-stack-create-service-principals)方法に関するページを参照してください。
  - **各サービス プリンシパルのアプリケーション ID、クライアント シークレット、テナント名 (xxxxx.onmicrosoft.com) をメモしてください。**
- 各 Azure Stack Hub の Marketplace にシンジケート化された SQL Server 2016 Enterprise。 マーケットプレース シンジケーションの詳細については、「[Azure Stack Hub に Marketplace の項目をダウンロードする](/azure-stack/operator/azure-stack-download-azure-marketplace-item)」を参照してください。
    **組織に適切な SQL ライセンスが与えられていることを確認してください。**
- ローカル コンピューターにインストールされた [Docker for Windows](https://docs.docker.com/docker-for-windows/)。

## <a name="get-the-docker-image"></a>Docker イメージを取得する

各デプロイの Docker イメージにより、異なる Azure PowerShell バージョン間の依存関係イシューがなくなります。

1. Docker for Windows で Windows コンテナーが使用されていることを確認します。
2. 管理者特権でのコマンド プロンプトで次のスクリプトを実行し、Docker コンテナーとデプロイ スクリプトを取得します。

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a>可用性グループをデプロイする

1. コンテナー イメージが正常にプルされたら、イメージを起動します。

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. コンテナーが起動すると、コンテナーに管理者特権の PowerShell ターミナルが表示されます。 ディレクトリを変更し、デプロイ スクリプトに移動します。

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. デプロイを実行します。 必要とされる箇所で資格情報とリソース名を入力します。 HA は、HA クラスターがデプロイされる Azure Stack Hub を指します。 DR は、DR クラスターがデプロイされる Azure Stack Hub を指します。

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
      -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
      -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
      -AADTenantName_HA "hatenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_HA "haresourcegroupname" `
      -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
      -AzureStackSubscriptionId_HA "haSubscriptionId" `
      -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
      -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
      -AADTenantName_DR "drtenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_DR "drresourcegroupname" `
      -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
      -AzureStackSubscriptionId_DR "drSubscriptionId"
      ```

4. 「`Y`」と入力すると、NuGet プロバイダーのインストールが許可され、API Profile "2018-03-01-hybrid" モジュールのインストールが開始されます。

5. リソース デプロイが完了するまで待ちます。

6. DR リソース デプロイが完了したら、コンテナーを終了します。

      ```powershell
      exit
      ```

7. 各 Azure Stack Hub のポータルでリソースを表示し、デプロイを調べます。 HA 環境で SQL インスタンスの 1 つに接続し、SQL Server Management Studio (SSMS) で可用性グループを調べます。

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a>次のステップ

- SQL Server Management Studio を使用して手動でクラスターをフェールオーバーします。 「[Always On 可用性グループの強制手動フェールオーバーの実行 (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)」を参照してください
- ハイブリッド クラウド アプリの詳細を確認してください。 [ハイブリッド クラウド ソリューション](https://aka.ms/azsdevtutorials)に関するページを参照してください。
- 独自のデータを使用するか、[GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) のこのサンプルに合わせてコードを変更します。
