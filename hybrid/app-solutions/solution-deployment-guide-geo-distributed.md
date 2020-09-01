---
title: Azure と Azure Stack Hub を利用し、地理的分散アプリでトラフィックを転送する
description: Azure と Azure Stack Hub を使用する地理的に分散されたアプリ ソリューションで、トラフィックを特定のエンドポイントに転送する方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 27d07070becfa902a715b451baae7c81c7e4b46f
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886834"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="0862d-103">Azure と Azure Stack Hub を利用し、地理的分散アプリでトラフィックを転送する</span><span class="sxs-lookup"><span data-stu-id="0862d-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="0862d-104">地理的分散アプリ パターンを使用して、さまざまなメトリックに基づき、特定のエンドポイントにトラフィックを送信する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="0862d-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="0862d-105">地理的ベースのルーティングとエンドポイント構成で Traffic Manager プロファイルを作成すると、リージョンの要件、企業および国際的な規制、およびデータ ニーズに基づいて、情報がエンドポイントにルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="0862d-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="0862d-106">このソリューションでは、以下を実現するためのサンプル環境を構築します。</span><span class="sxs-lookup"><span data-stu-id="0862d-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="0862d-107">地理的分散アプリを作成する</span><span class="sxs-lookup"><span data-stu-id="0862d-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="0862d-108">Traffic Manager を使用して、アプリを対象にします。</span><span class="sxs-lookup"><span data-stu-id="0862d-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="0862d-109">地理的分散アプリ パターンを使用する</span><span class="sxs-lookup"><span data-stu-id="0862d-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="0862d-110">地理的分散パターンを使用すると、アプリは複数のリージョンにまたがります。</span><span class="sxs-lookup"><span data-stu-id="0862d-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="0862d-111">パブリック クラウドを既定として設定できますが、一部のユーザーは、場合によっては、自分のリージョンにデータを残す必要がある場合があります。</span><span class="sxs-lookup"><span data-stu-id="0862d-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="0862d-112">ユーザーをその要件に基づいて最も適したクラウドに送信できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="0862d-113">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="0862d-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="0862d-114">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="0862d-114">Scalability considerations</span></span>

<span data-ttu-id="0862d-115">この記事で作成するソリューションは、スケーラビリティに対応しません。</span><span class="sxs-lookup"><span data-stu-id="0862d-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="0862d-116">ただし、他の Azure とオンプレミスのソリューションと組み合わせて使用すれば、スケーラビリティ要件に対応できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="0862d-117">Traffic Manager を介した自動スケーリングを伴うハイブリッド ソリューションの作成に関する詳細については、「[Azure でクラウド間スケーリング ソリューションを作成する](solution-deployment-guide-cross-cloud-scaling.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0862d-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="0862d-118">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="0862d-118">Availability considerations</span></span>

<span data-ttu-id="0862d-119">スケーラビリティに関する考慮事項と同様に、このソリューションでは可用性を直接扱いません。</span><span class="sxs-lookup"><span data-stu-id="0862d-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="0862d-120">ただし、Azure とオンプレミスのソリューションをこのソリューション内に実装して、関連するすべてのコンポーネントの高可用性を確保できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="0862d-121">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="0862d-121">When to use this pattern</span></span>

- <span data-ttu-id="0862d-122">組織には、リージョンのセキュリティおよび分散に関するカスタム ポリシーが必要な支店が世界中にあります。</span><span class="sxs-lookup"><span data-stu-id="0862d-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="0862d-123">組織の支店それぞれは、従業員、ビジネス、施設のデータをプルしますが、これには地域の規制やタイム ゾーンに従ったレポート アクティビティが必要になります。</span><span class="sxs-lookup"><span data-stu-id="0862d-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="0862d-124">高スケール要件を満たすには、極端に負荷の大きい要件に対応するように、単一のリージョン内、または複数のリージョンにわたって、複数のアプリ デプロイを使用してアプリを水平方向に拡張します。</span><span class="sxs-lookup"><span data-stu-id="0862d-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="0862d-125">トポロジの計画</span><span class="sxs-lookup"><span data-stu-id="0862d-125">Planning the topology</span></span>

