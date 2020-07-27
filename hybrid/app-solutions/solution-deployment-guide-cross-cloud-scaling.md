---
title: Azure と Azure Stack Hub でクロスクラウドをスケーリングするアプリをデプロイする
description: Azure と Azure Stack Hub でクロスクラウドをスケーリングするアプリをデプロイする方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 10cb042e2c6d0c6cb567e14072cd80bc663d686c
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477339"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="78172-103">Azure と Azure Stack Hub を使用してクロスクラウドをスケーリングするアプリをデプロイする</span><span class="sxs-lookup"><span data-stu-id="78172-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="78172-104">Azure Stack Hub でホストされる Web アプリから、Traffic Manager を介した自動スケーリングを備える Azure でホストされる Web アプリに切り替えて、手動トリガー プロセスを提供するクラウド間ソリューションを作成する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="78172-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="78172-105">このプロセスにより、負荷時に柔軟でスケーラブルなクラウド ユーティリティが実現します。</span><span class="sxs-lookup"><span data-stu-id="78172-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="78172-106">このパターンでは、パブリック クラウドでアプリを実行するようにテナントが準備されていない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="78172-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="78172-107">ただし、アプリの需要の急増に対処するためにオンプレミス環境で必要になる容量を維持することは、企業にとって経済的に見合わない場合があります。</span><span class="sxs-lookup"><span data-stu-id="78172-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="78172-108">テナントは、パブリック クラウドの弾力性をオンプレミス ソリューションとともに活用できます。</span><span class="sxs-lookup"><span data-stu-id="78172-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="78172-109">このソリューションでは、以下を実現するためのサンプル環境を構築します。</span><span class="sxs-lookup"><span data-stu-id="78172-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="78172-110">マルチノードの Web アプリを作成します。</span><span class="sxs-lookup"><span data-stu-id="78172-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="78172-111">継続的配置 (CD) プロセスを構成および管理します。</span><span class="sxs-lookup"><span data-stu-id="78172-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="78172-112">Web アプリを Azure Stack Hub に発行します。</span><span class="sxs-lookup"><span data-stu-id="78172-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="78172-113">リリースを作成します。</span><span class="sxs-lookup"><span data-stu-id="78172-113">Create a release.</span></span>
> - <span data-ttu-id="78172-114">デプロイを監視し追跡するについて説明します。</span><span class="sxs-lookup"><span data-stu-id="78172-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="78172-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="78172-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="78172-116">Microsoft Azure Stack Hub は Azure の拡張機能です。</span><span class="sxs-lookup"><span data-stu-id="78172-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="78172-117">Azure Stack Hub はオンプレミス環境にクラウド コンピューティングの機敏性とイノベーションをもたらし、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドを可能にします。</span><span class="sxs-lookup"><span data-stu-id="78172-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="78172-118">[ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。</span><span class="sxs-lookup"><span data-stu-id="78172-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="78172-119">これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="78172-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="78172-120">前提条件</span><span class="sxs-lookup"><span data-stu-id="78172-120">Prerequisites</span></span>

