---
title: Azure と Azure Stack Hub 向けのハイブリッド パターンとソリューションの例
description: Azure と Azure Stack Hub でのハイブリッド ソリューションの学習と構築のためのハイブリッド パターンとソリューションの例の概要。
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343860"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a><span data-ttu-id="c051e-103">Azure と Azure Stack 向けのハイブリッド ソリューションのパターンと例</span><span class="sxs-lookup"><span data-stu-id="c051e-103">Hybrid solution patterns and examples for Azure and Azure Stack</span></span>

<span data-ttu-id="c051e-104">Microsoft では、Azure と Azure Stack の製品とソリューションを 1 つの一貫した Azure エコシステムとして提供しています。</span><span class="sxs-lookup"><span data-stu-id="c051e-104">Microsoft provides Azure and Azure Stack products and solutions as one consistent Azure ecosystem.</span></span> <span data-ttu-id="c051e-105">Microsoft Azure Stack ファミリは Azure の拡張機能です。</span><span class="sxs-lookup"><span data-stu-id="c051e-105">The Microsoft Azure Stack family is an extension of Azure.</span></span>

## <a name="the-hybrid-cloud-and-hybrid-apps"></a><span data-ttu-id="c051e-106">ハイブリッド クラウドとハイブリッド アプリ</span><span class="sxs-lookup"><span data-stu-id="c051e-106">The hybrid cloud and hybrid apps</span></span>

<span data-ttu-id="c051e-107">Azure Stack は、"*ハイブリッド クラウド*" を有効にすることにより、オンプレミス環境とエッジにクラウド コンピューティングの機敏性を提供します。</span><span class="sxs-lookup"><span data-stu-id="c051e-107">Azure Stack brings the agility of cloud computing to your on-premises environment and the edge by enabling a *hybrid cloud*.</span></span> <span data-ttu-id="c051e-108">Azure Stack Hub、Azure Stack HCI、Azure Stack Edge により、Azure がクラウドから専用のデータセンター、支店、現場などにまで拡張されます。</span><span class="sxs-lookup"><span data-stu-id="c051e-108">Azure Stack Hub, Azure Stack HCI, and Azure Stack Edge extend Azure from the cloud into your sovereign datacenters, branch offices, field, and beyond.</span></span> <span data-ttu-id="c051e-109">このさまざまな機能セットを使用すると、次のことが可能になります。</span><span class="sxs-lookup"><span data-stu-id="c051e-109">With this diverse set of capabilities, you can:</span></span>

- <span data-ttu-id="c051e-110">コードを再利用し、Azure とオンプレミス環境全体でクラウド ネイティブ アプリを一貫した方法で実行する。</span><span class="sxs-lookup"><span data-stu-id="c051e-110">Reuse code and run cloud-native apps consistently across Azure and your on-premises environments.</span></span>
- <span data-ttu-id="c051e-111">Azure サービスへのオプション接続を使用して、従来の仮想化されたワークロードを実行する。</span><span class="sxs-lookup"><span data-stu-id="c051e-111">Run traditional virtualized workloads with optional connections to Azure services.</span></span>
- <span data-ttu-id="c051e-112">データをクラウドに転送するか、専用のデータセンターに保管してコンプライアンスを維持する。</span><span class="sxs-lookup"><span data-stu-id="c051e-112">Transfer data to the cloud, or keep it in your sovereign datacenter to maintain compliance.</span></span>
- <span data-ttu-id="c051e-113">ハードウェア アクセラレータによる機械学習、コンテナー化、または仮想化されたワークロードをすべてインテリジェント エッジで実行する。</span><span class="sxs-lookup"><span data-stu-id="c051e-113">Run hardware-accelerated machine-learning, containerized, or virtualized workloads, all at the intelligent edge.</span></span>

<span data-ttu-id="c051e-114">クラウドをまたぐアプリは "*ハイブリッド アプリ*" とも呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="c051e-114">Apps that span clouds are also referred to as *hybrid apps*.</span></span> <span data-ttu-id="c051e-115">Azure でハイブリッド クラウド アプリを構築し、接続されている/いないを問わず、あらゆる場所にあるデータセンターにデプロイすることができます。</span><span class="sxs-lookup"><span data-stu-id="c051e-115">You can build hybrid cloud apps in Azure and deploy them to your connected or disconnected datacenter located anywhere.</span></span>

<span data-ttu-id="c051e-116">ハイブリッド アプリのシナリオは、開発に使用できるリソースによって大きく異なります。</span><span class="sxs-lookup"><span data-stu-id="c051e-116">Hybrid app scenarios vary greatly with the resources that are available for development.</span></span> <span data-ttu-id="c051e-117">また、地理、セキュリティ、インターネット アクセスなどの考慮事項も存在します。</span><span class="sxs-lookup"><span data-stu-id="c051e-117">They also span considerations such as geography, security, internet access, and others.</span></span> <span data-ttu-id="c051e-118">ここで説明するソリューションのパターンと例は、すべての要件に対応しているとは限りませんが、ハイブリッド ソリューションを実装する際に探索して再利用するためのガイドラインと例を提供します。</span><span class="sxs-lookup"><span data-stu-id="c051e-118">Although the solution patterns and examples described here may not address all requirements, they provide guidelines and examples to explore and reuse while implementing hybrid solutions.</span></span>