<span data-ttu-id="0862d-126">分散アプリのフットプリントを構築する前に、次を知っておくと役立ちます。</span><span class="sxs-lookup"><span data-stu-id="0862d-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="0862d-127">**アプリのカスタム ドメイン:** 顧客がアプリへのアクセスに使用するカスタム ドメイン名が必要です。</span><span class="sxs-lookup"><span data-stu-id="0862d-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="0862d-128">サンプル アプリでは、カスタム ドメイン名は *www\.scalableasedemo.com* です。</span><span class="sxs-lookup"><span data-stu-id="0862d-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="0862d-129">**Traffic Manager ドメイン:** [Azure Traffic Manager プロファイル](/azure/traffic-manager/traffic-manager-manage-profiles)の作成時に、ドメイン名が選択されます。</span><span class="sxs-lookup"><span data-stu-id="0862d-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="0862d-130">この名前は、Traffic Manager が管理するドメイン エントリを登録する際に、*trafficmanager.net* サフィックスと組み合わされます。</span><span class="sxs-lookup"><span data-stu-id="0862d-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="0862d-131">サンプル アプリでは、選択される名前は *scalable-ase-demo*です。</span><span class="sxs-lookup"><span data-stu-id="0862d-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="0862d-132">そのため、Traffic Manager で管理される完全なドメイン名は、*scalable-ase-demo.trafficmanager.net* になります。</span><span class="sxs-lookup"><span data-stu-id="0862d-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="0862d-133">**アプリ フットプリントのスケーリングに関する戦略:** アプリのフットプリントは単一リージョン内の複数の App Service 環境に分散されるのか、リージョンは複数なのか、あるいは両方の手法の混在になるのかを決定します。</span><span class="sxs-lookup"><span data-stu-id="0862d-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="0862d-134">この決定は、顧客のトラフィックが発生する場所に加えて、アプリをサポートするバックエンド インフラストラクチャの他の要素のスケーラビリティに関する期待事項に基づく必要があります。</span><span class="sxs-lookup"><span data-stu-id="0862d-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="0862d-135">たとえば、完全にステートレスなアプリでは、各 Azure リージョンで複数の App Service 環境を組み合わせ、さらに複数の Azure リージョンにデプロイされた App Service 環境を掛け合わせることで、大規模なスケーリングを実施できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="0862d-136">選択できるグローバルな Azure リージョンは 15 以上あるため、顧客はスケーラビリティのきわめて高いアプリ フットプリントを世界規模で構築できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="0862d-137">ここで使用されるサンプル アプリでは、単一の Azure リージョン (米国中南部) に 3 つの App Service 環境が作成されています。</span><span class="sxs-lookup"><span data-stu-id="0862d-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="0862d-138">**App Service 環境の名前付け規則:** 各 App Service 環境には一意の名前が必要です。</span><span class="sxs-lookup"><span data-stu-id="0862d-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="0862d-139">App Service 環境が 1 つか 2 つ以上になるとき、各 App Service 環境を識別する目的で名前付け規則を用意すると便利です。</span><span class="sxs-lookup"><span data-stu-id="0862d-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="0862d-140">ここで使用されるサンプル アプリには、簡単な名前付け規則が使用されました。</span><span class="sxs-lookup"><span data-stu-id="0862d-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="0862d-141">3 つの App Service 環境の名前は *fe1ase*、*fe2ase*、*fe3ase* です。</span><span class="sxs-lookup"><span data-stu-id="0862d-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="0862d-142">**アプリの名前付け規則**:アプリのインスタンスが複数デプロイされるため、デプロイされたアプリの各インスタンスに名前が必要です。</span><span class="sxs-lookup"><span data-stu-id="0862d-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="0862d-143">Power Apps 用の App Service Environment では、同じアプリ名を複数の環境で使用できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="0862d-144">App Service 環境ごとに一意のドメイン サフィックスがあるため、開発者は各環境でまったく同じアプリ名を再利用できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="0862d-145">たとえば、開発者はアプリに *myapp.foo1.p.azurewebsites.net*、*myapp.foo2.p.azurewebsites.net*、*myapp.foo3.p.azurewebsites.net* のように名前を付けることができます。</span><span class="sxs-lookup"><span data-stu-id="0862d-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="0862d-146">ここで使用されるアプリについては、アプリ インスタンスごとに一意の名前が付けられています。</span><span class="sxs-lookup"><span data-stu-id="0862d-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="0862d-147">使用されているアプリ インスタンス名は *webfrontend1*、*webfrontend2*、*webfrontend3* です。</span><span class="sxs-lookup"><span data-stu-id="0862d-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="0862d-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="0862d-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="0862d-149">Microsoft Azure Stack Hub は Azure の拡張機能です。</span><span class="sxs-lookup"><span data-stu-id="0862d-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="0862d-150">Azure Stack Hub により、オンプレミス環境にクラウド コンピューティングの機敏性とイノベーションがもたらされ、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドが可能になります。</span><span class="sxs-lookup"><span data-stu-id="0862d-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="0862d-151">[ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。</span><span class="sxs-lookup"><span data-stu-id="0862d-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="0862d-152">これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="0862d-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="0862d-153">パート 1: 地理的分散アプリを作成する</span><span class="sxs-lookup"><span data-stu-id="0862d-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="0862d-154">このパートでは、Web アプリを作成します。</span><span class="sxs-lookup"><span data-stu-id="0862d-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="0862d-155">Web アプリを作成し、公開します。</span><span class="sxs-lookup"><span data-stu-id="0862d-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="0862d-156">Azure Repos にコードを追加する</span><span class="sxs-lookup"><span data-stu-id="0862d-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="0862d-157">アプリ ビルドを複数のクラウド ターゲットにポイントします。</span><span class="sxs-lookup"><span data-stu-id="0862d-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="0862d-158">CD プロセスを管理し、構成します。</span><span class="sxs-lookup"><span data-stu-id="0862d-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="0862d-159">前提条件</span><span class="sxs-lookup"><span data-stu-id="0862d-159">Prerequisites</span></span>

<span data-ttu-id="0862d-160">Azure サブスクリプションと Azure Stack Hub のインストールが必要です。</span><span class="sxs-lookup"><span data-stu-id="0862d-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="0862d-161">地理的に分散されたアプリの手順</span><span class="sxs-lookup"><span data-stu-id="0862d-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="0862d-162">カスタム ドメインを取得し DNS を構成する</span><span class="sxs-lookup"><span data-stu-id="0862d-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="0862d-163">ドメインの DNS ゾーン ファイルを更新します。</span><span class="sxs-lookup"><span data-stu-id="0862d-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="0862d-164">Azure AD は続いて、カスタム ドメイン名の所有権を確認できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="0862d-165">Azure 内の Azure/Microsoft 365/外部 DNS レコードに [Azure DNS](/azure/dns/dns-getstarted-portal) を使用するか、[別の DNS レジストラー](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)で DNS エントリを追加します。</span><span class="sxs-lookup"><span data-stu-id="0862d-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="0862d-166">パブリック レジストラーでカスタム ドメインを登録します。</span><span class="sxs-lookup"><span data-stu-id="0862d-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="0862d-167">ドメインのドメイン名レジストラーにサインインします。</span><span class="sxs-lookup"><span data-stu-id="0862d-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="0862d-168">DNS の更新を行うには、承認された管理者が必要になることがあります。</span><span class="sxs-lookup"><span data-stu-id="0862d-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="0862d-169">Azure AD から提供された DNS エントリを追加して、ドメインの DNS ゾーン ファイルを更新します。</span><span class="sxs-lookup"><span data-stu-id="0862d-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="0862d-170">メール ルーティングや Web ホスティングなどの動作は変更されません。</span><span class="sxs-lookup"><span data-stu-id="0862d-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="0862d-171">Web アプリを作成し発行する</span><span class="sxs-lookup"><span data-stu-id="0862d-171">Create web apps and publish</span></span>

<span data-ttu-id="0862d-172">ハイブリッドの継続的インテグレーション/継続的デリバリー (CI/CD) を設定し、Web アプリを Azure と Azure Stack Hub にデプロイし、両方のクラウドに変更を自動プッシュします。</span><span class="sxs-lookup"><span data-stu-id="0862d-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="0862d-173">(Windows Server と SQL の) 実行および App Service のデプロイには、適切なイメージがシンジケート化された Azure Stack Hub が必要です。</span><span class="sxs-lookup"><span data-stu-id="0862d-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="0862d-174">詳細については、「[App Service on Azure Stack Hub のデプロイの前提条件](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0862d-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="0862d-175">Azure Repos にコードを追加する</span><span class="sxs-lookup"><span data-stu-id="0862d-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="0862d-176">Azure Repos 上で**プロジェクト作成権限が付与されているアカウント**を使用して、Visual Studio にサインインします。</span><span class="sxs-lookup"><span data-stu-id="0862d-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="0862d-177">CI/CD は、アプリ コードとインフラストラクチャ コードの両方に適用できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="0862d-178">プライベート クラウド開発とホステッド クラウド開発の両方に、[Azure Resource Manager テンプレート](https://azure.microsoft.com/resources/templates/)を使用します。</span><span class="sxs-lookup"><span data-stu-id="0862d-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Visual Studio でプロジェクトに接続する](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="0862d-180">既定の Web アプリを作成して開くことで、**リポジトリを複製**します。</span><span class="sxs-lookup"><span data-stu-id="0862d-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Visual Studio でリポジトリを複製する](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="0862d-182">両方のクラウドで Web アプリ デプロイを作成する</span><span class="sxs-lookup"><span data-stu-id="0862d-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="0862d-183">**WebApplication.csproj** ファイルを編集します。`Runtimeidentifier` を選択し、`win10-x64` を追加します。</span><span class="sxs-lookup"><span data-stu-id="0862d-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="0862d-184">(「[自己完結型デプロイ](/dotnet/core/deploying/deploy-with-vs#simpleSelf)」に関するドキュメントを参照してください)。</span><span class="sxs-lookup"><span data-stu-id="0862d-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Visual Studio で Web アプリ プロジェクト ファイルを編集する](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="0862d-186">チーム エクスプローラーを使用して、**コードを Azure Repos にチェックインします**。</span><span class="sxs-lookup"><span data-stu-id="0862d-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="0862d-187">**アプリケーション コード**が Azure Repos にチェックインされたことを確認します。</span><span class="sxs-lookup"><span data-stu-id="0862d-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="0862d-188">ビルド定義を作成する</span><span class="sxs-lookup"><span data-stu-id="0862d-188">Create the build definition</span></span>

1. <span data-ttu-id="0862d-189">**Azure Pipelines にサインイン**し、ビルド定義を作成する機能を確認します。</span><span class="sxs-lookup"><span data-stu-id="0862d-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="0862d-190">`-r win10-x64` コードを追加します。</span><span class="sxs-lookup"><span data-stu-id="0862d-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="0862d-191">この追加は、.NET Core を使用して自己完結型のデプロイをトリガーするために必要です。</span><span class="sxs-lookup"><span data-stu-id="0862d-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Azure Pipelines でビルド定義にコードを追加する](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="0862d-193">**ビルドを実行します**。</span><span class="sxs-lookup"><span data-stu-id="0862d-193">**Run the build**.</span></span> <span data-ttu-id="0862d-194">[自己完結型のデプロイ ビルド](/dotnet/core/deploying/deploy-with-vs#simpleSelf)のプロセスにより、Azure および Azure Stack Hub 上で実行できる成果物が発行されます。</span><span class="sxs-lookup"><span data-stu-id="0862d-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="0862d-195">Azure ホステッド エージェントを使用する</span><span class="sxs-lookup"><span data-stu-id="0862d-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="0862d-196">Web アプリをビルドおよびデプロイする場合、Azure Pipelines でホステッド エージェントを使用すると便利です。</span><span class="sxs-lookup"><span data-stu-id="0862d-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="0862d-197">Microsoft Azure によってメンテナンスやアップグレードが自動的に実施され、開発、テスト、デプロイには中断がありません。</span><span class="sxs-lookup"><span data-stu-id="0862d-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="0862d-198">CD プロセスを管理および構成する</span><span class="sxs-lookup"><span data-stu-id="0862d-198">Manage and configure the CD process</span></span>

<span data-ttu-id="0862d-199">Azure DevOps Services が提供するパイプラインは自由に構成でき、管理性にも優れ、開発、ステージング、QA、運用など、さまざまな環境へのリリースに使用できます。また、特定のステージで承認を要求できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="0862d-200">リリース定義の作成</span><span class="sxs-lookup"><span data-stu-id="0862d-200">Create release definition</span></span>

1. <span data-ttu-id="0862d-201">Azure DevOps Services の **[ビルドとリリース]** セクションの **[リリース]** タブで **[+]** ボタンを選択して、新しいリリースを追加します。</span><span class="sxs-lookup"><span data-stu-id="0862d-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Azure DevOps Services でリリース定義を作成する](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="0862d-203">Azure App Service Deployment テンプレートを適用します。</span><span class="sxs-lookup"><span data-stu-id="0862d-203">Apply the Azure App Service Deployment template.</span></span>

   ![Azure DevOps Services で [Azure App Service の配置] テンプレートを適用する](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="0862d-205">**[成果物の追加]** で、Azure Cloud ビルド アプリに対して成果物を追加します。</span><span class="sxs-lookup"><span data-stu-id="0862d-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Azure DevOps Services で Azure Cloud ビルドに成果物を追加する](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="0862d-207">[パイプライン] タブで、環境の**フェーズ、タスク** リンクを選択し、Azure のクラウド環境の値を設定します。</span><span class="sxs-lookup"><span data-stu-id="0862d-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Azure DevOps Services で Azure クラウド環境の値を設定する](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="0862d-209">**環境名**を設定し、Azure Cloud エンドポイントに対して **Azure サブスクリプション**を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Azure DevOps Services で Azure Cloud エンドポイントの Azure サブスクリプションを選択する](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="0862d-211">**[App Service の名前]** で、必須の Azure アプリ サービス名を設定します。</span><span class="sxs-lookup"><span data-stu-id="0862d-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Azure DevOps Services で Azure アプリ サービス名を設定する](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="0862d-213">Azure クラウドでホストされる環境の **[エージェント キュー]** に「Hosted VS2017」と入力します。</span><span class="sxs-lookup"><span data-stu-id="0862d-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Azure DevOps Services で Azure クラウド ホスト環境のエージェント キューを設定する](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="0862d-215">[Azure App Service 配置] メニューで、環境に対して有効な**パッケージまたはフォルダー**を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="0862d-216">**[OK]** を選択して、**フォルダーの場所**を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-216">Select **OK** to **folder location**.</span></span>
  
      ![Azure DevOps Services で Azure App Service 環境のパッケージまたはフォルダーを選択する](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Azure DevOps Services で Azure App Service 環境のパッケージまたはフォルダーを選択する](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="0862d-219">すべての変更を保存し、**リリース パイプライン**に戻ります。</span><span class="sxs-lookup"><span data-stu-id="0862d-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Azure DevOps Services でリリース パイプラインの変更を保存する](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="0862d-221">Azure Stack Hub アプリのビルドを選択して、新しい成果物を追加します。</span><span class="sxs-lookup"><span data-stu-id="0862d-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Azure DevOps Services で Azure Stack Hub アプリの新しい成果物を追加する](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="0862d-223">[Azure App Service の配置] を適用し、環境をもう 1 つ追加します。</span><span class="sxs-lookup"><span data-stu-id="0862d-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Azure DevOps Services で [Azure App Service の配置] に環境を追加する](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="0862d-225">新しい環境に Azure Stack Hub という名前を付けます。</span><span class="sxs-lookup"><span data-stu-id="0862d-225">Name the new environment Azure Stack Hub.</span></span>

    ![Azure DevOps Services の [Azure App Service の配置] の環境に名前を付ける](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="0862d-227">**[タスク]** タブで Azure Stack Hub 環境を見つけます。</span><span class="sxs-lookup"><span data-stu-id="0862d-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Azure DevOps Services の Azure DevOps Services の Azure Stack Hub 環境](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="0862d-229">Azure Stack Hub エンドポイントのサブスクリプションを選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Azure DevOps Services で Azure Stack Hub エンドポイントのサブスクリプションを選択する](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="0862d-231">[App Service の名前] として Azure Stack Hub Web アプリの名前を設定します。</span><span class="sxs-lookup"><span data-stu-id="0862d-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Azure DevOps Services で Azure Stack Hub Web アプリ名を設定する](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="0862d-233">Azure Stack Hub エージェントを選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-233">Select the Azure Stack Hub agent.</span></span>

    ![Azure DevOps Services で Azure Stack Hub エージェントを選択する](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="0862d-235">[Azure App Service 配置] セクションで、環境に対して有効な**パッケージまたはフォルダー**を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="0862d-236">**[OK]** を選択して、フォルダーの場所を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-236">Select **OK** to folder location.</span></span>

    ![Azure DevOps Services で [Azure App Service の配置] のフォルダーを選択する](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Azure DevOps Services で [Azure App Service の配置] のフォルダーを選択する](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="0862d-239">[変数] タブで、`VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` という名前の変数を追加し、その値を **true** に設定し、スコープを Azure Stack Hub に設定します。</span><span class="sxs-lookup"><span data-stu-id="0862d-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Azure DevOps Services で [Azure App の配置] に変数を追加する](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="0862d-241">両方の成果物で**継続的**配置トリガー アイコンを選択し、**継続的**配置トリガーを有効にします。</span><span class="sxs-lookup"><span data-stu-id="0862d-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Azure DevOps Services で継続的配置トリガーを選択する](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="0862d-243">Azure Stack Hub 環境で**配置前**条件アイコンを選択し、トリガーを**リリース後**に設定します。</span><span class="sxs-lookup"><span data-stu-id="0862d-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Azure DevOps Services で配置前の条件を選択する](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="0862d-245">すべての変更を保存します。</span><span class="sxs-lookup"><span data-stu-id="0862d-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="0862d-246">タスクの一部の設定は、テンプレートからリリース定義を作成したときに、[環境変数](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables)として自動的に定義されている可能性があります。</span><span class="sxs-lookup"><span data-stu-id="0862d-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="0862d-247">こうした設定は、タスクの設定では変更できません。これらの設定を編集するには、親環境項目を選択する必要があります。</span><span class="sxs-lookup"><span data-stu-id="0862d-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="0862d-248">パート 2: Web アプリ オプションを更新する</span><span class="sxs-lookup"><span data-stu-id="0862d-248">Part 2: Update web app options</span></span>

<span data-ttu-id="0862d-249">[Azure App Service](/azure/app-service/overview) は、非常にスケーラブルな、自己適用型の Web ホスティング サービスを提供します。</span><span class="sxs-lookup"><span data-stu-id="0862d-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="0862d-251">既存のカスタム DNS 名を Azure Web Apps にマップします。</span><span class="sxs-lookup"><span data-stu-id="0862d-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="0862d-252">**CNAME レコード**と **A レコード**を使用し、カスタム DNS 名を App Service にマップします。</span><span class="sxs-lookup"><span data-stu-id="0862d-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="0862d-253">既存のカスタム DNS 名を Azure Web Apps にマップする</span><span class="sxs-lookup"><span data-stu-id="0862d-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="0862d-254">ルート ドメイン (northwind.com など) を除くすべてのカスタム DNS 名に CNAME を使用します。</span><span class="sxs-lookup"><span data-stu-id="0862d-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="0862d-255">ライブ サイトとその DNS ドメイン名を App Service に移行する方法については、「[Azure App Service へのアクティブな DNS 名の移行](/azure/app-service/manage-custom-dns-migrate-domain)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="0862d-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="0862d-256">前提条件</span><span class="sxs-lookup"><span data-stu-id="0862d-256">Prerequisites</span></span>

<span data-ttu-id="0862d-257">このソリューションを完了するには:</span><span class="sxs-lookup"><span data-stu-id="0862d-257">To complete this solution:</span></span>

- <span data-ttu-id="0862d-258">[App Service アプリを作成する](/azure/app-service/)か、別のソリューションで作成したアプリを使用します。</span><span class="sxs-lookup"><span data-stu-id="0862d-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="0862d-259">ドメイン名を購入し、ドメイン プロバイダーの DNS レジストリへのアクセスを確認します。</span><span class="sxs-lookup"><span data-stu-id="0862d-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="0862d-260">ドメインの DNS ゾーン ファイルを更新します。</span><span class="sxs-lookup"><span data-stu-id="0862d-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="0862d-261">Azure AD は、カスタム ドメイン名の所有権を確認します。</span><span class="sxs-lookup"><span data-stu-id="0862d-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="0862d-262">Azure 内の Azure/Microsoft 365/外部 DNS レコードに [Azure DNS](/azure/dns/dns-getstarted-portal) を使用するか、[別の DNS レジストラー](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)で DNS エントリを追加します。</span><span class="sxs-lookup"><span data-stu-id="0862d-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

- <span data-ttu-id="0862d-263">パブリック レジストラーでカスタム ドメインを登録します。</span><span class="sxs-lookup"><span data-stu-id="0862d-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="0862d-264">ドメインのドメイン名レジストラーにサインインします。</span><span class="sxs-lookup"><span data-stu-id="0862d-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="0862d-265">(DNS の更新を行うには、承認された管理者が必要になることがあります)。</span><span class="sxs-lookup"><span data-stu-id="0862d-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="0862d-266">Azure AD から提供された DNS エントリを追加して、ドメインの DNS ゾーン ファイルを更新します。</span><span class="sxs-lookup"><span data-stu-id="0862d-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="0862d-267">たとえば、northwindcloud.com と www\.northwindcloud.com の DNS エントリを追加するには、northwindcloud.com ルート ドメインの DNS 設定を構成します。</span><span class="sxs-lookup"><span data-stu-id="0862d-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="0862d-268">ドメイン名は [Microsoft Azure portal](/azure/app-service/manage-custom-dns-buy-domain) を使用して購入できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="0862d-269">Web アプリにカスタム DNS 名をマップするには、Web アプリの [App Service プラン](https://azure.microsoft.com/pricing/details/app-service/)が有料レベル (**Shared**、**Basic**、**Standard**、または **Premium**) である必要があります。</span><span class="sxs-lookup"><span data-stu-id="0862d-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="0862d-270">CNAME および A レコードを作成してマップする</span><span class="sxs-lookup"><span data-stu-id="0862d-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="0862d-271">ドメイン プロバイダーで DNS レコードにアクセスする</span><span class="sxs-lookup"><span data-stu-id="0862d-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="0862d-272">Azure DNS を使用して、Azure Web Apps のカスタム DNS 名を構成します。</span><span class="sxs-lookup"><span data-stu-id="0862d-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="0862d-273">詳細については、「[Azure DNS を使用して Azure サービス用のカスタム ドメイン設定を提供する](/azure/dns/dns-custom-domain)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="0862d-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="0862d-274">ドメイン プロバイダーの Web サイトにサインインします。</span><span class="sxs-lookup"><span data-stu-id="0862d-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="0862d-275">DNS レコードの管理ページを探します。</span><span class="sxs-lookup"><span data-stu-id="0862d-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="0862d-276">各ドメイン プロバイダーは、独自の DNS レコード インターフェイスを保有します。</span><span class="sxs-lookup"><span data-stu-id="0862d-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="0862d-277">**[ドメイン名]** 、 **[DNS]** 、 **[ネーム サーバー管理]** というラベルが付いたサイトの領域を探します。</span><span class="sxs-lookup"><span data-stu-id="0862d-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="0862d-278">DNS レコード ページは、 **[My domains] (マイ ドメイン)** で表示できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="0862d-279">**[ゾーン ファイル]** 、 **[DNS レコード]** 、または **[詳細構成]** という名前のリンクを見つけます。</span><span class="sxs-lookup"><span data-stu-id="0862d-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="0862d-280">以下のスクリーンショットは、DNS レコード ページの例です。</span><span class="sxs-lookup"><span data-stu-id="0862d-280">The following screenshot is an example of a DNS records page:</span></span>

![DNS レコード ページの例](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="0862d-282">ドメイン名レジストラーで、 **[Add or Create] (追加または作成)** を選択してレコードを作成します。</span><span class="sxs-lookup"><span data-stu-id="0862d-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="0862d-283">プロバイダーによっては、追加するレコード タイプごとに異なるリンクが用意されています。</span><span class="sxs-lookup"><span data-stu-id="0862d-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="0862d-284">プロバイダーのドキュメントを参照してください。</span><span class="sxs-lookup"><span data-stu-id="0862d-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="0862d-285">CNAME レコードを追加して、サブドメインをアプリの既定のホスト名にマップします。</span><span class="sxs-lookup"><span data-stu-id="0862d-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="0862d-286">www\.northwindcloud.com ドメインの例では、名前を `<app_name>.azurewebsites.net` にマップする CNAME レコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="0862d-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="0862d-287">CNAME を追加した後の DNS レコード ページは次の例のようになります。</span><span class="sxs-lookup"><span data-stu-id="0862d-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Azure アプリへのポータル ナビゲーション](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="0862d-289">Azure で CNAME レコード マッピングを有効にする</span><span class="sxs-lookup"><span data-stu-id="0862d-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="0862d-290">新しいタブで Azure portal にサインインします。</span><span class="sxs-lookup"><span data-stu-id="0862d-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="0862d-291">[App Services] に移動します。</span><span class="sxs-lookup"><span data-stu-id="0862d-291">Go to App Services.</span></span>

3. <span data-ttu-id="0862d-292">Web アプリを選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-292">Select web app.</span></span>

4. <span data-ttu-id="0862d-293">Azure Portal のアプリ ページの左側のナビゲーションで、 **[カスタム ドメイン]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="0862d-294">**[ホスト名の追加]** の横の **+** アイコンを選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="0862d-295">`www.northwindcloud.com` のように完全修飾ドメイン名を入力します。</span><span class="sxs-lookup"><span data-stu-id="0862d-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="0862d-296">**[検証]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-296">Select **Validate**.</span></span>

8. <span data-ttu-id="0862d-297">指示された場合は、他の種類 (`A` または `TXT`) の追加レコードをドメイン名レジストラー DNS レコードに追加します。</span><span class="sxs-lookup"><span data-stu-id="0862d-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="0862d-298">Azure は、これらのレコードの値と種類を提供します。</span><span class="sxs-lookup"><span data-stu-id="0862d-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="0862d-299">a.</span><span class="sxs-lookup"><span data-stu-id="0862d-299">a.</span></span>  <span data-ttu-id="0862d-300">アプリの IP アドレスにマップするための **A** レコード。</span><span class="sxs-lookup"><span data-stu-id="0862d-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="0862d-301">b.</span><span class="sxs-lookup"><span data-stu-id="0862d-301">b.</span></span>  <span data-ttu-id="0862d-302">アプリの既定のホスト名 `<app_name>.azurewebsites.net` にマップするための **TXT** レコード。</span><span class="sxs-lookup"><span data-stu-id="0862d-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="0862d-303">App Service は、このレコードを、カスタム ドメインの所有者を検証するために構成時にのみ使用します。</span><span class="sxs-lookup"><span data-stu-id="0862d-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="0862d-304">検証後、TXT レコードを削除してください。</span><span class="sxs-lookup"><span data-stu-id="0862d-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="0862d-305">ドメイン レジスター タブでこのタスクを完了し、 **[ホスト名の追加]** ボタンがアクティブになるまで、再検証します。</span><span class="sxs-lookup"><span data-stu-id="0862d-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="0862d-306">**[ホスト名レコード タイプ]** が **[CNAME]** (www.example.com または任意のサブドメイン) に設定されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="0862d-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="0862d-307">**[ホスト名の追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="0862d-308">`northwindcloud.com` のように完全修飾ドメイン名を入力します。</span><span class="sxs-lookup"><span data-stu-id="0862d-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="0862d-309">**[検証]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-309">Select **Validate**.</span></span> <span data-ttu-id="0862d-310">**[追加]** がアクティブになります。</span><span class="sxs-lookup"><span data-stu-id="0862d-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="0862d-311">**[ホスト名レコード タイプ]** が **[A レコード]** (example.com) に設定されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="0862d-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="0862d-312">**ホスト名を追加します**。</span><span class="sxs-lookup"><span data-stu-id="0862d-312">**Add hostname**.</span></span>

    <span data-ttu-id="0862d-313">アプリの **[カスタム ドメイン]** ページに新しいホスト名が反映されるまで時間がかかることがあります。</span><span class="sxs-lookup"><span data-stu-id="0862d-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="0862d-314">データを更新するために、ブラウザーの表示を更新してみてください。</span><span class="sxs-lookup"><span data-stu-id="0862d-314">Try refreshing the browser to update the data.</span></span>
  
    ![カスタム ドメイン](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="0862d-316">エラーが発生した場合は、検証エラーの通知がページの下部に表示されます。</span><span class="sxs-lookup"><span data-stu-id="0862d-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![ドメインの検証エラー](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="0862d-318">上記の手順を繰り返して、ワイルドカード ドメイン (\*northwindcloud.com) をマップできます。</span><span class="sxs-lookup"><span data-stu-id="0862d-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="0862d-319">これにより、それぞれに個別の CNAME レコードを作成せずに、このアプリ サービスに別のサブドメインを追加できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="0862d-320">レジストラーの指示に従って、この設定を構成します。</span><span class="sxs-lookup"><span data-stu-id="0862d-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="0862d-321">ブラウザーでテストする</span><span class="sxs-lookup"><span data-stu-id="0862d-321">Test in a browser</span></span>

<span data-ttu-id="0862d-322">先ほど構成された DNS 名 (`northwindcloud.com`、`www.northwindcloud.com` など) を参照します。</span><span class="sxs-lookup"><span data-stu-id="0862d-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="0862d-323">パート 3: カスタム SSL 証明書をバインドする</span><span class="sxs-lookup"><span data-stu-id="0862d-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="0862d-324">このパートでの作業:</span><span class="sxs-lookup"><span data-stu-id="0862d-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="0862d-325">カスタム SSL 証明書を App Service にバインドします。</span><span class="sxs-lookup"><span data-stu-id="0862d-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="0862d-326">アプリに HTTPS を適用します。</span><span class="sxs-lookup"><span data-stu-id="0862d-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="0862d-327">スクリプトで SSL 証明書のバインドを自動化します。</span><span class="sxs-lookup"><span data-stu-id="0862d-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="0862d-328">必要に応じて、Microsoft Azure portal で顧客の SSL 証明書を取得し、それを Web アプリにバインドします。</span><span class="sxs-lookup"><span data-stu-id="0862d-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="0862d-329">詳細については、[Azure App Service 証明書のチュートリアル](/azure/app-service/web-sites-purchase-ssl-web-site)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0862d-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="0862d-330">前提条件</span><span class="sxs-lookup"><span data-stu-id="0862d-330">Prerequisites</span></span>

<span data-ttu-id="0862d-331">このソリューションを完了するには:</span><span class="sxs-lookup"><span data-stu-id="0862d-331">To complete this  solution:</span></span>

- [<span data-ttu-id="0862d-332">App Service アプリを作成します。</span><span class="sxs-lookup"><span data-stu-id="0862d-332">Create an App Service app.</span></span>](/azure/app-service/)
- [<span data-ttu-id="0862d-333">カスタム DNS 名を Web アプリにマップします。</span><span class="sxs-lookup"><span data-stu-id="0862d-333">Map a custom DNS name to your web app.</span></span>](/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="0862d-334">信頼された証明機関から SSL 証明書を取得し、キーを使用して要求に署名します。</span><span class="sxs-lookup"><span data-stu-id="0862d-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="0862d-335">SSL 証明書の必要条件</span><span class="sxs-lookup"><span data-stu-id="0862d-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="0862d-336">App Service で証明書を使用するには、証明書が次のすべての要件を満たしている必要があります。</span><span class="sxs-lookup"><span data-stu-id="0862d-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="0862d-337">信頼された証明機関によって署名されています。</span><span class="sxs-lookup"><span data-stu-id="0862d-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="0862d-338">パスワードで保護された PFX ファイルとしてエクスポートされています。</span><span class="sxs-lookup"><span data-stu-id="0862d-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="0862d-339">少なくとも 2048 ビット長の秘密キーが含まれています。</span><span class="sxs-lookup"><span data-stu-id="0862d-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="0862d-340">証明書チェーン内のすべての中間証明書が含まれています。</span><span class="sxs-lookup"><span data-stu-id="0862d-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="0862d-341">**楕円曲線暗号 (ECC) 証明書**は、App Service で有効ですが、このガイドでは取り上げていません。</span><span class="sxs-lookup"><span data-stu-id="0862d-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="0862d-342">ECC 証明書の作成でサポートが必要なときは、証明機関に問い合わせてください。</span><span class="sxs-lookup"><span data-stu-id="0862d-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="0862d-343">Web アプリを用意する</span><span class="sxs-lookup"><span data-stu-id="0862d-343">Prepare the web app</span></span>

<span data-ttu-id="0862d-344">カスタム SSL 証明書を Web アプリにバインドするには、[App Service プラン](https://azure.microsoft.com/pricing/details/app-service/)が **Basic**、**Standard** または **Premium** のいずれかのレベルである必要があります。</span><span class="sxs-lookup"><span data-stu-id="0862d-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="0862d-345">Azure へのサインイン</span><span class="sxs-lookup"><span data-stu-id="0862d-345">Sign in to Azure</span></span>

1. <span data-ttu-id="0862d-346">[Azure portal](https://portal.azure.com/) を開き、Web アプリに移動します。</span><span class="sxs-lookup"><span data-stu-id="0862d-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="0862d-347">左側のメニューで、 **[App Services]** を選択し、Web アプリ名を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Azure portal で Web アプリを選択する](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="0862d-349">価格レベルの確認</span><span class="sxs-lookup"><span data-stu-id="0862d-349">Check the pricing tier</span></span>

1. <span data-ttu-id="0862d-350">Web アプリ ページの左側のナビゲーションで **[設定]** セクションまでスクロールし、 **[スケール アップ (App Service プラン)]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Web アプリのスケールアップ メニュー](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="0862d-352">Web アプリが **Free** レベルまたは **Shared** レベルに含まれていないことを確認します。</span><span class="sxs-lookup"><span data-stu-id="0862d-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="0862d-353">Web アプリの現在の層が濃青色のボックスに強調表示されます。</span><span class="sxs-lookup"><span data-stu-id="0862d-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![Web アプリで価格レベルを確認する](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="0862d-355">カスタム SSL は、**Free** レベルまたは **Shared** レベルではサポートされていません。</span><span class="sxs-lookup"><span data-stu-id="0862d-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="0862d-356">アップスケールするには、次のセクション、 **[価格レベルの選択]** ページの手順に従い、[[Upload and bind your SSL certificate]\(SSL 証明書のアップロードおよびバインド\)](/azure/app-service/app-service-web-tutorial-custom-ssl) にスキップします。</span><span class="sxs-lookup"><span data-stu-id="0862d-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="0862d-357">App Service プランのスケール アップ</span><span class="sxs-lookup"><span data-stu-id="0862d-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="0862d-358">**Basic**、**Standard**、**Premium** のいずれかのレベルを選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="0862d-359">**[選択]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-359">Select **Select**.</span></span>

![Web アプリの価格レベルを選択する](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="0862d-361">通知が表示されたら、スケール操作は完了です。</span><span class="sxs-lookup"><span data-stu-id="0862d-361">The scale operation is complete when notification is displayed.</span></span>

![スケール アップの通知](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="0862d-363">SSL 証明書をバインドし、中間証明書を結合する</span><span class="sxs-lookup"><span data-stu-id="0862d-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="0862d-364">チェーン内の複数の証明書を結合します。</span><span class="sxs-lookup"><span data-stu-id="0862d-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="0862d-365">テキストエディタで、受信した**それぞれの証明書を開きます**。</span><span class="sxs-lookup"><span data-stu-id="0862d-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="0862d-366">結合した証明書用に *mergedcertificate.crt* という名前のファイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="0862d-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="0862d-367">テキスト エディターで、このファイルに各証明書の内容をコピーします。</span><span class="sxs-lookup"><span data-stu-id="0862d-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="0862d-368">証明書の順序は、証明書チェーンの順番に従う必要があります。自分の証明書から始まり、ルート証明書で終わります。</span><span class="sxs-lookup"><span data-stu-id="0862d-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="0862d-369">次の例のようになります。</span><span class="sxs-lookup"><span data-stu-id="0862d-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="0862d-370">PFX への証明書のエクスポート</span><span class="sxs-lookup"><span data-stu-id="0862d-370">Export certificate to PFX</span></span>

<span data-ttu-id="0862d-371">証明書で生成された秘密キーを使用して、結合した SSL 証明書をエクスポートします。</span><span class="sxs-lookup"><span data-stu-id="0862d-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="0862d-372">秘密キー ファイルは OpenSSL 経由で作成されます。</span><span class="sxs-lookup"><span data-stu-id="0862d-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="0862d-373">証明書を PFX にエクスポートするには、次のコマンドを実行し、プレースホルダー `<private-key-file>` と `<merged-certificate-file>` を秘密キーのパスとマージされた証明書ファイルに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="0862d-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="0862d-374">プロンプトが表示されたら、後から SSL 証明書を App Service にアップロードするためのエクスポート パスワードを定義します。</span><span class="sxs-lookup"><span data-stu-id="0862d-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="0862d-375">IIS または **Certreq.exe** を使用して証明書の要求を生成した場合は、ローカル コンピューターに証明書をインストールした後で[証明書を PFX にエクスポート](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11))します。</span><span class="sxs-lookup"><span data-stu-id="0862d-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="0862d-376">SSL 証明書をアップロードする</span><span class="sxs-lookup"><span data-stu-id="0862d-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="0862d-377">Web アプリの左側のナビゲーションで **[SSL 設定]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="0862d-378">**[証明書のアップロード]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="0862d-379">**[PFX 証明書ファイル]** で、PFX ファイルを選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="0862d-380">**証明書のパスワード**、PFX ファイルをエクスポートするときに作成したパスワードを入力します。</span><span class="sxs-lookup"><span data-stu-id="0862d-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="0862d-381">**[アップロード]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-381">Select **Upload**.</span></span>

    ![SSL 証明書をアップロードする](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="0862d-383">App Service による証明書のアップロードが完了すると、 **[SSL 設定]** ページにアップロードした証明書が表示されます。</span><span class="sxs-lookup"><span data-stu-id="0862d-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![SSL 設定](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="0862d-385">SSL 証明書のバインド</span><span class="sxs-lookup"><span data-stu-id="0862d-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="0862d-386">**[SSL バインド]** セクションで **[バインドの追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="0862d-387">証明書をアップロードしたのに **[ホスト名]** ドロップダウンにドメイン名が表示されない場合は、ブラウザのページを最新の情報に更新してみてください。</span><span class="sxs-lookup"><span data-stu-id="0862d-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="0862d-388">**[SSL バインディングの追加]** ページで、ドロップダウンから保護するドメインの名前と使用する証明書を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="0862d-389">**[SSL Type] \(SSL の種類)** で、[**Server Name Indication (SNI)** ](https://en.wikipedia.org/wiki/Server_Name_Indication) ベースの SSL を使用するか IP ベースの SSL を使用するかを選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="0862d-390">**SNI ベースの SSL**:複数の SNI ベースの SSL バインディングを追加できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="0862d-391">このオプションでは、複数の SSL 証明書を使用して、同一の IP アドレス上の複数のドメインを保護できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="0862d-392">最新のブラウザーのほとんど (Inernet Explorer、Chrome、Firefox、Opera など) が SNI をサポートしています (ブラウザーのサポートに関するより包括的な情報については、「[Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)」を参照してください)。</span><span class="sxs-lookup"><span data-stu-id="0862d-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="0862d-393">**IP ベースの SSL**:IP ベースの SSL バインディングを 1 つだけ追加できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="0862d-394">このオプションでは、SSL 証明書を 1 つだけ使用して、専用のパブリック IP アドレスを保護します。</span><span class="sxs-lookup"><span data-stu-id="0862d-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="0862d-395">複数のドメインを保護するには、同じ SSL 証明書を使用してすべてのドメインを保護します。</span><span class="sxs-lookup"><span data-stu-id="0862d-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="0862d-396">IP ベースの SSL は、SSL バインドの従来のオプションです。</span><span class="sxs-lookup"><span data-stu-id="0862d-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="0862d-397">**[バインドの追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-397">Select **Add Binding**.</span></span>

    ![SSL バインドの追加](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="0862d-399">App Service による証明書のアップロードが完了すると、 **[SSL バインド]** セクションにアップロードした証明書が表示されます。</span><span class="sxs-lookup"><span data-stu-id="0862d-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![SSL バインドのアップロードの完了](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="0862d-401">IP SSL の A レコードを再マップする</span><span class="sxs-lookup"><span data-stu-id="0862d-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="0862d-402">Web アプリで IP ベースの SSL を使用していない場合、[カスタム ドメインの HTTPS のテスト](/azure/app-service/app-service-web-tutorial-custom-ssl)に関するセクションにスキップしてください。</span><span class="sxs-lookup"><span data-stu-id="0862d-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="0862d-403">既定では、Web アプリは、共有のパブリック IP アドレスを使用します。</span><span class="sxs-lookup"><span data-stu-id="0862d-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="0862d-404">IP ベースの SSL で証明書をバインドすると、Web アプリ用の新規の専用 IP アドレスが App Service によって作成されます。</span><span class="sxs-lookup"><span data-stu-id="0862d-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="0862d-405">A レコードが Web アプリにマップされた場合、ドメイン レジストリを専用の IP アドレスで更新する必要があります。</span><span class="sxs-lookup"><span data-stu-id="0862d-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="0862d-406">**[カスタム ドメイン]** ページが、新規の専用 IP アドレスで更新されます。</span><span class="sxs-lookup"><span data-stu-id="0862d-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="0862d-407">[この IP アドレスをコピー](/azure/app-service/app-service-web-tutorial-custom-domain)して、この新しい IP アドレスに [A レコードを再マップ](/azure/app-service/app-service-web-tutorial-custom-domain)します。</span><span class="sxs-lookup"><span data-stu-id="0862d-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="0862d-408">HTTPS のテスト</span><span class="sxs-lookup"><span data-stu-id="0862d-408">Test HTTPS</span></span>

<span data-ttu-id="0862d-409">さまざまなブラウザーで `https://<your.custom.domain>` にアクセスして、Web アプリが提供されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="0862d-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![Web アプリの参照](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="0862d-411">証明書の検証エラーが発生した場合、自己署名証明書が原因であるか、PFX ファイルにエクスポートするときに中間証明書が除外された可能性があります。</span><span class="sxs-lookup"><span data-stu-id="0862d-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="0862d-412">HTTPS の適用</span><span class="sxs-lookup"><span data-stu-id="0862d-412">Enforce HTTPS</span></span>

<span data-ttu-id="0862d-413">既定では、どなたでも HTTP を使用して Web アプリにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="0862d-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="0862d-414">HTTPS ポートへのすべての HTTP 要求をリダイレクトできます。</span><span class="sxs-lookup"><span data-stu-id="0862d-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="0862d-415">Web アプリページで、 **[SSL 設定]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="0862d-416">その後、 **[HTTPS のみ]** で、 **[On]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-416">Then, in **HTTPS Only**, select **On**.</span></span>

![HTTPS の適用](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="0862d-418">操作が完了すると、アプリを指定する HTTP URL のいずれかに移動します。</span><span class="sxs-lookup"><span data-stu-id="0862d-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="0862d-419">次に例を示します。</span><span class="sxs-lookup"><span data-stu-id="0862d-419">For example:</span></span>

- <span data-ttu-id="0862d-420">https://<app_name>.azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="0862d-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="0862d-421">TLS 1.1/1.2 の適用</span><span class="sxs-lookup"><span data-stu-id="0862d-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="0862d-422">アプリでは既定で [TLS 1.0](https://wikipedia.org/wiki/Transport_Layer_Security) が有効です。これは、業界標準 ([PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard) など) によって安全であると見なされなくなっています。</span><span class="sxs-lookup"><span data-stu-id="0862d-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="0862d-423">より上位の TLS バージョンを適用するには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="0862d-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="0862d-424">Web アプリ ページで、左側のナビゲーションにある **[SSL 設定]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="0862d-425">**[TLS version] (TLS バージョン)** で、最低限の TLS バージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![TLS 1.1/1.2 の適用](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="0862d-427">Traffic Manager プロファイルの作成</span><span class="sxs-lookup"><span data-stu-id="0862d-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="0862d-428">**[リソースの作成]**  >  **[ネットワーク]**  >  **[Traffic Manager プロファイル]**  >  **[作成]** の順に選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="0862d-429">**[Traffic Manager プロファイルの作成]** で、以下を実行します。</span><span class="sxs-lookup"><span data-stu-id="0862d-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="0862d-430">**[名前]** に、プロファイルの名前を入力します。</span><span class="sxs-lookup"><span data-stu-id="0862d-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="0862d-431">この名前は trafficmanager.net ゾーン内で一意である必要があります。結果的に、Traffic Manager プロファイルへのアクセスに使用される DNS 名 trafficmanager.net になるためです。</span><span class="sxs-lookup"><span data-stu-id="0862d-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="0862d-432">**[ルーティング方法]** で、**地理的ルーティング方法**を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="0862d-433">**[サブスクリプション]** で、このプロファイルを作成するサブスクリプションを選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="0862d-434">**[リソース グループ]** で、このプロファイルを配置する新しいリソース グループを作成します。</span><span class="sxs-lookup"><span data-stu-id="0862d-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="0862d-435">**[リソース グループの場所]** で、リソース グループの場所を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="0862d-436">これはリソース グループの場所を指定する設定であり、グローバルにデプロイされる Traffic Manager プロファイルには影響しません。</span><span class="sxs-lookup"><span data-stu-id="0862d-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="0862d-437">**［作成］** を選択します</span><span class="sxs-lookup"><span data-stu-id="0862d-437">Select **Create**.</span></span>

    7. <span data-ttu-id="0862d-438">Traffic Manager プロファイルは、グローバルなデプロイが完了すると、それぞれのリソース グループ内にリソースの 1 つとして表示されます。</span><span class="sxs-lookup"><span data-stu-id="0862d-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Traffic Manager プロファイルのリソース グループ作成](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="0862d-440">Traffic Manager エンドポイントの追加</span><span class="sxs-lookup"><span data-stu-id="0862d-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="0862d-441">ポータルの検索バーで、前のセクションで作成した **Traffic Manager プロファイル**の名前を検索し、表示された結果から Traffic Manager プロファイルを選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="0862d-442">**[Traffic Manager プロファイル]** の **[設定]** セクションで、 **[エンドポイント]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="0862d-443">**[追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-443">Select **Add**.</span></span>

4. <span data-ttu-id="0862d-444">Azure Stack Hub エンドポイントを追加します。</span><span class="sxs-lookup"><span data-stu-id="0862d-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="0862d-445">**[Type] (種類)** で、 **[外部エンドポイント]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="0862d-446">このエンドポイントの**名前**を、理想的には Azure Stack Hub の名前を入力します。</span><span class="sxs-lookup"><span data-stu-id="0862d-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="0862d-447">完全修飾ドメイン名 (**FQDN**) については、Azure Stack Hub Web アプリの外部 URL を使用します。</span><span class="sxs-lookup"><span data-stu-id="0862d-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="0862d-448">地理的マッピングで、リソースが置かれているリージョン/大陸を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="0862d-449">たとえば**ヨーロッパ**を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="0862d-450">表示された [Country/Region]\(国/リージョン\) ドロップダウンで、このエンドポイントに適用される国を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="0862d-451">たとえば**ドイツ**を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="0862d-452">**[Add as disabled (無効として追加)]** はオフのままにします。</span><span class="sxs-lookup"><span data-stu-id="0862d-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="0862d-453">**[OK]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-453">Select **OK**.</span></span>

12. <span data-ttu-id="0862d-454">Azure エンドポイントの追加:</span><span class="sxs-lookup"><span data-stu-id="0862d-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="0862d-455">**[Type] (種類)** で、 **[Azure エンドポイント]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="0862d-456">エンドポイントに **[名前]** を入力します。</span><span class="sxs-lookup"><span data-stu-id="0862d-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="0862d-457">**[ターゲット リソースの種類]** で、 **[App Service]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="0862d-458">**[ターゲット リソース]** で、 **[アプリ サービスの選択]** を選択し、同じサブスクリプションにある Web Apps の一覧を表示します。</span><span class="sxs-lookup"><span data-stu-id="0862d-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="0862d-459">**[リソース]** で、最初のエンドポイントとして使用されるアプリ サービスを選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="0862d-460">地理的マッピングで、リソースが置かれているリージョン/大陸を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="0862d-461">たとえば、**北米/中央アメリカ/カリブ**を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="0862d-462">表示された [Country/Region]\(国/リージョン\) ドロップダウンで、このスポットを空のまま残すと、上のリージョン グループがすべて選択されます。</span><span class="sxs-lookup"><span data-stu-id="0862d-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="0862d-463">**[Add as disabled (無効として追加)]** はオフのままにします。</span><span class="sxs-lookup"><span data-stu-id="0862d-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="0862d-464">**[OK]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0862d-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="0862d-465">[All (World)] (すべて (世界)) の地理的範囲を持つ少なくとも 1 つのエンドポイントを作成して、リソースの既定のエンドポイントとして機能します。</span><span class="sxs-lookup"><span data-stu-id="0862d-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="0862d-466">両方のエンドポイントが追加されると、 **[Traffic Manager プロファイル]** に表示され、監視ステータスが **[オンライン]** になります。</span><span class="sxs-lookup"><span data-stu-id="0862d-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Traffic Manager プロファイル エンドポイントの状態](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="0862d-468">グローバル エンタープライズには Azure の地理的分散機能が必要</span><span class="sxs-lookup"><span data-stu-id="0862d-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="0862d-469">Azure Traffic Manager と地域固有のエンドポイントを利用してデータ トラフィックを転送することで、グローバル企業は地域の規制に準拠し、現地/遠隔地を問わず、ビジネスの成功に不可欠であるデータ コンプライアンスとデータ セキュリティを維持できます。</span><span class="sxs-lookup"><span data-stu-id="0862d-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="0862d-470">次のステップ</span><span class="sxs-lookup"><span data-stu-id="0862d-470">Next steps</span></span>

- <span data-ttu-id="0862d-471">Azure のクラウド パターンの詳細については、「[Cloud Design Pattern (クラウド設計パターン)](/azure/architecture/patterns)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0862d-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