- <span data-ttu-id="78172-121">Azure のサブスクリプション。</span><span class="sxs-lookup"><span data-stu-id="78172-121">Azure subscription.</span></span> <span data-ttu-id="78172-122">必要に応じて、開始前に[無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)を作成します。</span><span class="sxs-lookup"><span data-stu-id="78172-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="78172-123">Azure Stack Hub 統合システムまたは Azure Stack Development Kit (ASDK) のデプロイ。</span><span class="sxs-lookup"><span data-stu-id="78172-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="78172-124">Azure Stack Hub をインストールする手順については、「[ASDK のインストール](/azure-stack/asdk/asdk-install.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="78172-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="78172-125">ASDK デプロイ後の自動化スクリプトについては、[https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack) にアクセスしてください。</span><span class="sxs-lookup"><span data-stu-id="78172-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="78172-126">このインストールが完了するまで数時間かかることがあります。</span><span class="sxs-lookup"><span data-stu-id="78172-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="78172-127">[App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS サービスを Azure Stack Hub にデプロイします。</span><span class="sxs-lookup"><span data-stu-id="78172-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="78172-128">Azure Stack Hub 環境内で[プラン/オファーを作成](/azure-stack/operator/service-plan-offer-subscription-overview.md)します。</span><span class="sxs-lookup"><span data-stu-id="78172-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="78172-129">Azure Stack Hub 環境内で[テナント サブスクリプションを作成](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md)します。</span><span class="sxs-lookup"><span data-stu-id="78172-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="78172-130">テナント サブスクリプション内で Web アプリを作成します。</span><span class="sxs-lookup"><span data-stu-id="78172-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="78172-131">後で使用できるように新しい Web アプリの URL を書き留めておきます。</span><span class="sxs-lookup"><span data-stu-id="78172-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="78172-132">テナント サブスクリプション内で、Azure Pipelines 仮想マシン (VM) をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="78172-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="78172-133">.NET 3.5 がインストールされた Windows Server 2016 VM が必要です。</span><span class="sxs-lookup"><span data-stu-id="78172-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="78172-134">この VM は、プライベート ビルド エージェントとして Azure Stack Hub 上のテナント サブスクリプションに構築されます。</span><span class="sxs-lookup"><span data-stu-id="78172-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="78172-135">[Windows Server 2016 with SQL 2017 VM イメージ](/azure-stack/operator/azure-stack-add-vm-image.md)は、Azure Stack Hub Marketplace で入手できます。</span><span class="sxs-lookup"><span data-stu-id="78172-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="78172-136">このイメージを入手できない場合は、Azure Stack Hub のオペレーターと協力して、環境に追加されていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="78172-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="78172-137">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="78172-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="78172-138">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="78172-138">Scalability</span></span>

<span data-ttu-id="78172-139">クラウド間スケーリングの主要コンポーネントは、パブリック クラウドとオンプレミス クラウド インフラストラクチャ間の迅速なオンデマンド スケーリングを実現する機能であり、一貫性があり信頼性の高いサービスを提供します。</span><span class="sxs-lookup"><span data-stu-id="78172-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="78172-140">可用性</span><span class="sxs-lookup"><span data-stu-id="78172-140">Availability</span></span>

<span data-ttu-id="78172-141">オンプレミス ハードウェア構成およびソフトウェア デプロイを通じて高可用性をもたらすように、ローカルでデプロイされたアプリが構成されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="78172-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="78172-142">管理の容易性</span><span class="sxs-lookup"><span data-stu-id="78172-142">Manageability</span></span>

<span data-ttu-id="78172-143">クラウド間ソリューションは、複数の環境にわたるシームレスな管理と使い慣れたインターフェイスを実現します。</span><span class="sxs-lookup"><span data-stu-id="78172-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="78172-144">クロスプラットフォームの管理には、PowerShell を使用することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="78172-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="78172-145">クラウド間スケーリング</span><span class="sxs-lookup"><span data-stu-id="78172-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="78172-146">カスタム ドメインを取得し DNS を構成する</span><span class="sxs-lookup"><span data-stu-id="78172-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="78172-147">ドメインの DNS ゾーン ファイルを更新します。</span><span class="sxs-lookup"><span data-stu-id="78172-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="78172-148">Azure AD は、カスタム ドメイン名の所有権を確認します。</span><span class="sxs-lookup"><span data-stu-id="78172-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="78172-149">Azure 内の Azure/Office 365/外部 DNS レコードに [Azure DNS](/azure/dns/dns-getstarted-portal) を使用するか、または[別の DNS レジストラー](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)で DNS エントリを追加します。</span><span class="sxs-lookup"><span data-stu-id="78172-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

1. <span data-ttu-id="78172-150">パブリック レジストラーでカスタム ドメインを登録します。</span><span class="sxs-lookup"><span data-stu-id="78172-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="78172-151">ドメインのドメイン名レジストラーにサインインします。</span><span class="sxs-lookup"><span data-stu-id="78172-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="78172-152">DNS の更新を行うには、承認された管理者が必要になることがあります。</span><span class="sxs-lookup"><span data-stu-id="78172-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="78172-153">Azure AD から提供された DNS エントリを追加して、ドメインの DNS ゾーン ファイルを更新します。</span><span class="sxs-lookup"><span data-stu-id="78172-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="78172-154">(DNS エントリはメール ルーティングや Web ホスティングの動作には影響しません)。</span><span class="sxs-lookup"><span data-stu-id="78172-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="78172-155">Azure Stack Hub で既定のマルチノード Web アプリを作成する</span><span class="sxs-lookup"><span data-stu-id="78172-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="78172-156">ハイブリッドの継続的インテグレーションおよび継続的配置 (CI/CD) を設定して、Web アプリを Azure と Azure Stack Hub にデプロイし、両方のクラウドに変更を自動プッシュします。</span><span class="sxs-lookup"><span data-stu-id="78172-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="78172-157">(Windows Server と SQL の) 実行および App Service のデプロイには、適切なイメージがシンジケート化された Azure Stack Hub が必要です。</span><span class="sxs-lookup"><span data-stu-id="78172-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="78172-158">詳細については、App Service のドキュメント「[App Service on Azure Stack Hub のデプロイの前提条件](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="78172-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="78172-159">Azure Repos にコードを追加する</span><span class="sxs-lookup"><span data-stu-id="78172-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="78172-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="78172-160">Azure Repos</span></span>

1. <span data-ttu-id="78172-161">Azure Repos でのプロジェクト作成権限が付与されているアカウントを使用して、Azure Repos にサインインします。</span><span class="sxs-lookup"><span data-stu-id="78172-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="78172-162">ハイブリッド CI/CD は、アプリ コードとインフラストラクチャ コードの両方に適用できます。</span><span class="sxs-lookup"><span data-stu-id="78172-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="78172-163">プライベート クラウド開発とホステッド クラウド開発の両方に、[Azure Resource Manager テンプレート](https://azure.microsoft.com/resources/templates/)を使用します。</span><span class="sxs-lookup"><span data-stu-id="78172-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Azure Repos のプロジェクトに接続する](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="78172-165">既定の Web アプリを作成して開くことで、**リポジトリを複製**します。</span><span class="sxs-lookup"><span data-stu-id="78172-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Azure Web アプリでリポジトリを複製する](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="78172-167">両方のクラウドで App Services の自己完結型 Web アプリ デプロイを作成する</span><span class="sxs-lookup"><span data-stu-id="78172-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="78172-168">**WebApplication.csproj** ファイルを編集します。</span><span class="sxs-lookup"><span data-stu-id="78172-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="78172-169">`Runtimeidentifier` を選択し、`win10-x64` を追加します。</span><span class="sxs-lookup"><span data-stu-id="78172-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="78172-170">(「[自己完結型デプロイ](/dotnet/core/deploying/deploy-with-vs#simpleSelf)」に関するドキュメントを参照してください)。</span><span class="sxs-lookup"><span data-stu-id="78172-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Web アプリ プロジェクト ファイルを編集する](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="78172-172">チーム エクスプローラーを使用して、コードを Azure Repos にチェックインします。</span><span class="sxs-lookup"><span data-stu-id="78172-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="78172-173">アプリ コードが Azure Repos にチェックインされたことを確認します。</span><span class="sxs-lookup"><span data-stu-id="78172-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="78172-174">ビルド定義を作成する</span><span class="sxs-lookup"><span data-stu-id="78172-174">Create the build definition</span></span>

1. <span data-ttu-id="78172-175">Azure Pipelines にサインインし、ビルド定義を作成する機能を確認します。</span><span class="sxs-lookup"><span data-stu-id="78172-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="78172-176">**-r win10-x64** コードを追加します。</span><span class="sxs-lookup"><span data-stu-id="78172-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="78172-177">この追加は、.NET Core を使用して自己完結型のデプロイをトリガーするために必要です。</span><span class="sxs-lookup"><span data-stu-id="78172-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Web アプリにコードを追加する](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="78172-179">ビルドを実行します。</span><span class="sxs-lookup"><span data-stu-id="78172-179">Run the build.</span></span> <span data-ttu-id="78172-180">[自己完結型のデプロイ ビルド](/dotnet/core/deploying/deploy-with-vs#simpleSelf)のプロセスにより、Azure 上と Azure Stack Hub 上で実行される成果物が発行されます。</span><span class="sxs-lookup"><span data-stu-id="78172-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="78172-181">Azure でホストされるエージェントを使用する</span><span class="sxs-lookup"><span data-stu-id="78172-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="78172-182">Web アプリをビルドし、デプロイする場合、Azure Pipelines でホステッド ビルド エージェントを使用すると便利です。</span><span class="sxs-lookup"><span data-stu-id="78172-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="78172-183">メンテナンスやアップグレードは Microsoft Azure によって自動的に実施されるので、中断のない継続的な開発サイクルが可能になります。</span><span class="sxs-lookup"><span data-stu-id="78172-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="78172-184">CD プロセスを管理および構成する</span><span class="sxs-lookup"><span data-stu-id="78172-184">Manage and configure the CD process</span></span>

<span data-ttu-id="78172-185">Azure Pipelines および Azure DevOps Services が提供するパイプラインは自由に構成でき、管理性にも優れ、開発、ステージング、QA、運用など、さまざまな環境へのリリースに使用できます。また、特定のステージで承認を要求することもできます。</span><span class="sxs-lookup"><span data-stu-id="78172-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="78172-186">リリース定義の作成</span><span class="sxs-lookup"><span data-stu-id="78172-186">Create release definition</span></span>

1. <span data-ttu-id="78172-187">Azure DevOps Services の **[ビルドとリリース]** セクションの **[リリース]** タブで **[+]** ボタンを選択して、新しいリリースを追加します。</span><span class="sxs-lookup"><span data-stu-id="78172-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![リリース定義の作成](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="78172-189">Azure App Service Deployment テンプレートを適用します。</span><span class="sxs-lookup"><span data-stu-id="78172-189">Apply the Azure App Service Deployment template.</span></span>

   ![[Azure App Service の配置] テンプレートを適用する](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="78172-191">**[成果物の追加]** で、Azure Cloud ビルド アプリに対して成果物を追加します。</span><span class="sxs-lookup"><span data-stu-id="78172-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Azure Cloud ビルドに成果物を追加する](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="78172-193">[パイプライン] タブで、環境の**フェーズ、タスク** リンクを選択し、Azure のクラウド環境の値を設定します。</span><span class="sxs-lookup"><span data-stu-id="78172-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Azure クラウド環境値を設定する](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="78172-195">**環境名**を設定し、Azure Cloud エンドポイントに対して **Azure サブスクリプション**を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Azure Cloud エンドポイントの Azure サブスクリプションを選択する](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="78172-197">**[App Service の名前]** で、必須の Azure アプリ サービス名を設定します。</span><span class="sxs-lookup"><span data-stu-id="78172-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Azure アプリ サービス名を設定する](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="78172-199">Azure クラウドでホストされる環境の **[エージェント キュー]** に「Hosted VS2017」と入力します。</span><span class="sxs-lookup"><span data-stu-id="78172-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Azure クラウドでホストされる環境のエージェント キューを設定する](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="78172-201">[Azure App Service 配置] メニューで、環境に対して有効な**パッケージまたはフォルダー**を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="78172-202">**[OK]** を選択して、**フォルダーの場所**を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-202">Select **OK** to **folder location**.</span></span>
  
      ![Azure App Service 環境のパッケージまたはフォルダーを選択する](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Azure App Service 環境のパッケージまたはフォルダーを選択する](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="78172-205">すべての変更を保存し、**リリース パイプライン**に戻ります。</span><span class="sxs-lookup"><span data-stu-id="78172-205">Save all changes and go back to **release pipeline**.</span></span>

    ![リリース パイプラインの変更を保存する](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="78172-207">Azure Stack Hub アプリのビルドを選択して、新しい成果物を追加します。</span><span class="sxs-lookup"><span data-stu-id="78172-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Azure Stack Hub アプリの新しい成果物を追加する](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="78172-209">[Azure App Service の配置] を適用し、環境をもう 1 つ追加します。</span><span class="sxs-lookup"><span data-stu-id="78172-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![[Azure App Service の配置] に環境を追加する](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="78172-211">新しい環境に「Azure Stack」という名前を付けます。</span><span class="sxs-lookup"><span data-stu-id="78172-211">Name the new environment "Azure Stack".</span></span>

    ![[Azure App Service の配置] の環境に名前を付ける](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="78172-213">**[タスク]** タブで Azure Stack 環境を見つけます。</span><span class="sxs-lookup"><span data-stu-id="78172-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Azure Stack 環境](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="78172-215">Azure Stack エンドポイントのサブスクリプションを選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Azure Stack エンドポイントのサブスクリプションを選択する](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="78172-217">[App Service の名前] として、Azure Stack Web アプリの名前を設定します。</span><span class="sxs-lookup"><span data-stu-id="78172-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="78172-218">![Azure Stack Web アプリ名を設定する](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="78172-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="78172-219">Azure Stack エージェントを選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-219">Select the Azure Stack agent.</span></span>

    ![Azure Stack エージェントを選択する](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="78172-221">[Azure App Service 配置] セクションで、環境に対して有効な**パッケージまたはフォルダー**を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="78172-222">**[OK]** を選択して、フォルダーの場所を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-222">Select **OK** to folder location.</span></span>

    ![[Azure App Service の配置] のフォルダーを選択する](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![[Azure App Service の配置] のフォルダーを選択する](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="78172-225">[変数] タブで、`VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` という名前の変数を追加し、その値を **true** に設定し、スコープを Azure Stack に設定します。</span><span class="sxs-lookup"><span data-stu-id="78172-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Azure App の配置に変数を追加する](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="78172-227">両方の成果物で**継続的**配置トリガー アイコンを選択し、**継続的**配置トリガーを有効にします。</span><span class="sxs-lookup"><span data-stu-id="78172-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![継続的配置トリガーを選択する](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="78172-229">Azure Stack 環境で**配置前**条件アイコンを選択し、トリガーを**リリース後**に設定します。</span><span class="sxs-lookup"><span data-stu-id="78172-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![配置前条件を選択する](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="78172-231">すべての変更を保存します。</span><span class="sxs-lookup"><span data-stu-id="78172-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="78172-232">タスクの一部の設定は、テンプレートからリリース定義を作成したときに、[環境変数](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables)として自動的に定義されている可能性があります。</span><span class="sxs-lookup"><span data-stu-id="78172-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="78172-233">こうした設定は、タスクの設定では変更できません。これらの設定を編集するには、親環境項目を選択する必要があります。</span><span class="sxs-lookup"><span data-stu-id="78172-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="78172-234">Visual Studio 経由で Azure Stack Hub に発行する</span><span class="sxs-lookup"><span data-stu-id="78172-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="78172-235">エンドポイントを作成すると、Azure DevOps Services ビルドを使用して Azure Service アプリを Azure Stack Hub にデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="78172-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="78172-236">Azure Pipelines がビルド エージェントに接続し、エージェントが Azure Stack Hub に接続します。</span><span class="sxs-lookup"><span data-stu-id="78172-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="78172-237">Azure DevOps Services にサインインし、アプリ設定ページに移動します。</span><span class="sxs-lookup"><span data-stu-id="78172-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="78172-238">**[設定]** の **[セキュリティ]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="78172-239">**[VSTS グループ]** の **[エンドポイント作成者]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="78172-240">**[メンバー]** タブで **[追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="78172-241">**[ユーザーとグループの追加]** にユーザー名を入力し、そのユーザーをユーザー一覧から選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="78172-242">**[変更の保存]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="78172-243">**[VSTS グループ]** の一覧で、 **[エンドポイント管理者]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="78172-244">**[メンバー]** タブで **[追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="78172-245">**[ユーザーとグループの追加]** にユーザー名を入力し、そのユーザーをユーザー一覧から選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="78172-246">**[変更の保存]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-246">Select **Save changes**.</span></span>

<span data-ttu-id="78172-247">これでエンドポイント情報が存在するので、Azure Pipelines から Azure Stack Hub への接続を使用する準備ができました。</span><span class="sxs-lookup"><span data-stu-id="78172-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="78172-248">Azure Stack Hub のビルド エージェントは、Azure Pipelines から命令を受け取った後、Azure Stack Hub との通信のためのエンドポイント情報を伝達します。</span><span class="sxs-lookup"><span data-stu-id="78172-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="78172-249">アプリ ビルドを開発する</span><span class="sxs-lookup"><span data-stu-id="78172-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="78172-250">(Windows Server と SQL の) 実行および App Service のデプロイには、適切なイメージがシンジケート化された Azure Stack Hub が必要です。</span><span class="sxs-lookup"><span data-stu-id="78172-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="78172-251">詳細については、「[App Service on Azure Stack Hub のデプロイの前提条件](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="78172-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="78172-252">Azure Repos から [Azure Resource Manager テンプレート](https://azure.microsoft.com/resources/templates/)を Web アプリ コードのように使用して両方のクラウドに対してデプロイします。</span><span class="sxs-lookup"><span data-stu-id="78172-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="78172-253">Azure Repos プロジェクトにコードを追加する</span><span class="sxs-lookup"><span data-stu-id="78172-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="78172-254">Azure Stack Hub でのプロジェクト作成権限が付与されているアカウントを使用して、Azure Repos にサインインします。</span><span class="sxs-lookup"><span data-stu-id="78172-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="78172-255">既定の Web アプリを作成して開くことで、**リポジトリを複製**します。</span><span class="sxs-lookup"><span data-stu-id="78172-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="78172-256">両方のクラウドで App Services の自己完結型 Web アプリ デプロイを作成する</span><span class="sxs-lookup"><span data-stu-id="78172-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="78172-257">**WebApplication.csproj** ファイルを編集します。`Runtimeidentifier` を選択し、`win10-x64` を追加します。</span><span class="sxs-lookup"><span data-stu-id="78172-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="78172-258">詳細については、「[自己完結型デプロイ](/dotnet/core/deploying/deploy-with-vs#simpleSelf)」に関するドキュメントを参照してください。</span><span class="sxs-lookup"><span data-stu-id="78172-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="78172-259">チーム エクスプローラーを使用して、コードを Azure Repos にチェックインします。</span><span class="sxs-lookup"><span data-stu-id="78172-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="78172-260">アプリ コードが Azure Repos にチェックインされたことを確認します。</span><span class="sxs-lookup"><span data-stu-id="78172-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="78172-261">ビルド定義を作成する</span><span class="sxs-lookup"><span data-stu-id="78172-261">Create the build definition</span></span>

1. <span data-ttu-id="78172-262">ビルド定義を作成できるアカウントで Azure Pipelines にサインインします。</span><span class="sxs-lookup"><span data-stu-id="78172-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="78172-263">プロジェクトの **[Build Web Application]\(Web アプリケーションのビルド\)** ページに移動します。</span><span class="sxs-lookup"><span data-stu-id="78172-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="78172-264">**[引数]** に **-r win10-x64** コードを追加します。</span><span class="sxs-lookup"><span data-stu-id="78172-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="78172-265">この追加は、.NET Core を使用して自己完結型のデプロイをトリガーするために必要です。</span><span class="sxs-lookup"><span data-stu-id="78172-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="78172-266">ビルドを実行します。</span><span class="sxs-lookup"><span data-stu-id="78172-266">Run the build.</span></span> <span data-ttu-id="78172-267">[自己完結型のデプロイ ビルド](/dotnet/core/deploying/deploy-with-vs#simpleSelf)のプロセスにより、Azure および Azure Stack Hub 上で実行できる成果物が発行されます。</span><span class="sxs-lookup"><span data-stu-id="78172-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="78172-268">Azure ホスト ビルド エージェントを使用する</span><span class="sxs-lookup"><span data-stu-id="78172-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="78172-269">Web アプリをビルドし、デプロイする場合、Azure Pipelines でホステッド ビルド エージェントを使用すると便利です。</span><span class="sxs-lookup"><span data-stu-id="78172-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="78172-270">メンテナンスやアップグレードは Microsoft Azure によって自動的に実施されるので、中断のない継続的な開発サイクルが可能になります。</span><span class="sxs-lookup"><span data-stu-id="78172-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="78172-271">継続的配置 (CD) プロセスを構成する</span><span class="sxs-lookup"><span data-stu-id="78172-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="78172-272">Azure Pipelines および Azure DevOps Services が提供するパイプラインは自由に構成でき、管理性にも優れ、開発、ステージング、品質保証 (QA)、運用など、さまざまな環境へのリリースに使用できます。</span><span class="sxs-lookup"><span data-stu-id="78172-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="78172-273">このプロセスの一環として、アプリ ライフ サイクルの特定のステージで承認を要求することもできます。</span><span class="sxs-lookup"><span data-stu-id="78172-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="78172-274">リリース定義の作成</span><span class="sxs-lookup"><span data-stu-id="78172-274">Create release definition</span></span>

<span data-ttu-id="78172-275">アプリ ビルド プロセスの最後の手順は、リリース定義の作成です。</span><span class="sxs-lookup"><span data-stu-id="78172-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="78172-276">このリリース定義を使用してリリースを作成し、ビルドをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="78172-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="78172-277">Azure Pipelines にサインインして、プロジェクトの **[ビルドとリリース]** に移動します。</span><span class="sxs-lookup"><span data-stu-id="78172-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="78172-278">**[リリース]** タブで **[+]** を選択し、 **[リリース定義の作成]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="78172-279">**[テンプレートの選択]** で **[Azure App Service の配置]** を選択し、 **[適用]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="78172-280">**[成果物の追加]** の **[ソース (ビルド定義)]** から Azure Cloud ビルド アプリを選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="78172-281">**[パイプライン]** タブで、**1 フェーズ**、**1 タスク**のリンクを選択し、**環境のタスクを表示**します。</span><span class="sxs-lookup"><span data-stu-id="78172-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="78172-282">**[タスク]** タブで **[環境名]** に「Azure」と入力し、 **[Azure サブスクリプション]** リストから [AzureCloud Traders-Web EP] を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="78172-283">**Azure App Service の名前**として「`northwindtraders`」と入力します。次のキャプチャ画面を参照してください。</span><span class="sxs-lookup"><span data-stu-id="78172-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="78172-284">[エージェント フェーズ] で、 **[エージェント キュー]** リストから **[Hosted VS2017]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="78172-285">**[Azure App Service 配置]** で、環境に対して有効な**パッケージまたはフォルダー**を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="78172-286">**[ファイルまたはフォルダーの選択]** で、**保存先**に対して **[OK]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="78172-287">すべての変更を保存し、 **[パイプライン]** に戻ります。</span><span class="sxs-lookup"><span data-stu-id="78172-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="78172-288">**[パイプライン]** タブで **[成果物の追加]** を選択し、 **[ソース (ビルド定義)]** リストから **[NorthwindCloud Traders-Vessel]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="78172-289">**[テンプレートの選択]** で、もう 1 つ環境を追加します。</span><span class="sxs-lookup"><span data-stu-id="78172-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="78172-290">**[Azure App Service の配置]** を選択し、 **[適用]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="78172-291">**[環境名]** として「`Azure Stack Hub`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="78172-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="78172-292">**[タスク]** タブで、Azure Stack Hub を見つけて選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="78172-293">**[Azure サブスクリプション]** リストから、Azure Stack Hub のエンドポイントに使用する **[AzureStack Traders-Vessel EP]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="78172-294">**[App Service の名前]** として、Azure Stack Hub Web アプリの名前を入力します。</span><span class="sxs-lookup"><span data-stu-id="78172-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="78172-295">**[エージェントの選択]** で、 **[エージェント キュー]** リストから **[AzureStack -bDouglas Fir]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="78172-296">**[Azure App Service 配置]** で、環境に対して有効な**パッケージまたはフォルダー**を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="78172-297">**[ファイルまたはフォルダーの選択]** で、フォルダーの**保存先**に対して **[OK]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="78172-298">**[変数]** タブで、`VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` という名前の変数を見つけます。</span><span class="sxs-lookup"><span data-stu-id="78172-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="78172-299">変数の値を **[true]** に設定し、そのスコープを **[Azure Stack Hub]** に設定します。</span><span class="sxs-lookup"><span data-stu-id="78172-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="78172-300">**[パイプライン]** タブで、NorthwindCloud Traders-Web の成果物に使用する **[継続的配置トリガー]** アイコンを選択し、 **[継続的配置トリガー]** を **[有効]** に設定します。</span><span class="sxs-lookup"><span data-stu-id="78172-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="78172-301">**NorthwindCloud Traders-Vessel** の成果物についても同じ操作を行います。</span><span class="sxs-lookup"><span data-stu-id="78172-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="78172-302">Azure Stack Hub 環境で**配置前**条件アイコンを選択し、トリガーを**リリース後**に設定します。</span><span class="sxs-lookup"><span data-stu-id="78172-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="78172-303">すべての変更を保存します。</span><span class="sxs-lookup"><span data-stu-id="78172-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="78172-304">リリース タスクの一部の設定は、テンプレートからリリース定義を作成したときに、[環境変数](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables)として自動的に定義されます。</span><span class="sxs-lookup"><span data-stu-id="78172-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="78172-305">これらの設定は、タスク設定では変更できませんが、親環境項目で変更できます。</span><span class="sxs-lookup"><span data-stu-id="78172-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="78172-306">リリースを作成する</span><span class="sxs-lookup"><span data-stu-id="78172-306">Create a release</span></span>

1. <span data-ttu-id="78172-307">**[パイプライン]** タブの **[リリース]** リストを開いて、 **[リリースの作成]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="78172-308">リリースの説明を入力し、正しい成果物が選択されていることを確認してから、 **[作成]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="78172-309">しばらくすると、新しいリリースが作成されたことを示すバナーが表示され、そのリリース名がリンクとして表示されます。</span><span class="sxs-lookup"><span data-stu-id="78172-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="78172-310">リンクを選択してリリース概要ページを表示します。</span><span class="sxs-lookup"><span data-stu-id="78172-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="78172-311">リリースの概要ページに、リリースに関する詳細が表示されます。</span><span class="sxs-lookup"><span data-stu-id="78172-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="78172-312">次の "Release-2" のキャプチャ画面では、 **[環境]** セクションに、Azure については **[デプロイ状態]** が "進行中" と表示され、Azure Stack Hub の状態は "成功" と表示されています。</span><span class="sxs-lookup"><span data-stu-id="78172-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="78172-313">Azure 環境のデプロイ状態が "成功" に変わると、リリースの承認準備が完了したことを示すバナーが表示されます。</span><span class="sxs-lookup"><span data-stu-id="78172-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="78172-314">デプロイが保留中か、失敗した場合は、青い **(i)** 情報アイコンが表示されます。</span><span class="sxs-lookup"><span data-stu-id="78172-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="78172-315">このアイコンにマウス ポインターを合わせると、遅延または失敗の理由を示すポップアップが表示されます。</span><span class="sxs-lookup"><span data-stu-id="78172-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="78172-316">リリースの一覧など、その他のビューにも、承認待ちであることを示すアイコンが表示されます。</span><span class="sxs-lookup"><span data-stu-id="78172-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="78172-317">このアイコンのポップアップには、環境の名前のほか、デプロイに関連するさらに詳しい情報が表示されます。</span><span class="sxs-lookup"><span data-stu-id="78172-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="78172-318">管理者は、リリースの全体的な進行状況を見て、どのリリースが承認待ちになっているかを簡単に確認できます。</span><span class="sxs-lookup"><span data-stu-id="78172-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="78172-319">デプロイを監視および追跡する</span><span class="sxs-lookup"><span data-stu-id="78172-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="78172-320">**Release-2** の概要ページで、 **[ログ]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="78172-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="78172-321">デプロイ時には、エージェントからリアルタイムのログがこのページに表示されます。</span><span class="sxs-lookup"><span data-stu-id="78172-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="78172-322">左側のウィンドウには、デプロイに伴う各操作の状態が環境ごとに表示されます。</span><span class="sxs-lookup"><span data-stu-id="78172-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="78172-323">デプロイ前またはデプロイ後の承認の **[アクション]** 列で人アイコンを選択し、デプロイを承認 (または拒否) したユーザーと、そのユーザーが入力したメッセージを表示します。</span><span class="sxs-lookup"><span data-stu-id="78172-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="78172-324">デプロイ完了後、ログ ファイル全体が右側のウィンドウに表示されます。</span><span class="sxs-lookup"><span data-stu-id="78172-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="78172-325">左側のペインでいずれかの **[ステップ]** を選択し、**ジョブの初期化**など、単一ステップのログ ファイルを表示します。</span><span class="sxs-lookup"><span data-stu-id="78172-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="78172-326">ログを個別に表示できることから、デプロイ全体を構成する各要素の追跡とデバッグがしやすくなっています。</span><span class="sxs-lookup"><span data-stu-id="78172-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="78172-327">特定のステップのログ ファイルを**保存**するか、**すべてのログを zip としてダウンロード**します。</span><span class="sxs-lookup"><span data-stu-id="78172-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="78172-328">**[概要]** タブを開くと、そのリリースの概要情報が表示されます。</span><span class="sxs-lookup"><span data-stu-id="78172-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="78172-329">ビルドとそのデプロイ先となった環境の詳細、デプロイ状態など、リリースに関する各種の情報がこのビューに表示されます。</span><span class="sxs-lookup"><span data-stu-id="78172-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="78172-330">環境リンク (**Azure** または **Azure Stack Hub**) を選択すると、特定の環境に対する既存のデプロイと保留中のデプロイについての情報が表示されます。</span><span class="sxs-lookup"><span data-stu-id="78172-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="78172-331">これらのビューを使って簡単に、同じビルドが両方の環境にデプロイされていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="78172-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="78172-332">**デプロイされた運用アプリ**をブラウザーで開きます。</span><span class="sxs-lookup"><span data-stu-id="78172-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="78172-333">たとえば、Azure App Services Web サイトの URL `https://[your-app-name\].azurewebsites.net` を開きます。</span><span class="sxs-lookup"><span data-stu-id="78172-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="78172-334">Azure と Azure Stack Hub を統合することにより、スケーラブルなクラウド間ソリューションが実現する</span><span class="sxs-lookup"><span data-stu-id="78172-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="78172-335">柔軟で堅牢なマルチ クラウド サービスは、データ セキュリティ、バックアップおよび冗長性、一貫性のある迅速な可用性、スケーラブルなストレージおよび分散、地理的準拠ルーティングを提供します。</span><span class="sxs-lookup"><span data-stu-id="78172-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="78172-336">この手動でトリガーされるプロセスにより、ホストされる Web アプリ間での、信頼性が高く、効率的な負荷の切り替えが実現し、重要なデータがすぐに使用できるようになります。</span><span class="sxs-lookup"><span data-stu-id="78172-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="78172-337">次のステップ</span><span class="sxs-lookup"><span data-stu-id="78172-337">Next steps</span></span>

- <span data-ttu-id="78172-338">Azure のクラウド パターンの詳細については、「[Cloud Design Pattern (クラウド設計パターン)](/azure/architecture/patterns)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="78172-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
