---
title: オンプレミス データを使用してクロスクラウドをスケーリングするハイブリッド アプリをデプロイする
description: オンプレミス データを使用し、Azure と Azure Stack Hub でクロスクラウドをスケーリングするアプリをデプロイする方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6de35cb55c4c35a2a9927f9ffc2516ccb00cd89f
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477322"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="09518-103">オンプレミス データを使用してクロスクラウドをスケーリングするハイブリッド アプリをデプロイする</span><span class="sxs-lookup"><span data-stu-id="09518-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="09518-104">このソリューション ガイドでは、Azure と Azure Stack Hub の両方にまたがり、1 つのオンプレミス データ ソースを使用するハイブリッド アプリをデプロイする方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="09518-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="09518-105">ハイブリッド クラウド ソリューションを使用することで、プライベート クラウドが持つコンプライアンス面でのメリットとパブリック クラウドが持つスケーラビリティとを融合することができます。</span><span class="sxs-lookup"><span data-stu-id="09518-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="09518-106">また、開発者は、Microsoft デベロッパーのエコシステムを活用し、クラウド環境とオンプレミス環境にそのスキルを活かすこともできます。</span><span class="sxs-lookup"><span data-stu-id="09518-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="09518-107">概要と前提条件</span><span class="sxs-lookup"><span data-stu-id="09518-107">Overview and assumptions</span></span>

<span data-ttu-id="09518-108">このチュートリアルに従うことにより、開発者がパブリック クラウドとプライベート クラウドにまったく同じ Web アプリをデプロイできるようにするワークフローを構築します。</span><span class="sxs-lookup"><span data-stu-id="09518-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="09518-109">このアプリは、プライベート クラウド上にホストされた、インターネットを介したルーティングが不可能なネットワークにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="09518-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="09518-110">これらの Web アプリは監視され、トラフィックが急増すると、トラフィックをパブリック クラウドにリダイレクトするように DNS レコードがプログラムによって書き換えられます。</span><span class="sxs-lookup"><span data-stu-id="09518-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="09518-111">急増前の水準までトラフィックが減少すると、トラフィックは再びプライベート クラウドへルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="09518-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="09518-112">このチュートリアルに含まれるタスクは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="09518-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="09518-113">ハイブリッド接続の SQL Server データベース サーバーをデプロイする。</span><span class="sxs-lookup"><span data-stu-id="09518-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="09518-114">グローバル Azure 内の Web アプリをハイブリッド ネットワークに接続する。</span><span class="sxs-lookup"><span data-stu-id="09518-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="09518-115">クラウド間スケーリング向けに DNS を構成する。</span><span class="sxs-lookup"><span data-stu-id="09518-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="09518-116">クラウド間スケーリング向けに SSL 証明書を構成する。</span><span class="sxs-lookup"><span data-stu-id="09518-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="09518-117">Web アプリを構成してデプロイする。</span><span class="sxs-lookup"><span data-stu-id="09518-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="09518-118">Traffic Manager プロファイルを作成し、クラウド間スケーリング向けに構成する。</span><span class="sxs-lookup"><span data-stu-id="09518-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="09518-119">トラフィックの増加に関して Application Insights の監視とアラートを設定する。</span><span class="sxs-lookup"><span data-stu-id="09518-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="09518-120">グローバル Azure と Azure Stack Hub の間で自動トラフィック切り替えを構成する。</span><span class="sxs-lookup"><span data-stu-id="09518-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="09518-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="09518-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="09518-122">Microsoft Azure Stack Hub は Azure の拡張機能です。</span><span class="sxs-lookup"><span data-stu-id="09518-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="09518-123">Azure Stack Hub により、オンプレミス環境にクラウド コンピューティングの機敏性とイノベーションがもたらされ、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドが可能になります。</span><span class="sxs-lookup"><span data-stu-id="09518-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="09518-124">[ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。</span><span class="sxs-lookup"><span data-stu-id="09518-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="09518-125">これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="09518-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="09518-126">前提条件</span><span class="sxs-lookup"><span data-stu-id="09518-126">Assumptions</span></span>

<span data-ttu-id="09518-127">このチュートリアルは、グローバル Azure と Azure Stack Hub についての基本知識があることを前提にしています。</span><span class="sxs-lookup"><span data-stu-id="09518-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="09518-128">チュートリアルを開始する前に、より詳しい情報を確認しておきたい場合は、以下の記事をお読みください。</span><span class="sxs-lookup"><span data-stu-id="09518-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="09518-129">Azure 入門</span><span class="sxs-lookup"><span data-stu-id="09518-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="09518-130">Azure Stack Hub の主要概念</span><span class="sxs-lookup"><span data-stu-id="09518-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="09518-131">このチュートリアルは、Azure サブスクリプションをお持ちであることも前提としています。</span><span class="sxs-lookup"><span data-stu-id="09518-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="09518-132">サブスクリプションをお持ちでない場合は、開始する前に[無料アカウントを作成](https://azure.microsoft.com/free/)してください。</span><span class="sxs-lookup"><span data-stu-id="09518-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="09518-133">前提条件</span><span class="sxs-lookup"><span data-stu-id="09518-133">Prerequisites</span></span>