## <a name="solution-patterns"></a><span data-ttu-id="c051e-119">ソリューション パターン</span><span class="sxs-lookup"><span data-stu-id="c051e-119">Solution patterns</span></span>

<span data-ttu-id="c051e-120">ソリューション パターンは、実際の顧客シナリオや経験から、一般化された反復可能な設計ガイダンスを選び抜いたものです。</span><span class="sxs-lookup"><span data-stu-id="c051e-120">Solution patterns cull generalized repeatable design guidance, from real world customer scenarios and experiences.</span></span> <span data-ttu-id="c051e-121">パターンは抽象的であるため、さまざまな種類のシナリオや業界に適用できます。</span><span class="sxs-lookup"><span data-stu-id="c051e-121">A pattern is abstract, allowing it to be applicable to different types of scenarios or vertical industries.</span></span> <span data-ttu-id="c051e-122">各パターンにはコンテキストと問題が文書化されており、ソリューションの例の概要を示します。</span><span class="sxs-lookup"><span data-stu-id="c051e-122">Each pattern documents the context and problem, and provides an overview of a solution example.</span></span> <span data-ttu-id="c051e-123">このソリューションの例は、考えられるパターンの実装を示しています。</span><span class="sxs-lookup"><span data-stu-id="c051e-123">The solution example is meant as a possible implementation of the pattern.</span></span>

<span data-ttu-id="c051e-124">パターンの記事には、次の 2 種類があります。</span><span class="sxs-lookup"><span data-stu-id="c051e-124">There are two types of pattern articles:</span></span>

- <span data-ttu-id="c051e-125">単一パターン: 単一の汎用シナリオ向けの設計ガイダンスを提供します。</span><span class="sxs-lookup"><span data-stu-id="c051e-125">Single pattern: provides design guidance for a single general-purpose scenario.</span></span>
- <span data-ttu-id="c051e-126">複数パターン: 複数のパターンの適用を使用する設計ガイダンスを提供します。</span><span class="sxs-lookup"><span data-stu-id="c051e-126">Multi-pattern: provides design guidance where the application of multiple patterns is used.</span></span> <span data-ttu-id="c051e-127">多くの場合、このパターンは、より複雑なシナリオや業界固有の問題を解決するために必要となります。</span><span class="sxs-lookup"><span data-stu-id="c051e-127">This pattern is frequently required for solving more complex scenarios or industry-specific problems.</span></span>

## <a name="solution-deployment-guides"></a><span data-ttu-id="c051e-128">ソリューション デプロイ ガイド</span><span class="sxs-lookup"><span data-stu-id="c051e-128">Solution deployment guides</span></span>

<span data-ttu-id="c051e-129">ステップバイステップのデプロイ ガイドにより、ソリューションの例をデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="c051e-129">Step-by-step deployment guides assist in deploying a solution example.</span></span> <span data-ttu-id="c051e-130">このガイドでは、GitHub の[ソリューション サンプル リポジトリ](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)に格納されているコンパニオン コード サンプルを参照する場合もあります。</span><span class="sxs-lookup"><span data-stu-id="c051e-130">The guide may also refer to a companion code sample, stored in the GitHub [solutions sample repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>

## <a name="next-steps"></a><span data-ttu-id="c051e-131">次のステップ</span><span class="sxs-lookup"><span data-stu-id="c051e-131">Next steps</span></span>

- <span data-ttu-id="c051e-132">製品とソリューションのポートフォリオ全体の詳細について、[Azure Stack ファミリの製品とソリューション](/azure-stack)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="c051e-132">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>
- <span data-ttu-id="c051e-133">目次の「パターン」セクションと「ソリューション デプロイ ガイド」セクションを探索し、それぞれの詳細を参照してください。</span><span class="sxs-lookup"><span data-stu-id="c051e-133">Explore the "Patterns" and the "Solution deployment guides" sections of the TOC to learn more about each.</span></span>
- <span data-ttu-id="c051e-134">「[ハイブリッド アプの設計の考慮事項](overview-app-design-considerations.md)」で、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素を確認します。</span><span class="sxs-lookup"><span data-stu-id="c051e-134">Read about [Hybrid app design considerations](overview-app-design-considerations.md) to review pillars of software quality for designing, deploying, and operating hybrid apps.</span></span>
- <span data-ttu-id="c051e-135">[Azure Stack で開発環境を設定](/azure-stack/user/azure-stack-dev-start)し、Azure Stack で[最初のアプリをデプロイ](/azure-stack/user/azure-stack-dev-start-deploy-app)します。</span><span class="sxs-lookup"><span data-stu-id="c051e-135">[Set up a development environment on Azure Stack](/azure-stack/user/azure-stack-dev-start) and [deploy your first app](/azure-stack/user/azure-stack-dev-start-deploy-app) on Azure Stack.</span></span>
