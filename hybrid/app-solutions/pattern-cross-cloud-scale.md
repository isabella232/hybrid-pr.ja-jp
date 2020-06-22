---
title: Azure Stack Hub のクラウド間スケーリング パターン
description: Azure と Azure Stack Hub でスケーラブルなクラウド間アプリを構築する方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911133"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="70aec-103">クラウド間スケーリング パターン</span><span class="sxs-lookup"><span data-stu-id="70aec-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="70aec-104">負荷の増加に対応するために、既存のアプリにリソースを自動的に追加します。</span><span class="sxs-lookup"><span data-stu-id="70aec-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="70aec-105">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="70aec-105">Context and problem</span></span>

<span data-ttu-id="70aec-106">アプリは、予期しない需要の増加に対応するために容量を増やすことはできません。</span><span class="sxs-lookup"><span data-stu-id="70aec-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="70aec-107">このように増量できないことで、使用量がピークの時間帯にユーザーがアプリにアクセスできなくなります。</span><span class="sxs-lookup"><span data-stu-id="70aec-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="70aec-108">アプリは一定数のユーザーにサービスを提供できます。</span><span class="sxs-lookup"><span data-stu-id="70aec-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="70aec-109">グローバル企業には、セキュリティで保護された、信頼性が高く、利用可能なクラウドベースのアプリが必要です。</span><span class="sxs-lookup"><span data-stu-id="70aec-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="70aec-110">需要の増加に対応し、適切なインフラストラクチャを使用してその需要をサポートすることが重要です。</span><span class="sxs-lookup"><span data-stu-id="70aec-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="70aec-111">ビジネスにとって、コストおよびメンテナンスと、ビジネス データのセキュリティ、ストレージ、リアルタイム可用性の間でバランスを取ることは困難になっています。</span><span class="sxs-lookup"><span data-stu-id="70aec-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="70aec-112">パブリック クラウドでアプリを実行できない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="70aec-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="70aec-113">ただし、アプリの需要の急増に対処するためにオンプレミス環境で必要になる容量を維持することは、企業にとって経済的に見合わない場合があります。</span><span class="sxs-lookup"><span data-stu-id="70aec-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="70aec-114">このパターンでは、パブリック クラウドの弾力性をオンプレミス ソリューションで利用できます。</span><span class="sxs-lookup"><span data-stu-id="70aec-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="70aec-115">解決策</span><span class="sxs-lookup"><span data-stu-id="70aec-115">Solution</span></span>

<span data-ttu-id="70aec-116">クラウド間スケーリング パターンでは、ローカル クラウドに配置されたアプリを、パブリック クラウドのリソースを使用して拡張します。</span><span class="sxs-lookup"><span data-stu-id="70aec-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="70aec-117">このパターンは、需要の増減によってトリガーされ、クラウド内のリソースを追加または削除します。</span><span class="sxs-lookup"><span data-stu-id="70aec-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="70aec-118">これらのリソースは、冗長性、迅速な可用性、および地理的に準拠したルーティングを提供します。</span><span class="sxs-lookup"><span data-stu-id="70aec-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![クラウド間スケーリング パターン](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="70aec-120">このパターンは、アプリのステートレス コンポーネントにのみ適用されます。</span><span class="sxs-lookup"><span data-stu-id="70aec-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="70aec-121">Components</span><span class="sxs-lookup"><span data-stu-id="70aec-121">Components</span></span>

<span data-ttu-id="70aec-122">クラウド間スケーリング パターンは、次のコンポーネントで構成されています。</span><span class="sxs-lookup"><span data-stu-id="70aec-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="70aec-123">クラウドの外部</span><span class="sxs-lookup"><span data-stu-id="70aec-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="70aec-124">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="70aec-124">Traffic Manager</span></span>