<span data-ttu-id="09518-134">このソリューションを開始する前に、次の要件を満たしてください。</span><span class="sxs-lookup"><span data-stu-id="09518-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="09518-135">Azure Stack Development Kit (ASDK) または Azure Stack Hub 統合システムのサブスクリプション。</span><span class="sxs-lookup"><span data-stu-id="09518-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="09518-136">ASDK をデプロイするには、[インストーラーを使用して ASDK をデプロイする](/azure-stack/asdk/asdk-install.md)方法の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="09518-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="09518-137">ご利用の Azure Stack Hub 環境に次のものがインストールされている必要があります。</span><span class="sxs-lookup"><span data-stu-id="09518-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="09518-138">Azure App Service。</span><span class="sxs-lookup"><span data-stu-id="09518-138">The Azure App Service.</span></span> <span data-ttu-id="09518-139">Azure Stack Hub のオペレーターと協力して、Azure App Service をご自分の環境にデプロイし、構成してください。</span><span class="sxs-lookup"><span data-stu-id="09518-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="09518-140">このチュートリアルでは、App Service で専用の worker ロールを少なくとも 1 つ利用できるようにすることが求められます。</span><span class="sxs-lookup"><span data-stu-id="09518-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="09518-141">Windows Server 2016 イメージ。</span><span class="sxs-lookup"><span data-stu-id="09518-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="09518-142">Windows Server 2016 と Microsoft SQL Server イメージ。</span><span class="sxs-lookup"><span data-stu-id="09518-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="09518-143">適切なプランとオファー。</span><span class="sxs-lookup"><span data-stu-id="09518-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="09518-144">Web アプリのドメイン名。</span><span class="sxs-lookup"><span data-stu-id="09518-144">A domain name for your web app.</span></span> <span data-ttu-id="09518-145">ドメイン名を持っていない場合、GoDaddy、Bluehost、InMotion などのドメイン プロバイダーから 1 つ購入できます。</span><span class="sxs-lookup"><span data-stu-id="09518-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="09518-146">信頼のおける証明機関 (LetsEncrypt など) から取得したドメインの SSL 証明書。</span><span class="sxs-lookup"><span data-stu-id="09518-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="09518-147">SQL Server データベースと通信し、Application Insights をサポートする Web アプリ。</span><span class="sxs-lookup"><span data-stu-id="09518-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="09518-148">GitHub から [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) サンプル アプリをダウンロードできます。</span><span class="sxs-lookup"><span data-stu-id="09518-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="09518-149">Azure 仮想ネットワークと Azure Stack Hub 仮想ネットワークの間のハイブリッド ネットワーク。</span><span class="sxs-lookup"><span data-stu-id="09518-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="09518-150">詳細な手順については、[Azure と Azure Stack Hub を使用したハイブリッド クラウド接続の構成](solution-deployment-guide-connectivity.md)に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="09518-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="09518-151">Azure Stack Hub にプライベート ビルド エージェントが存在する、継続的インテグレーション/継続的デプロイ (CI/CD) のハイブリッド パイプライン。</span><span class="sxs-lookup"><span data-stu-id="09518-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="09518-152">詳細な手順については、「[Azure および Azure Stack Hub アプリケーションのハイブリッド クラウド ID を構成する](solution-deployment-guide-identity.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="09518-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="09518-153">ハイブリッド接続の SQL Server データベース サーバーをデプロイする</span><span class="sxs-lookup"><span data-stu-id="09518-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="09518-154">Azure Stack Hub ユーザー ポータルにサインインします。</span><span class="sxs-lookup"><span data-stu-id="09518-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="09518-155">**[ダッシュボード]** の **[Marketplace]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Azure Stack Hub Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="09518-157">**[Marketplace]** で **[Compute]\(計算\)** を選択し、 **[More]\(その他\)** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="09518-158">**[More]\(その他\)** から **[Free SQL Server License: SQL Server 2017 Developer on Windows Server]** イメージを選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Azure Stack Hub ユーザー ポータルで仮想マシン イメージを選択する](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="09518-160">**[Free SQL Server License: SQL Server 2017 Developer on Windows Server]** で、 **[作成]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="09518-161">**[基本] の [基本設定の構成]** で、仮想マシン (VM) の **[名前]** 、SQL Server SA の **[ユーザー名]** 、SA の **[パスワード]** を入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="09518-162">**[サブスクリプション]** ドロップダウン リストから、デプロイ先のサブスクリプションを選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="09518-163">**[リソース グループ]** では **[Choose existing]\(既存の選択\)** を使用し、Azure Stack Hub Web アプリと同じリソース グループに VM を配置します。</span><span class="sxs-lookup"><span data-stu-id="09518-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Azure Stack Hub ユーザー ポータルで VM の基本設定を構成する](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="09518-165">**[サイズ]** で VM のサイズを選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="09518-166">このチュートリアルでは、A2_Standard または DS2_V2_Standard をお勧めします。</span><span class="sxs-lookup"><span data-stu-id="09518-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="09518-167">**[設定] の [オプション機能の構成]** で、次の設定を構成します。</span><span class="sxs-lookup"><span data-stu-id="09518-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="09518-168">**ストレージ アカウント**: 新しいアカウントが必要な場合は、作成します。</span><span class="sxs-lookup"><span data-stu-id="09518-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="09518-169">**仮想ネットワーク**:</span><span class="sxs-lookup"><span data-stu-id="09518-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="09518-170">SQL Server VM が VPN ゲートウェイと同じ仮想ネットワーク上にデプロイされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="09518-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="09518-171">**[パブリック IP アドレス]** : 既定の設定を使用します。</span><span class="sxs-lookup"><span data-stu-id="09518-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="09518-172">**[ネットワーク セキュリティ グループ]** : (NSG)。</span><span class="sxs-lookup"><span data-stu-id="09518-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="09518-173">新しい NSG を作成します。</span><span class="sxs-lookup"><span data-stu-id="09518-173">Create a new NSG.</span></span>
   - <span data-ttu-id="09518-174">**[拡張機能] と [監視]** :既定の設定のままにします。</span><span class="sxs-lookup"><span data-stu-id="09518-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="09518-175">**[診断ストレージ アカウント]** :新しいアカウントが必要な場合は、作成します。</span><span class="sxs-lookup"><span data-stu-id="09518-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="09518-176">**[OK]** を選択して構成を保存します。</span><span class="sxs-lookup"><span data-stu-id="09518-176">Select **OK** to save your configuration.</span></span>

     ![Azure Stack Hub ユーザー ポータルでオプションの VM 機能を構成する](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="09518-178">**[SQL Server の設定]** で、次の設定を構成します。</span><span class="sxs-lookup"><span data-stu-id="09518-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="09518-179">**[SQL 接続]** で **[パブリック (インターネット)]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="09518-180">**[ポート]** は、既定値 (**1433**) のままにします。</span><span class="sxs-lookup"><span data-stu-id="09518-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="09518-181">**[SQL 認証]** には **[有効]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="09518-182">SQL 認証を有効にすると、 **[基本]** で構成した "SQLAdmin" の情報が自動設定されます。</span><span class="sxs-lookup"><span data-stu-id="09518-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="09518-183">その他の設定は、既定値のままにしてください。</span><span class="sxs-lookup"><span data-stu-id="09518-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="09518-184">**[OK]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-184">Select **OK**.</span></span>

     ![Azure Stack Hub ユーザー ポータルで SQL Server 設定を構成する](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="09518-186">**[概要]** で VM の構成を確認し、 **[OK]** を選択してデプロイを開始します。</span><span class="sxs-lookup"><span data-stu-id="09518-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Azure Stack Hub ユーザー ポータルの構成の概要](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="09518-188">新しい VM の作成には時間がかかります。</span><span class="sxs-lookup"><span data-stu-id="09518-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="09518-189">VM の状態は、 **[仮想マシン]** で確認できます。</span><span class="sxs-lookup"><span data-stu-id="09518-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Azure Stack Hub ユーザー ポータルの仮想マシンの状態](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="09518-191">Azure と Azure Stack Hub に Web アプリを作成する</span><span class="sxs-lookup"><span data-stu-id="09518-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="09518-192">Azure App Service は、Web アプリの実行と管理を簡単にします。</span><span class="sxs-lookup"><span data-stu-id="09518-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="09518-193">Azure Stack Hub は Azure と一貫性があるため、App Service はどちらの環境でも実行できます。</span><span class="sxs-lookup"><span data-stu-id="09518-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="09518-194">App Service を使用し、アプリをホストします。</span><span class="sxs-lookup"><span data-stu-id="09518-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="09518-195">Web アプリを作成する</span><span class="sxs-lookup"><span data-stu-id="09518-195">Create web apps</span></span>

1. <span data-ttu-id="09518-196">「[Azure で App Service プランを管理する](/azure/app-service/app-service-plan-manage#create-an-app-service-plan)」の手順に従って、Azure で Web アプリを作成します。</span><span class="sxs-lookup"><span data-stu-id="09518-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="09518-197">Web アプリは、ご利用のハイブリッド ネットワークと同じサブスクリプションおよびリソース グループに配置してください。</span><span class="sxs-lookup"><span data-stu-id="09518-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="09518-198">前の手順 (1) を Azure Stack Hub でも行います。</span><span class="sxs-lookup"><span data-stu-id="09518-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="09518-199">Azure Stack Hub 用のルートを追加する</span><span class="sxs-lookup"><span data-stu-id="09518-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="09518-200">Azure Stack Hub 上の App Service は、ユーザーがアプリにアクセスできるよう、パブリック インターネットからルーティングできなければなりません。</span><span class="sxs-lookup"><span data-stu-id="09518-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="09518-201">Azure Stack Hub がインターネットからアクセスできる場合、Azure Stack Hub Web アプリの公開 IP アドレスまたは URL をメモします。</span><span class="sxs-lookup"><span data-stu-id="09518-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="09518-202">ASDK を使用している場合は、[静的 NAT のマッピングを構成](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal)することで、仮想環境の外部に App Service を公開することができます。</span><span class="sxs-lookup"><span data-stu-id="09518-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="09518-203">Azure 内の Web アプリをハイブリッド ネットワークに接続する</span><span class="sxs-lookup"><span data-stu-id="09518-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="09518-204">Azure の Web フロントエンドと Azure Stack Hub の SQL Server データベースを接続するには、Azure と Azure Stack Hub の間のハイブリッド ネットワークに Web アプリを接続する必要があります。</span><span class="sxs-lookup"><span data-stu-id="09518-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="09518-205">接続を有効にするためには、次の作業が必要となります。</span><span class="sxs-lookup"><span data-stu-id="09518-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="09518-206">ポイント対サイト接続を構成します。</span><span class="sxs-lookup"><span data-stu-id="09518-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="09518-207">Web アプリを構成します。</span><span class="sxs-lookup"><span data-stu-id="09518-207">Configure the web app.</span></span>
- <span data-ttu-id="09518-208">Azure Stack Hub 内のローカル ネットワーク ゲートウェイに変更を加える</span><span class="sxs-lookup"><span data-stu-id="09518-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="09518-209">ポイント対サイト接続のために Azure 仮想ネットワークを構成する</span><span class="sxs-lookup"><span data-stu-id="09518-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="09518-210">Azure App Service と統合するためには、ハイブリッド ネットワークの Azure 側にある仮想ネットワーク ゲートウェイでポイント対サイト接続を許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="09518-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="09518-211">Azure で仮想ネットワーク ゲートウェイ ページに移動します。</span><span class="sxs-lookup"><span data-stu-id="09518-211">In Azure, go to the virtual network gateway page.</span></span> <span data-ttu-id="09518-212">**[設定]** で、 **[ポイント対サイトの構成]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Azure 仮想ネットワーク ゲートウェイのポイント対サイト オプション](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="09518-214">**[今すぐ構成]** を選択し、ポイント対サイトを構成します。</span><span class="sxs-lookup"><span data-stu-id="09518-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Azure 仮想ネットワーク ゲートウェイでポイント対サイトの構成を開始する](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="09518-216">**[ポイント対サイト]** 構成ページで、使用するプライベート IP アドレス範囲を **[アドレス プール]** に入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="09518-217">指定する範囲が、ハイブリッド ネットワークのグローバル Azure コンポーネントまたは Azure Stack Hub コンポーネントのサブネットによって既に使用されているアドレス範囲と重複しないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="09518-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="09518-218">**[トンネルの種類]** の **[IKEv2 VPN]** チェック ボックスをオフにします。</span><span class="sxs-lookup"><span data-stu-id="09518-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="09518-219">**[保存]** を選択して、ポイント対サイトの構成を完了します。</span><span class="sxs-lookup"><span data-stu-id="09518-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Azure 仮想ネットワーク ゲートウェイのポイント対サイト設定](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="09518-221">Azure App Service アプリとハイブリッド ネットワークを統合する</span><span class="sxs-lookup"><span data-stu-id="09518-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="09518-222">Azure VNet にアプリを接続するには、「[ゲートウェイが必要な Vnet 統合](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)」の指示に従ってください。</span><span class="sxs-lookup"><span data-stu-id="09518-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="09518-223">Web アプリをホストしている App Service プランの **[設定]** に移動します。</span><span class="sxs-lookup"><span data-stu-id="09518-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="09518-224">**[設定]** の **[ネットワーク]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-224">In **Settings**, select **Networking**.</span></span>

    ![App Service プランのネットワークを構成する](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="09518-226">**[VNET 統合]** の **[管理するにはここをクリック]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![App Service プランの VNET 統合を管理する](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="09518-228">構成する VNET を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="09518-229">**[IP アドレスが VNET にルーティングされました]** で、Azure VNet、Azure Stack Hub VNet、ポイント対サイトのアドレス空間に使用する IP アドレス範囲を入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="09518-230">**[保存]** を選択し、これらの設定を確認して保存します。</span><span class="sxs-lookup"><span data-stu-id="09518-230">Select **Save** to validate and save these settings.</span></span>

    ![Virtual Network 統合でルーティングする IP アドレス範囲](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="09518-232">App Service と Azure VNet の統合方法の詳細については、「[アプリを Azure 仮想ネットワークに統合する](/azure/app-service/web-sites-integrate-with-vnet)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="09518-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="09518-233">Azure Stack Hub 仮想ネットワークを構成する</span><span class="sxs-lookup"><span data-stu-id="09518-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="09518-234">App Service のポイント対サイト アドレス範囲からのトラフィックをルーティングするために、Azure Stack Hub 仮想ネットワーク内のローカル ネットワーク ゲートウェイを構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="09518-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="09518-235">Azure Stack Hub で **[ローカル ネットワーク ゲートウェイ]** に移動します。</span><span class="sxs-lookup"><span data-stu-id="09518-235">In Azure Stack Hub, go to **Local network gateway**.</span></span> <span data-ttu-id="09518-236">**[設定]** で **[構成]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-236">Under **Settings**, select **Configuration**.</span></span>

    ![Azure Stack Hub ローカル ネットワーク ゲートウェイのゲートウェイ構成オプション](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="09518-238">**[アドレス空間]** に、Azure の仮想ネットワーク ゲートウェイのポイント対サイト アドレス範囲を入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Azure Stack Hub ローカル ネットワーク ゲートウェイのポイント対サイト アドレス空間](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="09518-240">**[保存]** を選択し、構成を検証して保存します。</span><span class="sxs-lookup"><span data-stu-id="09518-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="09518-241">クラウド間スケーリング向けに DNS を構成する</span><span class="sxs-lookup"><span data-stu-id="09518-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="09518-242">クラウド間アプリ向けに DNS を適切に構成することで、ユーザーはグローバル Azure と Azure Stack Hub の Web アプリ インスタンスにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="09518-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="09518-243">また、このチュートリアルの DNS 構成を使えば、負荷が増減したときに Azure Traffic Manager でトラフィックをルーティングすることも可能です。</span><span class="sxs-lookup"><span data-stu-id="09518-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="09518-244">App Service ドメインが機能しないため、このチュートリアルでは Azure DNS を使用して DNS を管理します。</span><span class="sxs-lookup"><span data-stu-id="09518-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="09518-245">サブドメインを作成する</span><span class="sxs-lookup"><span data-stu-id="09518-245">Create subdomains</span></span>

<span data-ttu-id="09518-246">Traffic Manager は DNS の CNAME に依存しているため、エンドポイントに対して適切にトラフィックをルーティングするためには、サブドメインが必要となります。</span><span class="sxs-lookup"><span data-stu-id="09518-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="09518-247">DNS レコードとドメイン マッピングの詳細については、「[Traffic Manager を使用したドメインのマップ](/azure/app-service/web-sites-traffic-manager-custom-domain-name)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="09518-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="09518-248">Azure エンドポイントについては、Web アプリにアクセスするためにユーザーが使用できるサブドメインを作成します。</span><span class="sxs-lookup"><span data-stu-id="09518-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="09518-249">このチュートリアルでは、**app.northwind.com** を使用できますが、この値はご自身のドメインに合わせてカスタマイズする必要があります。</span><span class="sxs-lookup"><span data-stu-id="09518-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="09518-250">また、Azure Stack Hub エンドポイントについても、A レコードを使用してサブドメインを作成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="09518-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="09518-251">**azurestack.northwind.com** を使用できます。</span><span class="sxs-lookup"><span data-stu-id="09518-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="09518-252">Azure でカスタム ドメインを構成する</span><span class="sxs-lookup"><span data-stu-id="09518-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="09518-253">[Azure App Service に CNAME をマップ](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)して、**app.northwind.com** ホスト名を Azure Web アプリに追加します。</span><span class="sxs-lookup"><span data-stu-id="09518-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="09518-254">Azure Stack Hub でカスタム ドメインを構成する</span><span class="sxs-lookup"><span data-stu-id="09518-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="09518-255">[Azure App Service に A レコードをマップ](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record)して、ホスト名 **azurestack.northwind.com** を Azure Stack Hub Web アプリに追加します。</span><span class="sxs-lookup"><span data-stu-id="09518-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="09518-256">App Service アプリには、インターネット ルーティング可能な IP アドレスを使用します。</span><span class="sxs-lookup"><span data-stu-id="09518-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="09518-257">[Azure App Service に CNAME をマップ](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)して、ホスト名 **app.northwind.com** を Azure Stack Hub Web アプリに追加します。</span><span class="sxs-lookup"><span data-stu-id="09518-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="09518-258">CNAME のターゲットとして、前の手順 (1) で構成したホスト名を使用してください。</span><span class="sxs-lookup"><span data-stu-id="09518-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="09518-259">クラウド間スケーリング向けに SSL 証明書を構成する</span><span class="sxs-lookup"><span data-stu-id="09518-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="09518-260">Web アプリによって収集される機密データを移動中および SQL データベースで保管されている間、セキュリティで保護することが重要です。</span><span class="sxs-lookup"><span data-stu-id="09518-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="09518-261">すべての受信トラフィックについて SSL 証明書を使用するように、Azure の Web アプリと Azure Stack Hub の Web アプリを構成します。</span><span class="sxs-lookup"><span data-stu-id="09518-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="09518-262">Azure と Azure Stack Hub に SSL を追加する</span><span class="sxs-lookup"><span data-stu-id="09518-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="09518-263">Azure に SSL を追加するには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="09518-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="09518-264">作成したサブドメインに対し、取得した SSL 証明書が有効であることを確認します </span><span class="sxs-lookup"><span data-stu-id="09518-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="09518-265">(ワイルドカード証明書を使用してもかまいません)。</span><span class="sxs-lookup"><span data-stu-id="09518-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="09518-266">Azure で、[Azure Web Apps に既存のカスタム SSL 証明書をバインドする](/azure/app-service/app-service-web-tutorial-custom-ssl)方法に関する記事の「**Web アプリの準備**」と **SSL 証明書のバインド**に関するセクションの指示に従います。</span><span class="sxs-lookup"><span data-stu-id="09518-266">In Azure, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="09518-267">**[SSL の種類]** として **[SNI ベースの SSL]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="09518-268">すべてのトラフィックを HTTP ポートにリダイレクトします。</span><span class="sxs-lookup"><span data-stu-id="09518-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="09518-269">[Azure Web Apps への既存のカスタム SSL 証明書のバインド](/azure/app-service/app-service-web-tutorial-custom-ssl)に関する記事のセクション「**HTTPS の適用**」の手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="09518-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="09518-270">Azure Stack Hub に SSL を追加するには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="09518-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="09518-271">Azure で使用した手順 1. から手順 3. を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="09518-271">Repeat steps 1-3 that you used for Azure.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="09518-272">Web アプリを構成し、デプロイする</span><span class="sxs-lookup"><span data-stu-id="09518-272">Configure and deploy the web app</span></span>

<span data-ttu-id="09518-273">テレメトリを正しい Application Insights インスタンスに報告するようにアプリ コードを構成し、正しい接続文字列で Web アプリを構成します。</span><span class="sxs-lookup"><span data-stu-id="09518-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="09518-274">Application Insights の詳細については、「[Application Insights とは何か?](/azure/application-insights/app-insights-overview)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="09518-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="09518-275">Application Insights を追加する</span><span class="sxs-lookup"><span data-stu-id="09518-275">Add Application Insights</span></span>

1. <span data-ttu-id="09518-276">Microsoft Visual Studio で Web アプリを開きます。</span><span class="sxs-lookup"><span data-stu-id="09518-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="09518-277">プロジェクトに [Application Insights を追加](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications)し、Web トラフィックが増減したときのアラートを生成するために Application Insights によって使用されるテレメトリが転送されるようにします。</span><span class="sxs-lookup"><span data-stu-id="09518-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="09518-278">動的接続文字列を構成する</span><span class="sxs-lookup"><span data-stu-id="09518-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="09518-279">Web アプリの各インスタンスでは、異なる方法を使用して SQL データベースに接続します。</span><span class="sxs-lookup"><span data-stu-id="09518-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="09518-280">Azure のアプリでは SQL Server VM のプライベート IP アドレスが使用され、Azure Stack Hub のアプリでは SQL Server VM のパブリック IP アドレスが使用されます。</span><span class="sxs-lookup"><span data-stu-id="09518-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="09518-281">Azure Stack Hub 統合システムでは、パブリック IP アドレスをインターネット ルーティング可能にしないでください。</span><span class="sxs-lookup"><span data-stu-id="09518-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="09518-282">ASDK では、パブリック IP アドレスは ASDK の外部にルーティングできません。</span><span class="sxs-lookup"><span data-stu-id="09518-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="09518-283">App Service 環境変数を使用し、アプリの各インスタンスに異なる接続文字列を渡すことができます。</span><span class="sxs-lookup"><span data-stu-id="09518-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="09518-284">Visual Studio でアプリを開きます。</span><span class="sxs-lookup"><span data-stu-id="09518-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="09518-285">Startup.cs を開いて、次のコード ブロックを見つけます。</span><span class="sxs-lookup"><span data-stu-id="09518-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="09518-286">前のコード ブロックを次のコードに置き換えます。このコードでは、*appsettings.json* ファイルで定義されている接続文字列が使用されます。</span><span class="sxs-lookup"><span data-stu-id="09518-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="09518-287">App Service アプリ設定を構成する</span><span class="sxs-lookup"><span data-stu-id="09518-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="09518-288">Azure と Azure Stack Hub 用の接続文字列を作成します。</span><span class="sxs-lookup"><span data-stu-id="09518-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="09518-289">IP アドレス以外は、同じ文字列を使用してください。</span><span class="sxs-lookup"><span data-stu-id="09518-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="09518-290">Azure と Azure Stack Hub で、Web アプリの[アプリ設定として](/azure/app-service/web-sites-configure)適切な接続文字列を追加します。そのとき、名前のプレフィックスとして `SQLCONNSTR\_` を使用します。</span><span class="sxs-lookup"><span data-stu-id="09518-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="09518-291">Web アプリ設定を**保存**し、アプリを再起動します。</span><span class="sxs-lookup"><span data-stu-id="09518-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="09518-292">グローバル Azure で自動スケーリングを有効にする</span><span class="sxs-lookup"><span data-stu-id="09518-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="09518-293">App Service 環境で Web アプリを作成するとき、1 つのインスタンスから始めます。</span><span class="sxs-lookup"><span data-stu-id="09518-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="09518-294">自動的にスケールアウトしてインスタンスを追加することで、アプリ用のコンピューティング リソースを増やすことができます。</span><span class="sxs-lookup"><span data-stu-id="09518-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="09518-295">同様に、自動的にスケールインして、アプリに必要なインスタンスの数を減らすことができます。</span><span class="sxs-lookup"><span data-stu-id="09518-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="09518-296">スケールアウトとスケールインを構成するには、App Service プランが必要です。</span><span class="sxs-lookup"><span data-stu-id="09518-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="09518-297">プランをお持ちでない場合は、作成したうえで次の手順を開始してください。</span><span class="sxs-lookup"><span data-stu-id="09518-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="09518-298">自動スケールアウトを有効にする</span><span class="sxs-lookup"><span data-stu-id="09518-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="09518-299">Azure で、スケールアウトしたいサイトの App Service プランを見つけて、 **[スケールアウト (App Service プラン)]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-299">In Azure, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Azure App Service をスケールアウトする](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="09518-301">**[自動スケールの有効化]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-301">Select **Enable autoscale**.</span></span>

    ![Azure App Service で自動スケーリングを有効にする](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="09518-303">**[自動スケール設定の名前]** に名前を入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="09518-304">**既存**の自動スケール ルールで、 **[メトリックに基づいてスケーリングする]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="09518-305">**[インスタンスの制限]** で、 **[最小]** を 1、 **[最大]** を 10、 **[既定]** を 1 に設定します。</span><span class="sxs-lookup"><span data-stu-id="09518-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Azure App Service で自動スケーリングを構成する](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="09518-307">**[+ ルールの追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="09518-308">**[メトリック ソース]** で **[現在のリソース]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="09518-309">このルールには、次の条件とアクションを使用します。</span><span class="sxs-lookup"><span data-stu-id="09518-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="09518-310">条件</span><span class="sxs-lookup"><span data-stu-id="09518-310">Criteria</span></span>

1. <span data-ttu-id="09518-311">**[時間の集計]** で **[平均]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="09518-312">**[メトリック名]** で **[CPU の割合]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="09518-313">**[演算子]** で **[より大きい]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="09518-314">**[しきい値]** を **50** に設定します。</span><span class="sxs-lookup"><span data-stu-id="09518-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="09518-315">**[期間]** を **10** に設定します。</span><span class="sxs-lookup"><span data-stu-id="09518-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="09518-316">アクション</span><span class="sxs-lookup"><span data-stu-id="09518-316">Action</span></span>

1. <span data-ttu-id="09518-317">**[操作]** で **[カウントを増やす量]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="09518-318">**[インスタンス数]** を **2** に設定します。</span><span class="sxs-lookup"><span data-stu-id="09518-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="09518-319">**[クール ダウン]** を **5** に設定します。</span><span class="sxs-lookup"><span data-stu-id="09518-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="09518-320">**[追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-320">Select **Add**.</span></span>

5. <span data-ttu-id="09518-321">**[+ ルールの追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="09518-322">**[メトリック ソース]** で **[現在のリソース]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="09518-323">現在のリソースには、App Service プランの名前/GUID が表示され、 **[リソースの種類]** ドロップダウン リストと **[リソース]** ドロップダウン リストは利用できません。</span><span class="sxs-lookup"><span data-stu-id="09518-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="09518-324">自動スケールインを有効にする</span><span class="sxs-lookup"><span data-stu-id="09518-324">Enable automatic scale in</span></span>

<span data-ttu-id="09518-325">トラフィックが減ると、Azure Web アプリでは、アクティブ インスタンスの数を自動的に減らし、コストを減らすことができます。</span><span class="sxs-lookup"><span data-stu-id="09518-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="09518-326">このアクションはスケールアウトより消極的であり、アプリ ユーザーへの影響を最小限に抑えます。</span><span class="sxs-lookup"><span data-stu-id="09518-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="09518-327">**[既定]** のスケールアウト条件に移動し、 **[+ ルールの追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="09518-328">このルールには、次の条件とアクションを使用します。</span><span class="sxs-lookup"><span data-stu-id="09518-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="09518-329">条件</span><span class="sxs-lookup"><span data-stu-id="09518-329">Criteria</span></span>

1. <span data-ttu-id="09518-330">**[時間の集計]** で **[平均]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="09518-331">**[メトリック名]** で **[CPU の割合]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="09518-332">**[演算子]** で **[より小さい]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="09518-333">**[しきい値]** を **30** に設定します。</span><span class="sxs-lookup"><span data-stu-id="09518-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="09518-334">**[期間]** を **10** に設定します。</span><span class="sxs-lookup"><span data-stu-id="09518-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="09518-335">アクション</span><span class="sxs-lookup"><span data-stu-id="09518-335">Action</span></span>

1. <span data-ttu-id="09518-336">**[操作]** で **[カウントを減らす量]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="09518-337">**[インスタンス数]** を **1** に設定します。</span><span class="sxs-lookup"><span data-stu-id="09518-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="09518-338">**[クール ダウン]** を **5** に設定します。</span><span class="sxs-lookup"><span data-stu-id="09518-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="09518-339">**[追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="09518-340">Traffic Manager プロファイルを作成し、クラウド間スケーリングを構成する</span><span class="sxs-lookup"><span data-stu-id="09518-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="09518-341">Azure で Traffic Manager プロファイルを作成し、クラウド間スケーリングを有効にするようにエンドポイントを構成します。</span><span class="sxs-lookup"><span data-stu-id="09518-341">Create a Traffic Manager profile in Azure and then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="09518-342">Traffic Manager プロファイルを作成する</span><span class="sxs-lookup"><span data-stu-id="09518-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="09518-343">**[リソースの作成]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="09518-344">**[ネットワーク]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-344">Select **Networking**.</span></span>
3. <span data-ttu-id="09518-345">**[Traffic Manager プロファイル]** を選択し、次の設定を構成します。</span><span class="sxs-lookup"><span data-stu-id="09518-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="09518-346">**[名前]** に、プロファイルの名前を入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="09518-347">この名前は、trafficmanager.net ゾーン内で一意であることが**必要**です。新しい DNS 名を作成するときに使用されます (例: northwindstore.trafficmanager.net)。</span><span class="sxs-lookup"><span data-stu-id="09518-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="09518-348">**[ルーティング方法]** で **[重み付け]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="09518-349">**[サブスクリプション]** で、このプロファイルを作成するサブスクリプションを選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="09518-350">**[リソース グループ]** で、このプロファイルの新しいリソース グループを作成します。</span><span class="sxs-lookup"><span data-stu-id="09518-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="09518-351">**[リソース グループの場所]** で、リソース グループの場所を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="09518-352">これはリソース グループの場所を指定する設定であり、グローバルにデプロイされる Traffic Manager プロファイルには影響しません。</span><span class="sxs-lookup"><span data-stu-id="09518-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="09518-353">**［作成］** を選択します</span><span class="sxs-lookup"><span data-stu-id="09518-353">Select **Create**.</span></span>

    ![Traffic Manager プロファイルを作成する](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="09518-355">Traffic Manager プロファイルのグローバル デプロイが完了すると、その作成先となったリソース グループのリソース一覧にそのプロファイルが表示されます。</span><span class="sxs-lookup"><span data-stu-id="09518-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="09518-356">Traffic Manager エンドポイントの追加</span><span class="sxs-lookup"><span data-stu-id="09518-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="09518-357">作成した Traffic Manager プロファイルを検索します </span><span class="sxs-lookup"><span data-stu-id="09518-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="09518-358">プロファイルのリソース グループに移動した場合は、プロファイルを選択してください。</span><span class="sxs-lookup"><span data-stu-id="09518-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="09518-359">**[Traffic Manager プロファイル]** の **[設定]** で、 **[エンドポイント]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="09518-360">**[追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-360">Select **Add**.</span></span>

4. <span data-ttu-id="09518-361">Azure Stack Hub について、 **[エンドポイントの追加]** で次の設定を使用します。</span><span class="sxs-lookup"><span data-stu-id="09518-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="09518-362">**[Type] (種類)** で、 **[外部エンドポイント]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="09518-363">エンドポイントの **[名前]** を入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="09518-364">**完全修飾ドメイン名 (FQDN) または IP** として、Azure Stack Hub Web アプリの外部 URL を入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="09518-365">**[重み]** は、既定値 (**1**) のままにします。</span><span class="sxs-lookup"><span data-stu-id="09518-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="09518-366">これにより、このエンドポイントが正常な状態である場合、すべてのトラフィックがそのエンドポイントに送信されるようになります。</span><span class="sxs-lookup"><span data-stu-id="09518-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="09518-367">**[無効として追加]** はオフのままにします。</span><span class="sxs-lookup"><span data-stu-id="09518-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="09518-368">**[OK]** を選択して、Azure Stack Hub エンドポイントを保存します。</span><span class="sxs-lookup"><span data-stu-id="09518-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="09518-369">次に、Azure エンドポイントを構成します。</span><span class="sxs-lookup"><span data-stu-id="09518-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="09518-370">**[Traffic Manager プロファイル]** で **[エンドポイント]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="09518-371">**[+追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-371">Select **+Add**.</span></span>
3. <span data-ttu-id="09518-372">Azure について、 **[エンドポイントの追加]** で次の設定を使用します。</span><span class="sxs-lookup"><span data-stu-id="09518-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="09518-373">**[Type] (種類)** で、 **[Azure エンドポイント]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="09518-374">エンドポイントの **[名前]** を入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="09518-375">**[ターゲット リソースの種類]** で、 **[App Service]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="09518-376">**[ターゲット リソース]** で **[アプリ サービスの選択]** を選択し、同じサブスクリプションにある Web アプリの一覧を表示します。</span><span class="sxs-lookup"><span data-stu-id="09518-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="09518-377">**[リソース]** で、最初のエンドポイントとして追加する App Service を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="09518-378">**[重み]** に **2** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="09518-379">この設定により、プライマリ エンドポイントが正常ではない場合や、トリガーされたらトラフィックをルーティングするルール/アラートがある場合、すべてのトラフィックがそのエンドポイントに送信されるようになります。</span><span class="sxs-lookup"><span data-stu-id="09518-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="09518-380">**[無効として追加]** はオフのままにします。</span><span class="sxs-lookup"><span data-stu-id="09518-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="09518-381">**[OK]** を選択して、Azure エンドポイントを保存します。</span><span class="sxs-lookup"><span data-stu-id="09518-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="09518-382">構成したエンドポイントはどちらも、 **[Traffic Manager プロファイル]** で **[エンドポイント]** を選択すると表示されます。</span><span class="sxs-lookup"><span data-stu-id="09518-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="09518-383">次の画面キャプチャの例には、2 つのエンドポイントが、それぞれの状態および構成情報と共に表示されています。</span><span class="sxs-lookup"><span data-stu-id="09518-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Traffic Manager プロファイルのエンドポイント](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a><span data-ttu-id="09518-385">Application Insights の監視とアラートを設定する</span><span class="sxs-lookup"><span data-stu-id="09518-385">Set up Application Insights monitoring and alerting</span></span>

<span data-ttu-id="09518-386">Azure Application Insights を使用すると、アプリを監視し、構成した条件に応じてアラートを送信できます。</span><span class="sxs-lookup"><span data-stu-id="09518-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="09518-387">たとえば、アプリが利用できなくなった、障害が発生した、パフォーマンスの問題が生じたなどの例があります。</span><span class="sxs-lookup"><span data-stu-id="09518-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="09518-388">アラートの作成には、Application Insights のメトリックを使用します。</span><span class="sxs-lookup"><span data-stu-id="09518-388">You'll use Application Insights metrics to create alerts.</span></span> <span data-ttu-id="09518-389">これらのアラートがトリガーされると、Web アプリ インスタンスが自動的に Azure Stack Hub から Azure に切り替わってスケールアウトし、その後、Azure Stack Hub に戻ってスケールインします。</span><span class="sxs-lookup"><span data-stu-id="09518-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="09518-390">メトリックに基づくアラートを作成する</span><span class="sxs-lookup"><span data-stu-id="09518-390">Create an alert from metrics</span></span>

<span data-ttu-id="09518-391">このチュートリアルのリソース グループに移動して Application Insights インスタンスを選択し、 **[Application Insights]** を開きます。</span><span class="sxs-lookup"><span data-stu-id="09518-391">Go to the resource group for this tutorial and then select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="09518-393">このビューを使用してスケールアウト アラートとスケールイン アラートを作成します。</span><span class="sxs-lookup"><span data-stu-id="09518-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="09518-394">スケールアウト アラートを作成する</span><span class="sxs-lookup"><span data-stu-id="09518-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="09518-395">**[構成]** で **[アラート (クラシック)]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="09518-396">**[メトリック アラートの追加 (クラシック)]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="09518-397">**[ルールの追加]** で、次の設定を構成します。</span><span class="sxs-lookup"><span data-stu-id="09518-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="09518-398">**[名前]** に「**Burst into Azure Cloud**」と入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="09518-399">**[説明]** は省略できます。</span><span class="sxs-lookup"><span data-stu-id="09518-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="09518-400">**[ソース]** の **[アラート対象]** で **[メトリック]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="09518-401">**[条件]** で、自分のサブスクリプション、Traffic Manager プロファイルのリソース グループ、リソースに使用する Traffic Manager プロファイルの名前を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="09518-402">**[メトリック]** で **[要求率]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="09518-403">**[条件]** で **[より大きい]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="09518-404">**[しきい値]** に「**2**」を入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="09518-405">**[期間]** で **[直近 5 分]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="09518-406">**[通知手段]** で次のように設定します。</span><span class="sxs-lookup"><span data-stu-id="09518-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="09518-407">**[所有者、共同作成者、閲覧者に電子メールを送信]** のチェック ボックスをオンにします。</span><span class="sxs-lookup"><span data-stu-id="09518-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="09518-408">**[追加する管理者の電子メール]** にメール アドレスを入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="09518-409">メニュー バーで **[保存]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="09518-410">スケールイン アラートを作成する</span><span class="sxs-lookup"><span data-stu-id="09518-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="09518-411">**[構成]** で **[アラート (クラシック)]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="09518-412">**[メトリック アラートの追加 (クラシック)]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="09518-413">**[ルールの追加]** で、次の設定を構成します。</span><span class="sxs-lookup"><span data-stu-id="09518-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="09518-414">**[名前]** に「**Scale back into Azure Stack Hub**」と入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="09518-415">**[説明]** は省略できます。</span><span class="sxs-lookup"><span data-stu-id="09518-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="09518-416">**[ソース]** の **[アラート対象]** で **[メトリック]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="09518-417">**[条件]** で、自分のサブスクリプション、Traffic Manager プロファイルのリソース グループ、リソースに使用する Traffic Manager プロファイルの名前を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="09518-418">**[メトリック]** で **[要求率]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="09518-419">**[条件]** で **[より小さい]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="09518-420">**[しきい値]** に「**2**」を入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="09518-421">**[期間]** で **[直近 5 分]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="09518-422">**[通知手段]** で次のように設定します。</span><span class="sxs-lookup"><span data-stu-id="09518-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="09518-423">**[所有者、共同作成者、閲覧者に電子メールを送信]** のチェック ボックスをオンにします。</span><span class="sxs-lookup"><span data-stu-id="09518-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="09518-424">**[追加する管理者の電子メール]** にメール アドレスを入力します。</span><span class="sxs-lookup"><span data-stu-id="09518-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="09518-425">メニュー バーで **[保存]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="09518-426">次のスクリーンショットには、スケールアウトとスケールインのアラートが示されています。</span><span class="sxs-lookup"><span data-stu-id="09518-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Application Insights のアラート (クラシック)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="09518-428">Azure と Azure Stack Hub の間でトラフィックをリダイレクトする</span><span class="sxs-lookup"><span data-stu-id="09518-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="09518-429">Azure と Azure Stack Hub の間で行われる Web アプリのトラフィックには、手動または自動の切り替えを構成できます。</span><span class="sxs-lookup"><span data-stu-id="09518-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="09518-430">Azure と Azure Stack Hub の間で手動切り替えを構成する</span><span class="sxs-lookup"><span data-stu-id="09518-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="09518-431">Web サイトが構成済みのしきい値に達した場合、アラートが届きます。</span><span class="sxs-lookup"><span data-stu-id="09518-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="09518-432">トラフィックを手動で Azure にリダイレクトするには、次の手順を使用します。</span><span class="sxs-lookup"><span data-stu-id="09518-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="09518-433">Azure portal で、該当する Traffic Manager プロファイルを選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Azure portal の Traffic Manager エンドポイント](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="09518-435">**[エンドポイント]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="09518-436">**[Azure エンドポイント]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="09518-437">**[状態]** で **[有効]** を選択し、 **[保存]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Azure portal で Azure エンドポイントを有効にする](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="09518-439">Traffic Manager プロファイルの **[エンドポイント]** で、 **[外部エンドポイント]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="09518-440">**[状態]** で **[無効]** を選択し、 **[保存]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="09518-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Azure portal で Azure Stack Hub エンドポイントを無効にする](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="09518-442">エンドポイントの構成後、アプリ トラフィックは、Azure Stack Hub Web アプリではなく、Azure スケールアウト Web アプリに送信されます。</span><span class="sxs-lookup"><span data-stu-id="09518-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Azure web アプリのトラフィックで変更されたエンドポイント](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="09518-444">フローを再び Azure Stack Hub に戻すには、前の手順を使用して次の設定を行います。</span><span class="sxs-lookup"><span data-stu-id="09518-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="09518-445">Azure Stack Hub エンドポイントを有効にします。</span><span class="sxs-lookup"><span data-stu-id="09518-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="09518-446">Azure エンドポイントを無効にします。</span><span class="sxs-lookup"><span data-stu-id="09518-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="09518-447">Azure と Azure Stack Hub の間で自動切り替えを構成する</span><span class="sxs-lookup"><span data-stu-id="09518-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="09518-448">Azure Functions によって実現される[サーバーレス](https://azure.microsoft.com/overview/serverless-computing/)環境で対象のアプリが実行されている場合は、Application Insights の監視を使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="09518-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="09518-449">このシナリオでは、関数アプリを呼び出す Webhook を使用するように Application Insights を構成することができます。</span><span class="sxs-lookup"><span data-stu-id="09518-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="09518-450">このアプリでは、アラートに応じてエンドポイントの有効と無効を自動的に切り替えます。</span><span class="sxs-lookup"><span data-stu-id="09518-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="09518-451">次の手順を参考にして、自動トラフィック切り替えを構成してください。</span><span class="sxs-lookup"><span data-stu-id="09518-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="09518-452">Azure 関数アプリを作成します。</span><span class="sxs-lookup"><span data-stu-id="09518-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="09518-453">HTTP によってトリガーされる関数を作成します。</span><span class="sxs-lookup"><span data-stu-id="09518-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="09518-454">Resource Manager、Web Apps、Traffic Manager 用の Azure SDK をインポートします。</span><span class="sxs-lookup"><span data-stu-id="09518-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="09518-455">次の処理を行うコードを作成します。</span><span class="sxs-lookup"><span data-stu-id="09518-455">Develop code to:</span></span>

   - <span data-ttu-id="09518-456">Azure サブスクリプションに対して認証を行う。</span><span class="sxs-lookup"><span data-stu-id="09518-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="09518-457">Traffic Manager のエンドポイントを切り替えるパラメーターを使用して、Azure または Azure Stack Hub にトラフィックを送信する。</span><span class="sxs-lookup"><span data-stu-id="09518-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="09518-458">作成したコードを保存し、Application Insights のアラート ルール設定の **Webhook** セクションに、適切なパラメーターと共に関数アプリの URL を追加します。</span><span class="sxs-lookup"><span data-stu-id="09518-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="09518-459">Application Insights のアラートが発生すると、トラフィックが自動的にリダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="09518-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="09518-460">次のステップ</span><span class="sxs-lookup"><span data-stu-id="09518-460">Next steps</span></span>

- <span data-ttu-id="09518-461">Azure のクラウド パターンの詳細については、「[Cloud Design Pattern (クラウド設計パターン)](/azure/architecture/patterns)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="09518-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
