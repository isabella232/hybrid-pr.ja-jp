---
title: Azure Stack Hub の地理的に分散されたアプリ パターン
description: Azure と Azure Stack Hub を使用した、インテリジェント エッジの地理的に分散されたアプリ パターンについて説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910881"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="99de8-103">地理的に分散されたアプリ パターン</span><span class="sxs-lookup"><span data-stu-id="99de8-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="99de8-104">複数のリージョンにアプリ エンドポイントを提供し、場所とコンプライアンスのニーズに基づいてユーザー トラフィックをルーティングする方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="99de8-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="99de8-105">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="99de8-105">Context and problem</span></span>

<span data-ttu-id="99de8-106">広い地理的範囲にまたがる組織は、必要なレベルのセキュリティ、コンプライアンス、およびパフォーマンスを、境界を越えてユーザー、場所、およびデバイスごとに確保しながら、データへのアクセスを安全かつ正確に分散させて実現するために努力します。</span><span class="sxs-lookup"><span data-stu-id="99de8-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="99de8-107">解決策</span><span class="sxs-lookup"><span data-stu-id="99de8-107">Solution</span></span>

<span data-ttu-id="99de8-108">Azure Stack Hub の地理的トラフィック ルーティング パターン (地理的に分散されたアプリ) では、さまざまなメトリックに基づいてトラフィックを特定のエンドポイントに送信できます。</span><span class="sxs-lookup"><span data-stu-id="99de8-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="99de8-109">地理的ベースのルーティングとエンドポイント構成を使用して Traffic Manager を作成すると、リージョンの要件、企業および国際的な規制、およびデータ ニーズに基づいてトラフィックをエンドポイントにルーティングできます。</span><span class="sxs-lookup"><span data-stu-id="99de8-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![地理的に分散されたパターン](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="99de8-111">Components</span><span class="sxs-lookup"><span data-stu-id="99de8-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="99de8-112">クラウドの外部</span><span class="sxs-lookup"><span data-stu-id="99de8-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="99de8-113">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="99de8-113">Traffic Manager</span></span>

<span data-ttu-id="99de8-114">この図では、Traffic Manager はパブリック クラウドの外部にありますが、ローカル データセンターとパブリック クラウドの両方でトラフィックを調整できる必要があります。</span><span class="sxs-lookup"><span data-stu-id="99de8-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="99de8-115">バランサーは地理的な場所にトラフィックをルーティングします。</span><span class="sxs-lookup"><span data-stu-id="99de8-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="99de8-116">ドメイン ネーム システム (DNS)</span><span class="sxs-lookup"><span data-stu-id="99de8-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="99de8-117">ドメイン ネーム システム (DNS) は、Web サイトまたはサービスの名前をその IP アドレスに変換する (または解決する) 役割を担います。</span><span class="sxs-lookup"><span data-stu-id="99de8-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="99de8-118">パブリック クラウド</span><span class="sxs-lookup"><span data-stu-id="99de8-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="99de8-119">クラウド エンドポイント</span><span class="sxs-lookup"><span data-stu-id="99de8-119">Cloud Endpoint</span></span>

<span data-ttu-id="99de8-120">パブリック IP アドレスは、受信トラフィックをトラフィック マネージャー経由でパブリック クラウド アプリのリソース エンドポイントにルーティングするために使用されます。</span><span class="sxs-lookup"><span data-stu-id="99de8-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="99de8-121">ローカル クラウド</span><span class="sxs-lookup"><span data-stu-id="99de8-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="99de8-122">ローカル エンドポイント</span><span class="sxs-lookup"><span data-stu-id="99de8-122">Local endpoint</span></span>

<span data-ttu-id="99de8-123">パブリック IP アドレスは、受信トラフィックをトラフィック マネージャー経由でパブリック クラウド アプリのリソース エンドポイントにルーティングするために使用されます。</span><span class="sxs-lookup"><span data-stu-id="99de8-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="99de8-124">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="99de8-124">Issues and considerations</span></span>

<span data-ttu-id="99de8-125">このパターンの実装方法を決めるときには、以下の点に注意してください。</span><span class="sxs-lookup"><span data-stu-id="99de8-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="99de8-126">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="99de8-126">Scalability</span></span>

<span data-ttu-id="99de8-127">このパターンでは、トラフィックの増加に合わせてスケールするのではなく、地理的なトラフィック ルーティングを処理します。</span><span class="sxs-lookup"><span data-stu-id="99de8-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="99de8-128">ただし、このパターンを他の Azure およびオンプレミスのソリューションと組み合わせることができます。</span><span class="sxs-lookup"><span data-stu-id="99de8-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="99de8-129">たとえば、このパターンはクラウド間スケーリング パターンと一緒に使用できます。</span><span class="sxs-lookup"><span data-stu-id="99de8-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="99de8-130">可用性</span><span class="sxs-lookup"><span data-stu-id="99de8-130">Availability</span></span>

<span data-ttu-id="99de8-131">オンプレミス ハードウェア構成およびソフトウェア デプロイを通じて高可用性をもたらすように、ローカルでデプロイされたアプリが構成されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="99de8-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="99de8-132">管理の容易性</span><span class="sxs-lookup"><span data-stu-id="99de8-132">Manageability</span></span>

<span data-ttu-id="99de8-133">パターンは、複数の環境にわたるシームレスな管理と使い慣れたインターフェイスを実現します。</span><span class="sxs-lookup"><span data-stu-id="99de8-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="99de8-134">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="99de8-134">When to use this pattern</span></span>

- <span data-ttu-id="99de8-135">組織には、リージョンのセキュリティおよび分散に関するカスタム ポリシーが必要な支店が世界中にあります。</span><span class="sxs-lookup"><span data-stu-id="99de8-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="99de8-136">組織の支店それぞれは、従業員、ビジネス、および施設のデータをプルしますが、これには地域の規制やタイム ゾーンに従ったレポート アクティビティが必要になります。</span><span class="sxs-lookup"><span data-stu-id="99de8-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="99de8-137">高スケール要件を満たすには、極端に負荷の大きい要件に対応できるように、単一のリージョン内、および複数のリージョンにわたって、複数のアプリ デプロイを使用してアプリを水平方向に拡張します。</span><span class="sxs-lookup"><span data-stu-id="99de8-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="99de8-138">1 つのリージョンで障害が発生した場合でも、アプリは高可用性とクライアント要求への応答性を実現する必要があります。</span><span class="sxs-lookup"><span data-stu-id="99de8-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="99de8-139">次のステップ</span><span class="sxs-lookup"><span data-stu-id="99de8-139">Next steps</span></span>

<span data-ttu-id="99de8-140">この記事で紹介したトピックの関連情報:</span><span class="sxs-lookup"><span data-stu-id="99de8-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="99de8-141">この DNS ベースのトラフィック ロード バランサーの詳細なしくみについては、「[Traffic Manager について](/azure/traffic-manager/traffic-manager-overview)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="99de8-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="99de8-142">ベスト プラクティスの詳細とその他の疑問の回答を得るには、「[ハイブリッド アプリケーション設計に関する考慮事項](overview-app-design-considerations.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="99de8-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="99de8-143">製品とソリューションのポートフォリオ全体の詳細について、[Azure Stack ファミリの製品とソリューション](/azure-stack)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="99de8-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="99de8-144">ソリューションの例をテストする準備ができたら、[地理的に分散されたアプリのソリューション デプロイ ガイド](solution-deployment-guide-geo-distributed.md)に進んでください。</span><span class="sxs-lookup"><span data-stu-id="99de8-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="99de8-145">デプロイ ガイドでは、コンポーネントをデプロイしてテストするための詳細な手順について説明します。</span><span class="sxs-lookup"><span data-stu-id="99de8-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="99de8-146">地理的に分散されたアプリ パターンを使用し、さまざまなメトリックに基づいて特定のエンドポイントにトラフィックを送信する方法を学習します。</span><span class="sxs-lookup"><span data-stu-id="99de8-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="99de8-147">地理的ベースのルーティングとエンドポイント構成で Traffic Manager プロファイルを作成すると、リージョンの要件、企業および国際的な規制、およびデータ ニーズに基づいて、情報がエンドポイントにルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="99de8-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