<span data-ttu-id="70aec-125">図では、これはパブリック クラウド グループの外側にありますが、ローカル データセンターとパブリック クラウドの両方でトラフィックを調整できる必要があります。</span><span class="sxs-lookup"><span data-stu-id="70aec-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="70aec-126">このバランサーは、エンドポイントを監視し、必要に応じてフェールオーバーの再分配を提供することによって、アプリの高可用性を実現します。</span><span class="sxs-lookup"><span data-stu-id="70aec-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="70aec-127">ドメイン ネーム システム (DNS)</span><span class="sxs-lookup"><span data-stu-id="70aec-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="70aec-128">ドメイン ネーム システム (DNS) は、Web サイトまたはサービスの名前をその IP アドレスに変換する (または解決する) 役割を担います。</span><span class="sxs-lookup"><span data-stu-id="70aec-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="70aec-129">クラウド</span><span class="sxs-lookup"><span data-stu-id="70aec-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="70aec-130">ホスト型ビルド サーバー</span><span class="sxs-lookup"><span data-stu-id="70aec-130">Hosted build server</span></span>

<span data-ttu-id="70aec-131">ビルド パイプラインをホストするための環境。</span><span class="sxs-lookup"><span data-stu-id="70aec-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="70aec-132">アプリ リソース</span><span class="sxs-lookup"><span data-stu-id="70aec-132">App resources</span></span>

<span data-ttu-id="70aec-133">アプリ リソースは、仮想マシン スケール セットや Containers のように、スケールインとスケールアウトが可能である必要があります。</span><span class="sxs-lookup"><span data-stu-id="70aec-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="70aec-134">カスタム ドメイン名</span><span class="sxs-lookup"><span data-stu-id="70aec-134">Custom domain name</span></span>

<span data-ttu-id="70aec-135">ルーティング要求 glob には、カスタム ドメイン名を使用します。</span><span class="sxs-lookup"><span data-stu-id="70aec-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="70aec-136">パブリック IP アドレス</span><span class="sxs-lookup"><span data-stu-id="70aec-136">Public IP addresses</span></span>

<span data-ttu-id="70aec-137">パブリック IP アドレスは、受信トラフィックをトラフィック マネージャー経由でパブリック クラウド アプリのリソース エンドポイントにルーティングするために使用されます。</span><span class="sxs-lookup"><span data-stu-id="70aec-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="70aec-138">ローカル クラウド</span><span class="sxs-lookup"><span data-stu-id="70aec-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="70aec-139">ホスト型ビルド サーバー</span><span class="sxs-lookup"><span data-stu-id="70aec-139">Hosted build server</span></span>

<span data-ttu-id="70aec-140">ビルド パイプラインをホストするための環境。</span><span class="sxs-lookup"><span data-stu-id="70aec-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="70aec-141">アプリ リソース</span><span class="sxs-lookup"><span data-stu-id="70aec-141">App resources</span></span>

<span data-ttu-id="70aec-142">アプリ リソースは、仮想マシン スケール セットや Containers のように、スケールインとスケールアウトが可能である必要があります。</span><span class="sxs-lookup"><span data-stu-id="70aec-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="70aec-143">カスタム ドメイン名</span><span class="sxs-lookup"><span data-stu-id="70aec-143">Custom domain name</span></span>

<span data-ttu-id="70aec-144">ルーティング要求 glob には、カスタム ドメイン名を使用します。</span><span class="sxs-lookup"><span data-stu-id="70aec-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="70aec-145">パブリック IP アドレス</span><span class="sxs-lookup"><span data-stu-id="70aec-145">Public IP addresses</span></span>

<span data-ttu-id="70aec-146">パブリック IP アドレスは、受信トラフィックをトラフィック マネージャー経由でパブリック クラウド アプリのリソース エンドポイントにルーティングするために使用されます。</span><span class="sxs-lookup"><span data-stu-id="70aec-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="70aec-147">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="70aec-147">Issues and considerations</span></span>

<span data-ttu-id="70aec-148">このパターンの実装方法を決めるときには、以下の点に注意してください。</span><span class="sxs-lookup"><span data-stu-id="70aec-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="70aec-149">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="70aec-149">Scalability</span></span>

<span data-ttu-id="70aec-150">クラウド間スケーリングの主要構成要素はオンデマンド スケーリングを与える機能です。</span><span class="sxs-lookup"><span data-stu-id="70aec-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="70aec-151">スケーリングはパブリックとローカルのクラウド インフラストラクチャの間で行う必要があります。また、要求に関して、一貫性があり、信頼できるサービスを提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="70aec-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="70aec-152">可用性</span><span class="sxs-lookup"><span data-stu-id="70aec-152">Availability</span></span>

