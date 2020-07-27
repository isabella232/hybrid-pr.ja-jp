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
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="e5f08-103">Azure と Azure Stack Hub に SQL Server 2016 可用性グループをデプロイする</span><span class="sxs-lookup"><span data-stu-id="e5f08-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="e5f08-104">この記事では、2 つの Azure Stack Hub 環境にわたって、非同期 DR (ディザスター リカバリー) サイトと共に、基本的な高可用性 (HA) SQL Server 2016 Enterprise クラスターを自動デプロイする方法を段階的に説明します。</span><span class="sxs-lookup"><span data-stu-id="e5f08-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="e5f08-105">SQL Server 2016 と高可用性の詳細については、「[AlwaysOn 可用性グループ: 高可用性とディザスター リカバリーのソリューション](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e5f08-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="e5f08-106">このソリューションでは、以下を実現するためのサンプル環境を構築します。</span><span class="sxs-lookup"><span data-stu-id="e5f08-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="e5f08-107">2 つの Azure Stack Hub にわたってデプロイを調整する。</span><span class="sxs-lookup"><span data-stu-id="e5f08-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="e5f08-108">Docker を使用し、Azure API プロファイルで依存関係の問題を最小限に抑える。</span><span class="sxs-lookup"><span data-stu-id="e5f08-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="e5f08-109">基本的な高可用性 SQL Server 2016 Enterprise クラスターをディザスター リカバリー サイトと共にデプロイする。</span><span class="sxs-lookup"><span data-stu-id="e5f08-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="e5f08-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="e5f08-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="e5f08-111">Microsoft Azure Stack Hub は Azure の拡張機能です。</span><span class="sxs-lookup"><span data-stu-id="e5f08-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="e5f08-112">Azure Stack Hub はオンプレミス環境にクラウド コンピューティングの機敏性とイノベーションをもたらし、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドを可能にします。</span><span class="sxs-lookup"><span data-stu-id="e5f08-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="e5f08-113">[ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。</span><span class="sxs-lookup"><span data-stu-id="e5f08-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="e5f08-114">これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="e5f08-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="e5f08-115">SQL Server 2016 のアーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="e5f08-115">Architecture for SQL Server 2016</span></span>

![SQL Server 2016 SQL HA Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="e5f08-117">SQL Server 2016 の前提条件</span><span class="sxs-lookup"><span data-stu-id="e5f08-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="e5f08-118">2 つの接続された Azure Stack Hub 統合システム (Azure Stack Hub)。</span><span class="sxs-lookup"><span data-stu-id="e5f08-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="e5f08-119">このデプロイは Azure Stack Development Kit (ASDK) では機能しません。</span><span class="sxs-lookup"><span data-stu-id="e5f08-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="e5f08-120">Azure Stack Hub の詳細については、「[Azure Stack の概要](https://azure.microsoft.com/overview/azure-stack/)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e5f08-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="e5f08-121">各 Azure Stack Hub のテナント サブスクリプション。</span><span class="sxs-lookup"><span data-stu-id="e5f08-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="e5f08-122">**各サブスクリプション ID と Azure Resource Manager エンドポイントを Azure Stack Hub ごとにメモしてください。**</span><span class="sxs-lookup"><span data-stu-id="e5f08-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="e5f08-123">各 Azure Stack Hub のテナント サブスクリプションに対する権限が与えられた Azure Active Directory (Azure AD) サービス プリンシパル。</span><span class="sxs-lookup"><span data-stu-id="e5f08-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="e5f08-124">複数の Azure AD テナントに対して Azure Stack Hub がデプロイされている場合、サービス プリンシパルを 2 つ作成しなければならないことがあります。</span><span class="sxs-lookup"><span data-stu-id="e5f08-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="e5f08-125">Azure Stack Hub のサービス プリンシパルを作成する方法については、[Azure Stack Hub リソースへのアクセスをアプリに提供するサービス プリンシパルを作成する](/azure-stack/user/azure-stack-create-service-principals)方法に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="e5f08-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="e5f08-126">**各サービス プリンシパルのアプリケーション ID、クライアント シークレット、テナント名 (xxxxx.onmicrosoft.com) をメモしてください。**</span><span class="sxs-lookup"><span data-stu-id="e5f08-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="e5f08-127">各 Azure Stack Hub の Marketplace にシンジケート化された SQL Server 2016 Enterprise。</span><span class="sxs-lookup"><span data-stu-id="e5f08-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="e5f08-128">マーケットプレース シンジケーションの詳細については、「[Azure Stack Hub に Marketplace の項目をダウンロードする](/azure-stack/operator/azure-stack-download-azure-marketplace-item)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e5f08-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="e5f08-129">**組織に適切な SQL ライセンスが与えられていることを確認してください。**</span><span class="sxs-lookup"><span data-stu-id="e5f08-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="e5f08-130">ローカル コンピューターにインストールされた [Docker for Windows](https://docs.docker.com/docker-for-windows/)。</span><span class="sxs-lookup"><span data-stu-id="e5f08-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="e5f08-131">Docker イメージを取得する</span><span class="sxs-lookup"><span data-stu-id="e5f08-131">Get the Docker image</span></span>

<span data-ttu-id="e5f08-132">各デプロイの Docker イメージにより、異なる Azure PowerShell バージョン間の依存関係イシューがなくなります。</span><span class="sxs-lookup"><span data-stu-id="e5f08-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="e5f08-133">Docker for Windows で Windows コンテナーが使用されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="e5f08-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="e5f08-134">管理者特権でのコマンド プロンプトで次のスクリプトを実行し、Docker コンテナーとデプロイ スクリプトを取得します。</span><span class="sxs-lookup"><span data-stu-id="e5f08-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="e5f08-135">可用性グループをデプロイする</span><span class="sxs-lookup"><span data-stu-id="e5f08-135">Deploy the availability group</span></span>

1. <span data-ttu-id="e5f08-136">コンテナー イメージが正常にプルされたら、イメージを起動します。</span><span class="sxs-lookup"><span data-stu-id="e5f08-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="e5f08-137">コンテナーが起動すると、コンテナーに管理者特権の PowerShell ターミナルが表示されます。</span><span class="sxs-lookup"><span data-stu-id="e5f08-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="e5f08-138">ディレクトリを変更し、デプロイ スクリプトに移動します。</span><span class="sxs-lookup"><span data-stu-id="e5f08-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="e5f08-139">デプロイを実行します。</span><span class="sxs-lookup"><span data-stu-id="e5f08-139">Run the deployment.</span></span> <span data-ttu-id="e5f08-140">必要とされる箇所で資格情報とリソース名を入力します。</span><span class="sxs-lookup"><span data-stu-id="e5f08-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="e5f08-141">HA は、HA クラスターがデプロイされる Azure Stack Hub を指します。</span><span class="sxs-lookup"><span data-stu-id="e5f08-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="e5f08-142">DR は、DR クラスターがデプロイされる Azure Stack Hub を指します。</span><span class="sxs-lookup"><span data-stu-id="e5f08-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

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

4. <span data-ttu-id="e5f08-143">「`Y`」と入力すると、NuGet プロバイダーのインストールが許可され、API Profile "2018-03-01-hybrid" モジュールのインストールが開始されます。</span><span class="sxs-lookup"><span data-stu-id="e5f08-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="e5f08-144">リソース デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="e5f08-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="e5f08-145">DR リソース デプロイが完了したら、コンテナーを終了します。</span><span class="sxs-lookup"><span data-stu-id="e5f08-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="e5f08-146">各 Azure Stack Hub のポータルでリソースを表示し、デプロイを調べます。</span><span class="sxs-lookup"><span data-stu-id="e5f08-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="e5f08-147">HA 環境で SQL インスタンスの 1 つに接続し、SQL Server Management Studio (SSMS) で可用性グループを調べます。</span><span class="sxs-lookup"><span data-stu-id="e5f08-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="e5f08-149">次のステップ</span><span class="sxs-lookup"><span data-stu-id="e5f08-149">Next steps</span></span>

- <span data-ttu-id="e5f08-150">SQL Server Management Studio を使用して手動でクラスターをフェールオーバーします。</span><span class="sxs-lookup"><span data-stu-id="e5f08-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="e5f08-151">「[Always On 可用性グループの強制手動フェールオーバーの実行 (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)」を参照してください</span><span class="sxs-lookup"><span data-stu-id="e5f08-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="e5f08-152">ハイブリッド クラウド アプリの詳細を確認してください。</span><span class="sxs-lookup"><span data-stu-id="e5f08-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="e5f08-153">[ハイブリッド クラウド ソリューション](https://aka.ms/azsdevtutorials)に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="e5f08-153">See [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="e5f08-154">独自のデータを使用するか、[GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) のこのサンプルに合わせてコードを変更します。</span><span class="sxs-lookup"><span data-stu-id="e5f08-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
