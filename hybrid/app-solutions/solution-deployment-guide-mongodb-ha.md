---
title: 高可用性の MongoDB ソリューションを Azure と Azure Stack Hub にデプロイする
description: 高可用性の MongoDB ソリューションを Azure と Azure Stack Hub にデプロイする方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 624f032def509d8e42d55807d72176e5fce85910
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901509"
---
# <a name="deploy-a-highly-available-mongodb-solution-across-two-azure-stack-hub-environments"></a><span data-ttu-id="41c08-103">高可用性の MongoDB ソリューションを 2 つの Azure Stack Hub 環境にデプロイする</span><span class="sxs-lookup"><span data-stu-id="41c08-103">Deploy a highly available MongoDB solution across two Azure Stack Hub environments</span></span>

<span data-ttu-id="41c08-104">この記事では、2 つの Azure Stack Hub 環境にわたって、DR (ディザスター リカバリー) サイトと共に、基本的な高可用性 (HA) MongoDB クラスターを自動デプロイする方法を段階的に説明します。</span><span class="sxs-lookup"><span data-stu-id="41c08-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="41c08-105">MongoDB と高可用性の詳細については、「[Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/)」(レプリカ セット メンバー) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="41c08-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="41c08-106">このソリューションでは、以下を実現するためのサンプル環境を作成します。</span><span class="sxs-lookup"><span data-stu-id="41c08-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="41c08-107">2 つの Azure Stack Hub にわたってデプロイを調整する。</span><span class="sxs-lookup"><span data-stu-id="41c08-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="41c08-108">Docker を使用し、Azure API プロファイルで依存関係の問題を最小限に抑える。</span><span class="sxs-lookup"><span data-stu-id="41c08-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="41c08-109">基本的な高可用性 MongoDB クラスターをディザスター リカバリー サイトと共にデプロイする。</span><span class="sxs-lookup"><span data-stu-id="41c08-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="41c08-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="41c08-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="41c08-111">Microsoft Azure Stack Hub は Azure の拡張機能です。</span><span class="sxs-lookup"><span data-stu-id="41c08-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="41c08-112">Azure Stack Hub はオンプレミス環境にクラウド コンピューティングの機敏性とイノベーションをもたらし、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドを可能にします。</span><span class="sxs-lookup"><span data-stu-id="41c08-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="41c08-113">[ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。</span><span class="sxs-lookup"><span data-stu-id="41c08-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="41c08-114">これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="41c08-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="41c08-115">Azure Stack Hub 利用時の MongoDB のアーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="41c08-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![Azure Stack Hub の高可用 MongoDB アーキテクチャ](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="41c08-117">Azure Stack Hub 利用時の MongoDB の前提条件</span><span class="sxs-lookup"><span data-stu-id="41c08-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="41c08-118">2 つの接続された Azure Stack Hub 統合システム (Azure Stack Hub)。</span><span class="sxs-lookup"><span data-stu-id="41c08-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="41c08-119">このデプロイは Azure Stack Development Kit (ASDK) では機能しません。</span><span class="sxs-lookup"><span data-stu-id="41c08-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="41c08-120">Azure Stack Hub の詳細については、「[Azure Stack Hub とは](https://azure.microsoft.com/products/azure-stack/hub/)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="41c08-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="41c08-121">各 Azure Stack Hub のテナント サブスクリプション。</span><span class="sxs-lookup"><span data-stu-id="41c08-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="41c08-122">**各サブスクリプション ID と Azure Resource Manager エンドポイントを Azure Stack Hub ごとにメモしてください。**</span><span class="sxs-lookup"><span data-stu-id="41c08-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="41c08-123">各 Azure Stack Hub のテナント サブスクリプションに対する権限が与えられた Azure Active Directory (Azure AD) サービス プリンシパル。</span><span class="sxs-lookup"><span data-stu-id="41c08-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="41c08-124">複数の Azure AD テナントに対して Azure Stack Hub がデプロイされている場合、サービス プリンシパルを 2 つ作成しなければならないことがあります。</span><span class="sxs-lookup"><span data-stu-id="41c08-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="41c08-125">Azure Stack Hub のサービス プリンシパルを作成する方法については、「[アプリ ID を使用して Azure Stack Hub リソースにアクセスする](/azure-stack/user/azure-stack-create-service-principals)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="41c08-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="41c08-126">**各サービス プリンシパルのアプリケーション ID、クライアント シークレット、テナント名 (xxxxx.onmicrosoft.com) をメモしてください。**</span><span class="sxs-lookup"><span data-stu-id="41c08-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="41c08-127">各 Azure Stack Hub の Marketplace にシンジケート化された Ubuntu 16.04。</span><span class="sxs-lookup"><span data-stu-id="41c08-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="41c08-128">マーケットプレース シンジケーションの詳細については、「[Azure Stack Hub に Marketplace の項目をダウンロードする](/azure-stack/operator/azure-stack-download-azure-marketplace-item)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="41c08-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="41c08-129">ローカル コンピューターにインストールされた [Docker for Windows](https://docs.docker.com/docker-for-windows/)。</span><span class="sxs-lookup"><span data-stu-id="41c08-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="41c08-130">Docker イメージを取得する</span><span class="sxs-lookup"><span data-stu-id="41c08-130">Get the Docker image</span></span>

<span data-ttu-id="41c08-131">各デプロイの Docker イメージにより、異なる Azure PowerShell バージョン間の依存関係イシューがなくなります。</span><span class="sxs-lookup"><span data-stu-id="41c08-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="41c08-132">Docker for Windows で Windows コンテナーが使用されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="41c08-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="41c08-133">管理者特権でのコマンド プロンプトで次のコマンドを実行し、Docker コンテナーとデプロイ スクリプトを取得します。</span><span class="sxs-lookup"><span data-stu-id="41c08-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="41c08-134">クラスターのデプロイ</span><span class="sxs-lookup"><span data-stu-id="41c08-134">Deploy the clusters</span></span>

1. <span data-ttu-id="41c08-135">コンテナー イメージが正常にプルされたら、イメージを起動します。</span><span class="sxs-lookup"><span data-stu-id="41c08-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="41c08-136">コンテナーが起動すると、コンテナーに管理者特権の PowerShell ターミナルが表示されます。</span><span class="sxs-lookup"><span data-stu-id="41c08-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="41c08-137">ディレクトリを変更し、デプロイ スクリプトに移動します。</span><span class="sxs-lookup"><span data-stu-id="41c08-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="41c08-138">デプロイを実行します。</span><span class="sxs-lookup"><span data-stu-id="41c08-138">Run the deployment.</span></span> <span data-ttu-id="41c08-139">必要とされる箇所で資格情報とリソース名を入力します。</span><span class="sxs-lookup"><span data-stu-id="41c08-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="41c08-140">HA は、HA クラスターがデプロイされる Azure Stack Hub を指します。</span><span class="sxs-lookup"><span data-stu-id="41c08-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="41c08-141">DR は、DR クラスターがデプロイされる Azure Stack Hub を指します。</span><span class="sxs-lookup"><span data-stu-id="41c08-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="41c08-142">「`Y`」と入力すると、NuGet プロバイダーのインストールが許可され、API Profile "2018-03-01-hybrid" モジュールのインストールが開始されます。</span><span class="sxs-lookup"><span data-stu-id="41c08-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="41c08-143">HA リソースが最初にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="41c08-143">The HA resources will deploy first.</span></span> <span data-ttu-id="41c08-144">デプロイを監視し、完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="41c08-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="41c08-145">HA デプロイ完了のメッセージが表示されたら、HA Azure Stack Hub のポータルでデプロイされたリソースを確認できます。</span><span class="sxs-lookup"><span data-stu-id="41c08-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="41c08-146">続いて DR リソースをデプロイし、クラスターとやりとりする目的で、DR Azure Stack Hub でジャンプ ボックスを有効にするかどうかを決定します。</span><span class="sxs-lookup"><span data-stu-id="41c08-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="41c08-147">DR リソース デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="41c08-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="41c08-148">DR リソースのデプロイが完了したら、コンテナーを終了します。</span><span class="sxs-lookup"><span data-stu-id="41c08-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="41c08-149">次のステップ</span><span class="sxs-lookup"><span data-stu-id="41c08-149">Next steps</span></span>

- <span data-ttu-id="41c08-150">DR Azure Stack Hub でジャンプ ボックス VM を有効にした場合、mongo CLI をインストールすることで、SSH 経由で接続し、MongoDB クラスターとやりとりできます。</span><span class="sxs-lookup"><span data-stu-id="41c08-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="41c08-151">MongoDB とやりとりする方法については、「[mongo Shell](https://docs.mongodb.com/manual/mongo/)」(mongo シェル) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="41c08-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="41c08-152">ハイブリッド クラウド アプリの詳細については、[ハイブリッド クラウド ソリューション](/azure-stack/user/)に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="41c08-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="41c08-153">[GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) のこのサンプルに合わせてコードを変更します。</span><span class="sxs-lookup"><span data-stu-id="41c08-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