<span data-ttu-id="70aec-153">オンプレミス ハードウェア構成およびソフトウェア デプロイを通じて高可用性をもたらすように、ローカルでデプロイされたアプリが構成されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="70aec-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="70aec-154">管理の容易性</span><span class="sxs-lookup"><span data-stu-id="70aec-154">Manageability</span></span>

<span data-ttu-id="70aec-155">クラウド間パターンは、複数の環境にわたるシームレスな管理と使い慣れたインターフェイスを実現します。</span><span class="sxs-lookup"><span data-stu-id="70aec-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="70aec-156">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="70aec-156">When to use this pattern</span></span>

<span data-ttu-id="70aec-157">このパターンは次の目的で使用します。</span><span class="sxs-lookup"><span data-stu-id="70aec-157">Use this pattern:</span></span>

- <span data-ttu-id="70aec-158">予期しない需要や定期的な需要に応じてアプリの容量を増やす必要がある場合。</span><span class="sxs-lookup"><span data-stu-id="70aec-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="70aec-159">ピーク時にしか使用されないリソースに投資したくない場合。</span><span class="sxs-lookup"><span data-stu-id="70aec-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="70aec-160">従量課金制となります。</span><span class="sxs-lookup"><span data-stu-id="70aec-160">Pay for what you use.</span></span>

<span data-ttu-id="70aec-161">このパターンは次の場合は推奨されません。</span><span class="sxs-lookup"><span data-stu-id="70aec-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="70aec-162">インターネット経由で接続するユーザーがソリューションに必要である。</span><span class="sxs-lookup"><span data-stu-id="70aec-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="70aec-163">発信接続がオンサイト呼び出しからのものであることを要求するローカルの規制がビジネスに適用される。</span><span class="sxs-lookup"><span data-stu-id="70aec-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="70aec-164">ネットワークで定期的なボトルネックが発生しており、これがスケーリングのパフォーマンスを制限している。</span><span class="sxs-lookup"><span data-stu-id="70aec-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="70aec-165">環境がインターネットから切断されていて、パブリック クラウドに接続できない。</span><span class="sxs-lookup"><span data-stu-id="70aec-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="70aec-166">次のステップ</span><span class="sxs-lookup"><span data-stu-id="70aec-166">Next steps</span></span>

<span data-ttu-id="70aec-167">この記事で紹介したトピックの関連情報:</span><span class="sxs-lookup"><span data-stu-id="70aec-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="70aec-168">この DNS ベースのトラフィック ロード バランサーの詳細なしくみについては、「[Traffic Manager について](/azure/traffic-manager/traffic-manager-overview)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="70aec-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="70aec-169">ベスト プラクティスの詳細を確認し、その他の疑問の回答を取得するには、[ハイブリッド アプリケーションの設計上の考慮事項](overview-app-design-considerations.md)に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="70aec-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="70aec-170">製品とソリューションのポートフォリオ全体の詳細について、[Azure Stack の製品ファミリとソリューション](/azure-stack)に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="70aec-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="70aec-171">ソリューションの例をテストする準備ができたら、[クラウド間スケーリングのソリューション デプロイ ガイド](solution-deployment-guide-cross-cloud-scaling.md)に進んでください。</span><span class="sxs-lookup"><span data-stu-id="70aec-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="70aec-172">デプロイ ガイドでは、コンポーネントをデプロイしてテストするための詳細な手順について説明します。</span><span class="sxs-lookup"><span data-stu-id="70aec-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="70aec-173">Azure Stack Hub でホストされる Web アプリから Azure でホストされる Web アプリに切り替えるための手動トリガー プロセスを提供する、クラウド間ソリューションを作成する方法について学習します。</span><span class="sxs-lookup"><span data-stu-id="70aec-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="70aec-174">また、Traffic Manager を介して自動スケーリングを使用し、負荷に対して柔軟かつスケーラブルなクラウド ユーティリティを実現する方法についても学習します。</span><span class="sxs-lookup"><span data-stu-id="70aec-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
