---
title: Azure と Azure Stack Hub のハイブリッド リレー パターン
description: Azure と Azure Stack Hub のハイブリッド リレー パターンを使用して、ファイアウォールで保護されているエッジ リソースに接続します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911147"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="3edb4-103">ハイブリッド リレー パターン</span><span class="sxs-lookup"><span data-stu-id="3edb4-103">Hybrid relay pattern</span></span>

<span data-ttu-id="3edb4-104">ハイブリッドリレーパターンと Azure Relay を使用して、ファイアウォールで保護されているエッジリソースまたはデバイスに接続する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="3edb4-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="3edb4-105">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="3edb4-105">Context and problem</span></span>

<span data-ttu-id="3edb4-106">多くの場合、エッジ デバイスは、企業のファイアウォールまたは NAT デバイスの背後にあります。</span><span class="sxs-lookup"><span data-stu-id="3edb4-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="3edb4-107">これらはセキュリティで保護されていますが、パブリック クラウド、または他の企業ネットワーク上のエッジ デバイスと通信できない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3edb4-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="3edb4-108">このため、パブリック クラウド内のユーザーに特定のポートおよび機能を安全な方法で公開することが必要になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="3edb4-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="3edb4-109">解決策</span><span class="sxs-lookup"><span data-stu-id="3edb4-109">Solution</span></span>

<span data-ttu-id="3edb4-110">ハイブリッドリレーパターンでは、Azure Relay を使用して、直接通信できない2つのエンドポイント間に Websocket トンネルを確立します。</span><span class="sxs-lookup"><span data-stu-id="3edb4-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="3edb4-111">オンプレミスではないが、オンプレミスのエンドポイントに接続する必要があるデバイスは、パブリック クラウド内のエンドポイントに接続されます。</span><span class="sxs-lookup"><span data-stu-id="3edb4-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="3edb4-112">このエンドポイントは、セキュリティ保護されたチャネルを介して、事前定義されたルートでトラフィックをリダイレクトします。</span><span class="sxs-lookup"><span data-stu-id="3edb4-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="3edb4-113">オンプレミスの環境内にあるエンドポイントはトラフィックを受信し、それを正しい宛先にルーティングします。</span><span class="sxs-lookup"><span data-stu-id="3edb4-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![ハイブリッド リレー パターンのソリューション アーキテクチャ](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="3edb4-115">ハイブリッド リレー パターンがどのように機能するかを次に示します。</span><span class="sxs-lookup"><span data-stu-id="3edb4-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="3edb4-116">デバイスが、事前定義されたポートで、Azure 内の仮想マシン (VM) に接続されます。</span><span class="sxs-lookup"><span data-stu-id="3edb4-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="3edb4-117">トラフィックは、Azure の Azure Relay に転送されます。</span><span class="sxs-lookup"><span data-stu-id="3edb4-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="3edb4-118">Azure Relay への長時間接続が確立されている Azure Stack Hub 上の VM は、トラフィックを受信し、転送先に転送します。</span><span class="sxs-lookup"><span data-stu-id="3edb4-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="3edb4-119">オンプレミスのサービスまたはエンドポイントがこの要求を処理します。</span><span class="sxs-lookup"><span data-stu-id="3edb4-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="3edb4-120">Components</span><span class="sxs-lookup"><span data-stu-id="3edb4-120">Components</span></span>

<span data-ttu-id="3edb4-121">このソリューションでは、次のコンポーネントを使用します。</span><span class="sxs-lookup"><span data-stu-id="3edb4-121">This solution uses the following components:</span></span>

| <span data-ttu-id="3edb4-122">レイヤー</span><span class="sxs-lookup"><span data-stu-id="3edb4-122">Layer</span></span> | <span data-ttu-id="3edb4-123">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="3edb4-123">Component</span></span> | <span data-ttu-id="3edb4-124">説明</span><span class="sxs-lookup"><span data-stu-id="3edb4-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="3edb4-125">Azure</span><span class="sxs-lookup"><span data-stu-id="3edb4-125">Azure</span></span> | <span data-ttu-id="3edb4-126">Azure VM</span><span class="sxs-lookup"><span data-stu-id="3edb4-126">Azure VM</span></span> | <span data-ttu-id="3edb4-127">Azure VM は、オンプレミス リソースに対して、パブリックにアクセス可能なエンドポイントを提供します。</span><span class="sxs-lookup"><span data-stu-id="3edb4-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="3edb4-128">Azure Relay</span><span class="sxs-lookup"><span data-stu-id="3edb4-128">Azure Relay</span></span> | <span data-ttu-id="3edb4-129">[Azure Relay](/azure/azure-relay/)には、Azure vm と AZURE STACK Hub vm 間のトンネルと接続を維持するためのインフラストラクチャが用意されています。</span><span class="sxs-lookup"><span data-stu-id="3edb4-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="3edb4-130">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3edb4-130">Azure Stack Hub</span></span> | <span data-ttu-id="3edb4-131">Compute</span><span class="sxs-lookup"><span data-stu-id="3edb4-131">Compute</span></span> | <span data-ttu-id="3edb4-132">Azure Stack Hub VM は、サーバー側のハイブリッド リレー トンネルを提供します。</span><span class="sxs-lookup"><span data-stu-id="3edb4-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="3edb4-133">ストレージ</span><span class="sxs-lookup"><span data-stu-id="3edb4-133">Storage</span></span> | <span data-ttu-id="3edb4-134">Azure Stack Hub にデプロイされた AKS エンジン クラスターによって、Face API コンテナーを実行するためのスケーラブルで回復性があるエンジンが提供されます。</span><span class="sxs-lookup"><span data-stu-id="3edb4-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="3edb4-135">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="3edb4-135">Issues and considerations</span></span>

<span data-ttu-id="3edb4-136">このソリューションの実装方法を決めるときには、以下の点を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="3edb4-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="3edb4-137">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="3edb4-137">Scalability</span></span>

<span data-ttu-id="3edb4-138">このパターンでは、クライアントとサーバーで 1:1 のポート マッピングのみを使用できます。</span><span class="sxs-lookup"><span data-stu-id="3edb4-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="3edb4-139">たとえば、ポート 80 が Azure エンドポイント上のあるサービス用にトンネリングされている場合、このポートを別のサービスに使用することはできません。</span><span class="sxs-lookup"><span data-stu-id="3edb4-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="3edb4-140">ポート マッピングを適切に計画する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3edb4-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="3edb4-141">トラフィックを処理するには、Azure Relay と Vm を適切にスケーリングする必要があります。</span><span class="sxs-lookup"><span data-stu-id="3edb4-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="3edb4-142">可用性</span><span class="sxs-lookup"><span data-stu-id="3edb4-142">Availability</span></span>

<span data-ttu-id="3edb4-143">これらのトンネルと接続は冗長ではありません。</span><span class="sxs-lookup"><span data-stu-id="3edb4-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="3edb4-144">高可用性を確保するために、エラー チェック コードを実装することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="3edb4-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="3edb4-145">もう1つの方法は、ロードバランサーの背後に Azure Relay 接続された Vm のプールを作成することです。</span><span class="sxs-lookup"><span data-stu-id="3edb4-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="3edb4-146">管理の容易性</span><span class="sxs-lookup"><span data-stu-id="3edb4-146">Manageability</span></span>

<span data-ttu-id="3edb4-147">このソリューションは多数のデバイスと場所にまたがることがあるため、扱いにくくなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3edb4-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="3edb4-148">Azure の IoT サービスにより、新しい場所とデバイスを自動的にオンラインにし、最新の状態に保つことができます。</span><span class="sxs-lookup"><span data-stu-id="3edb4-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="3edb4-149">Security</span><span class="sxs-lookup"><span data-stu-id="3edb4-149">Security</span></span>

<span data-ttu-id="3edb4-150">図のように、このパターンでは、エッジから内部デバイス上のポートに自由にアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="3edb4-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="3edb4-151">内部デバイス上のサービス、またはハイブリッド リレー エンドポイントの前面に、認証メカニズムを追加することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="3edb4-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3edb4-152">次のステップ</span><span class="sxs-lookup"><span data-stu-id="3edb4-152">Next steps</span></span>

<span data-ttu-id="3edb4-153">この記事で紹介したトピックの関連情報:</span><span class="sxs-lookup"><span data-stu-id="3edb4-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="3edb4-154">このパターンでは、Azure Relay を使用します。</span><span class="sxs-lookup"><span data-stu-id="3edb4-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="3edb4-155">詳細については、 [Azure Relay のドキュメント](/azure/azure-relay/)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3edb4-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="3edb4-156">ベスト プラクティスの詳細を確認し、その他の疑問の回答を得るには、「[ハイブリッド アプリケーション設計に関する考慮事項](overview-app-design-considerations.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3edb4-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="3edb4-157">製品とソリューションのポートフォリオ全体の詳細について、[Azure Stack ファミリの製品とソリューション](/azure-stack)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3edb4-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="3edb4-158">ソリューションの例をテストする準備ができたら、[ハイブリッド リレーのソリューション デプロイ ガイド](https://aka.ms/hybridrelaydeployment)に進んでください。</span><span class="sxs-lookup"><span data-stu-id="3edb4-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="3edb4-159">デプロイ ガイドでは、コンポーネントをデプロイしてテストするための詳細な手順について説明します。</span><span class="sxs-lookup"><span data-stu-id="3edb4-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>