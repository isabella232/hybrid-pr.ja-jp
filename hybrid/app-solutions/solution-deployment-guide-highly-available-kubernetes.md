---
title: 高可用性 Kubernetes クラスターを Azure Stack Hub でデプロイする
description: Azure と Azure Stack Hub を使用して高可用性を実現する Kubernetes クラスター ソリューションをデプロイする方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911923"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="47b77-103">高可用性 Kubernetes クラスターを Azure Stack Hub でデプロイする</span><span class="sxs-lookup"><span data-stu-id="47b77-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="47b77-104">この記事では、複数の物理的な場所にある複数の Azure Stack Hub インスタンスにデプロイされた高可用性 Kubernetes クラスター環境を構築する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="47b77-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="47b77-105">このソリューション デプロイ ガイドでは、以下を実行する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="47b77-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="47b77-106">AKS Engine をダウンロードして準備する</span><span class="sxs-lookup"><span data-stu-id="47b77-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="47b77-107">AKS Engine ヘルパー VM に接続する</span><span class="sxs-lookup"><span data-stu-id="47b77-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="47b77-108">Kubernetes クラスターのデプロイ</span><span class="sxs-lookup"><span data-stu-id="47b77-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="47b77-109">Kubernetes クラスターに接続する</span><span class="sxs-lookup"><span data-stu-id="47b77-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="47b77-110">Azure Pipelines を Kubernetes クラスターに接続する</span><span class="sxs-lookup"><span data-stu-id="47b77-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="47b77-111">監視の構成</span><span class="sxs-lookup"><span data-stu-id="47b77-111">Configure monitoring</span></span>
> - <span data-ttu-id="47b77-112">アプリケーションをデプロイする</span><span class="sxs-lookup"><span data-stu-id="47b77-112">Deploy application</span></span>
> - <span data-ttu-id="47b77-113">アプリケーションの自動スケーリングを実行する</span><span class="sxs-lookup"><span data-stu-id="47b77-113">Autoscale application</span></span>
> - <span data-ttu-id="47b77-114">Traffic Manager を構成する</span><span class="sxs-lookup"><span data-stu-id="47b77-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="47b77-115">Kubernetes のアップグレード</span><span class="sxs-lookup"><span data-stu-id="47b77-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="47b77-116">Kubernetes をスケーリングする</span><span class="sxs-lookup"><span data-stu-id="47b77-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="47b77-117">![ハイブリッドの柱](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="47b77-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="47b77-118">Microsoft Azure Stack Hub は Azure の拡張機能です。</span><span class="sxs-lookup"><span data-stu-id="47b77-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="47b77-119">Azure Stack Hub により、オンプレミス環境にクラウド コンピューティングの機敏性とイノベーションがもたらされ、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドが可能になります。</span><span class="sxs-lookup"><span data-stu-id="47b77-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="47b77-120">[ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。</span><span class="sxs-lookup"><span data-stu-id="47b77-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="47b77-121">これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="47b77-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="47b77-122">前提条件</span><span class="sxs-lookup"><span data-stu-id="47b77-122">Prerequisites</span></span>

<span data-ttu-id="47b77-123">このデプロイ ガイドの使用を開始する前に、必ず次のことを行ってください。</span><span class="sxs-lookup"><span data-stu-id="47b77-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="47b77-124">「[高可用性 Kubernetes クラスター パターン](pattern-highly-available-kubernetes.md)」の記事を確認する。</span><span class="sxs-lookup"><span data-stu-id="47b77-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="47b77-125">[コンパニオン GitHub リポジトリ](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)の内容を確認する。この記事で参照されている追加アセットが含まれています。</span><span class="sxs-lookup"><span data-stu-id="47b77-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="47b77-126">少なくとも ["共同作成者" アクセス許可](/azure-stack/user/azure-stack-manage-permissions)を使用して [Azure Stack Hub ユーザー ポータル](/azure-stack/user/azure-stack-use-portal)にアクセスできるアカウントを持っている。</span><span class="sxs-lookup"><span data-stu-id="47b77-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="47b77-127">AKS Engine をダウンロードして準備する</span><span class="sxs-lookup"><span data-stu-id="47b77-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="47b77-128">AKS Engine は、Azure Stack Hub Azure Resource Manager エンドポイントに接続できるすべての Windows または Linux ホストから使用できるバイナリです。</span><span class="sxs-lookup"><span data-stu-id="47b77-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="47b77-129">このガイドでは、Azure Stack Hub で新しい Linux (または Windows) VM をデプロイする方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="47b77-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="47b77-130">それは、後で AKS Engine によって Kubernetes クラスターがデプロイされるときに使用されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="47b77-131">既存の Windows または Linux VM を使用して、AKS Engine を使用する Azure Stack Hub で Kubernetes クラスターをデプロイすることもできます。</span><span class="sxs-lookup"><span data-stu-id="47b77-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="47b77-132">AKS Engine のステップバイステップ プロセスと要件については、次のドキュメントを参照してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="47b77-133">[Azure Stack Hub で AKS Engine を Linux にインストールする](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (または [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows) を使用する)</span><span class="sxs-lookup"><span data-stu-id="47b77-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="47b77-134">AKS Engine は、(アンマネージド) Kubernetes クラスターを (Azure と Azure Stack Hub で) デプロイして運用するためのヘルパー ツールです。</span><span class="sxs-lookup"><span data-stu-id="47b77-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="47b77-135">Azure Stack Hub の AKS Engine の詳細と相違点については、次を参照してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="47b77-136">Azure Stack Hub の AKS Engine とは</span><span class="sxs-lookup"><span data-stu-id="47b77-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="47b77-137">[Azure Stack Hub の AKS Engine](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (GitHub)</span><span class="sxs-lookup"><span data-stu-id="47b77-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="47b77-138">サンプル環境では、Terraform を使用して、AKS Engine VM のデプロイを自動化します。</span><span class="sxs-lookup"><span data-stu-id="47b77-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="47b77-139">[コンパニオン GitHub リポジトリで詳細とコード](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md)を確認できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="47b77-140">この手順の結果は、AKS Engine ヘルパー VM と関連リソースを含む Azure Stack Hub の新しいリソース グループになります。</span><span class="sxs-lookup"><span data-stu-id="47b77-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![Azure Stack Hub の AKS Engine VM リソース](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="47b77-142">接続されていないエアギャップされた環境に AKS Engine をデプロイする必要がある場合は、[接続されていない Azure Stack Hub インスタンス](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances)に関する記事で詳細を確認してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="47b77-143">次の手順では、新しくデプロイされた AKS Engine VM を使用して Kubernetes クラスターをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="47b77-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="47b77-144">AKS Engine ヘルパー VM に接続する</span><span class="sxs-lookup"><span data-stu-id="47b77-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="47b77-145">最初に、先ほど作成した AKS Engine ヘルパー VM に接続する必要があります。</span><span class="sxs-lookup"><span data-stu-id="47b77-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="47b77-146">VM にはパブリック IP アドレスが必要であり、SSH (ポート 22/TCP) を介してアクセスできる必要があります。</span><span class="sxs-lookup"><span data-stu-id="47b77-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![AKS Engine VM の概要ページ](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="47b77-148">Windows 10 の MobaXterm、puTTY、PowerShell などの任意のツールで、SSH を使用して Linux VM に接続できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="47b77-149">接続した後、コマンド `aks-engine` を実行します。</span><span class="sxs-lookup"><span data-stu-id="47b77-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="47b77-150">AKS Engine と Kubernetes のバージョンの詳細については、[サポートされている AKS Engine のバージョン](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions)に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![aks-engine コマンド ラインの例](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="47b77-152">Kubernetes クラスターのデプロイ</span><span class="sxs-lookup"><span data-stu-id="47b77-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="47b77-153">AKS Engine ヘルパー VM 自体には、まだ Azure Stack Hub で Kubernetes クラスターが作成されていません。</span><span class="sxs-lookup"><span data-stu-id="47b77-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="47b77-154">クラスターの作成は、AKS Engine ヘルパー VM で実行される最初のアクションです。</span><span class="sxs-lookup"><span data-stu-id="47b77-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="47b77-155">ステップバイステップ プロセスについては、次のドキュメントを参照してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="47b77-156">AKS エンジンを使用して Azure Stack Hub に Kubernetes クラスターをデプロイする</span><span class="sxs-lookup"><span data-stu-id="47b77-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="47b77-157">`aks-engine deploy` コマンドと前の手順で行った準備の最終的な結果は、最初の Azure Stack Hub インスタンスのテナント空間にデプロイされた完全に機能する Kubernetes クラスターです。</span><span class="sxs-lookup"><span data-stu-id="47b77-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="47b77-158">クラスター自体は、VM、ロード バランサー、VNet、ディスクなどの Azure IaaS コンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Azure Stack Hub ポータルでのクラスターの IaaS コンポーネント](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="47b77-160">Azure ロード バランサー (K8s API エンドポイント)</span><span class="sxs-lookup"><span data-stu-id="47b77-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="47b77-161">ワーカー ノード (エージェント プール)</span><span class="sxs-lookup"><span data-stu-id="47b77-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="47b77-162">マスター ノード</span><span class="sxs-lookup"><span data-stu-id="47b77-162">Master Nodes</span></span>

<span data-ttu-id="47b77-163">これでクラスターが稼働状態になったので、次の手順でそれに接続します。</span><span class="sxs-lookup"><span data-stu-id="47b77-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="47b77-164">Kubernetes クラスターに接続する</span><span class="sxs-lookup"><span data-stu-id="47b77-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="47b77-165">先ほど作成した Kubernetes クラスターに、SSH (デプロイの一部として指定された SSH キーを使用) または `kubectl` (推奨) を介して接続できるようになりました。</span><span class="sxs-lookup"><span data-stu-id="47b77-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="47b77-166">Windows、Linux、および macOS 用の Kubernetes コマンドライン ツール `kubectl` は、[こちら](https://kubernetes.io/docs/tasks/tools/install-kubectl/)から入手できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="47b77-167">それは既にクラスターのマスター ノードに事前インストールされて構成されています。</span><span class="sxs-lookup"><span data-stu-id="47b77-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![マスター ノードで kubectl を実行する](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="47b77-169">マスター ノードを管理タスク用のジャンプボックスとして使用することは推奨されません。</span><span class="sxs-lookup"><span data-stu-id="47b77-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="47b77-170">`kubectl` 構成は、マスター ノードと AKS Engine VM の `.kube/config` に格納されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="47b77-171">Kubernetes クラスターに接続される管理マシンに構成をコピーし、そこで `kubectl` コマンドを使用できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="47b77-172">`.kube/config` ファイルは、後で Azure Pipelines 内にサービスを構成するためにも使用されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="47b77-173">このファイルには Kubernetes クラスターの資格情報が含まれているので、常にセキュリティで保護してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="47b77-174">ファイルにアクセスできる攻撃者は、管理者としてアクセスするための十分な情報を持っています。</span><span class="sxs-lookup"><span data-stu-id="47b77-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="47b77-175">最初の `.kube/config` ファイルを使用して実行されるすべてのアクションは、クラスター管理者アカウントを使用して実行されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="47b77-176">ここで、クラスターの状態を確認するさまざまな `kubectl` のコマンドを試してみることができます。</span><span class="sxs-lookup"><span data-stu-id="47b77-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> <span data-ttu-id="47b77-177">Kubernetes には、きめ細かいロールの定義とロールのバインドを作成できる独自の _ *ロールベースのアクセス制御 (RBAC)* \* モデルがあります。</span><span class="sxs-lookup"><span data-stu-id="47b77-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="47b77-178">これは、cluster-admin アクセス許可を渡す代わりに、クラスターへのアクセスを制御するための望ましい方法です。</span><span class="sxs-lookup"><span data-stu-id="47b77-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="47b77-179">Azure Pipelines を Kubernetes クラスターに接続する</span><span class="sxs-lookup"><span data-stu-id="47b77-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="47b77-180">Azure Pipelines を新しくデプロイされた Kubernetes クラスターに接続するには、前の手順で説明した kube config (`.kube/config`) ファイルが必要です。</span><span class="sxs-lookup"><span data-stu-id="47b77-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="47b77-181">Kubernetes クラスターのマスター ノードのいずれかに接続します。</span><span class="sxs-lookup"><span data-stu-id="47b77-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="47b77-182">`.kube/config` ファイルの内容をコピーします。</span><span class="sxs-lookup"><span data-stu-id="47b77-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="47b77-183">[Azure DevOps] > [プロジェクトの設定] > [サービス接続] の順に移動して、新しい "Kubernetes" サービス接続を作成します (KubeConfig を認証方法として使用します)。</span><span class="sxs-lookup"><span data-stu-id="47b77-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="47b77-184">Azure Pipelines (またはそのビルド エージェント) が Kubernetes API にアクセスできる必要があります。</span><span class="sxs-lookup"><span data-stu-id="47b77-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="47b77-185">Azure Pipelines から Azure Stack Hub Kubernetes クラスターへのインターネット接続がある場合は、自己ホスト型 Azure Pipelines ビルド エージェントをデプロイする必要があります。</span><span class="sxs-lookup"><span data-stu-id="47b77-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="47b77-186">Azure Pipelines 用の自己ホスト型エージェントをデプロイする場合は、Azure Stack Hub または必要なすべての管理エンドポイントへのネットワーク接続を備えたコンピューターへのデプロイのいずれかを実行できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="47b77-187">詳細については、次を参照してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-187">See the details here:</span></span>

* <span data-ttu-id="47b77-188">[Windows](/azure/devops/pipelines/agents/v2-windows) または [Linux](/azure/devops/pipelines/agents/v2-linux) 上の [Azure Pipelines エージェント](/azure/devops/pipelines/agents/agents)</span><span class="sxs-lookup"><span data-stu-id="47b77-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="47b77-189">パターンの「[デプロイ (CI/CD) に関する考慮事項](pattern-highly-available-kubernetes.md#deployment-cicd-considerations)」セクションに、Microsoft によってホストされるエージェントと自己ホスト型エージェントのどちらを使用するかを理解するのに役立つ意思決定フローが含まれています。</span><span class="sxs-lookup"><span data-stu-id="47b77-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="47b77-190">[![自己ホスト型エージェントの意思決定フロー](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="47b77-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="47b77-191">このサンプル ソリューションのトポロジには、各 Azure Stack Hub インスタンスに自己ホスト型ビルド エージェントが含まれています。</span><span class="sxs-lookup"><span data-stu-id="47b77-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="47b77-192">エージェントは、Azure Stack Hub 管理エンドポイントと Kubernetes クラスターの API エンドポイントにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="47b77-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="47b77-193">[![送信トラフィックのみ](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="47b77-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="47b77-194">この設計では、アプリケーション ソリューションからの送信接続のみを対象とする一般的な規制要件に対応しています。</span><span class="sxs-lookup"><span data-stu-id="47b77-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="47b77-195">監視の構成</span><span class="sxs-lookup"><span data-stu-id="47b77-195">Configure monitoring</span></span>

<span data-ttu-id="47b77-196">コンテナー用の [Azure Monitor](/azure/azure-monitor/) を使用して、ソリューション内のコンテナーを監視できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="47b77-197">これは、Azure Stack Hub の AKS Engine によってデプロイされた Kubernetes クラスターに対する Azure Monitor を指します。</span><span class="sxs-lookup"><span data-stu-id="47b77-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="47b77-198">クラスターで Azure Monitor を有効にするには、2 つの方法があります。</span><span class="sxs-lookup"><span data-stu-id="47b77-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="47b77-199">どちらの方法でも、Azure 上に Log Analytics ワークスペースを設定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="47b77-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="47b77-200">[方法 1](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) では、Helm Chart が使用されます</span><span class="sxs-lookup"><span data-stu-id="47b77-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="47b77-201">[方法 2](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) は、AKS Engine クラスター仕様の一部です</span><span class="sxs-lookup"><span data-stu-id="47b77-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="47b77-202">サンプル トポロジでは、プロセスを自動化し、更新プログラムを簡単にインストールできる "方法 1" を使用します。</span><span class="sxs-lookup"><span data-stu-id="47b77-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="47b77-203">次の手順では、コンピューター上に Azure LogAnalytics ワークスペース (ID とキー)、`Helm` (バージョン 3)、および `kubectl` が必要です。</span><span class="sxs-lookup"><span data-stu-id="47b77-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="47b77-204">Helm は Kubernetes パッケージ マネージャーであり、macOS、Windows、および Linux で実行されるバイナリとして入手できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="47b77-205">こちらからダウンロードできます: [helm.sh](https://helm.sh/docs/intro/quickstart/)。Helm は、`kubectl` コマンドに使用される Kubernetes 構成ファイルに依存しています。</span><span class="sxs-lookup"><span data-stu-id="47b77-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="47b77-206">このコマンドにより、Kubernetes クラスターに Azure Monitor エージェントがインストールされます。</span><span class="sxs-lookup"><span data-stu-id="47b77-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="47b77-207">Kubernetes クラスターの Operations Management Suite (OMS) エージェントにより、(送信 HTTPS を使用して) Azure Log Analytics ワークスペースに監視データが送信されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="47b77-208">これで、Azure Monitor を使用して、Azure Stack Hub で Kubernetes クラスターに関するより深い分析情報を得ることができます。</span><span class="sxs-lookup"><span data-stu-id="47b77-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="47b77-209">この設計は、アプリケーションのクラスターと共に自動的にデプロイできる分析のパワーを示するための強力な方法です。</span><span class="sxs-lookup"><span data-stu-id="47b77-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="47b77-210">[![Azure Monitor でのAzure Stack Hub クラスター](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="47b77-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="47b77-211">[![Azure Monitor クラスターの詳細](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="47b77-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="47b77-212">Azure Monitor に Azure Stack Hub データが表示されない場合は、「[AzureMonitor-Containers ソリューションを Azure Loganalytics ワークスペースに追加する方法](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md)」の手順に従っていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="47b77-213">アプリケーションの配置</span><span class="sxs-lookup"><span data-stu-id="47b77-213">Deploy the application</span></span>

<span data-ttu-id="47b77-214">サンプル アプリケーションをインストールする前に、Kubernetes クラスターに nginx ベースのイングレス コントローラーを構成する別の手順があります。</span><span class="sxs-lookup"><span data-stu-id="47b77-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="47b77-215">イングレス コントローラーは、ホスト、パス、またはプロトコルに基づいてクラスター内のトラフィックをルーティングするためのレイヤー 7 のロード バランサーとして使用されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="47b77-216">nginx-ingress は、Helm Chart として使用できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="47b77-217">詳細な手順については、[Helm Chart GitHub リポジトリ](https://github.com/helm/charts/tree/master/stable/nginx-ingress)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="47b77-218">さらに、サンプル アプリケーションは、前の手順の [Azure Monitoring エージェント](#configure-monitoring)のように Helm Chart としてパッケージ化されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="47b77-219">そのため、Kubernetes クラスターへのアプリケーションのデプロイを簡単に実行できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="47b77-220">[コンパニオン GitHub リポジトリで Helm Chart](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm) を確認できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="47b77-221">サンプル アプリケーションは、2 つの Azure Stack Hub インスタンスの Kubernetes クラスターにデプロイされる 3 層アプリケーションです。</span><span class="sxs-lookup"><span data-stu-id="47b77-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="47b77-222">アプリケーションでは、MongoDB データベースが使用されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="47b77-223">複数のインスタンスにレプリケートされたデータを取得する方法の詳細については、パターンの「[データとストレージに関する考慮事項](pattern-highly-available-kubernetes.md#data-and-storage-considerations)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="47b77-224">アプリケーション用の Helm Chart をデプロイした後、単一のポッド、デプロイ、およびステートフル セット (データベース用) として表されたアプリケーションの 3 つの層のすべてを表示できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

<span data-ttu-id="47b77-225">サービス側では、nginx ベースのイングレス コントローラーとそのパブリック IP アドレスを確認できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

<span data-ttu-id="47b77-226">"外部 IP" アドレスが "アプリケーション エンドポイント" です。</span><span class="sxs-lookup"><span data-stu-id="47b77-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="47b77-227">それはユーザーがアプリケーションを開くために接続する方法であり、次の手順の「[Traffic Manager を構成する](#configure-traffic-manager)」でエンドポイントとしても使用されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="47b77-228">アプリケーションの自動スケーリングを行う</span><span class="sxs-lookup"><span data-stu-id="47b77-228">Autoscale the application</span></span>
<span data-ttu-id="47b77-229">必要に応じて [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) を構成して、CPU 使用率などの特定のメトリックに基づくスケールアップまたはスケールダウンを実行できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="47b77-230">次のコマンドを使用すると、ratings-web デプロイによって制御されるポッドの 1 から 10 個のレプリカを保持する Horizontal Pod Autoscaler が作成されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="47b77-231">HPA によって、すべてのポッドの平均 CPU 使用率を 80% に保つために、レプリカの数が (デプロイを介して) 増減されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="47b77-232">以下を実行することで、オートスケーラーの現在の状態を確認できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="47b77-233">Traffic Manager を構成する</span><span class="sxs-lookup"><span data-stu-id="47b77-233">Configure Traffic Manager</span></span>

<span data-ttu-id="47b77-234">アプリケーションの 2 つ (以上) のデプロイ間でトラフィックを分散するために、[Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) を使用します。</span><span class="sxs-lookup"><span data-stu-id="47b77-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="47b77-235">Azure Traffic Manager は、Azure 内の DNS ベースのトラフィック ロード バランサーです。</span><span class="sxs-lookup"><span data-stu-id="47b77-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="47b77-236">Traffic Manager では、DNS を使用して、トラフィック ルーティング方法とエンドポイントの正常性に基づいて最適なサービス エンドポイントにクライアント要求が送信されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="47b77-237">Azure Traffic Manager を使用する代わりに、オンプレミスでホストされている他のグローバルな負荷分散ソリューションを使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="47b77-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="47b77-238">サンプル シナリオでは、Azure Traffic Manager を使用して、アプリケーションの 2 つのインスタンス間にトラフィックを分散します。</span><span class="sxs-lookup"><span data-stu-id="47b77-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="47b77-239">同じ場所または異なる場所にある Azure Stack Hub インスタンスで実行できます。</span><span class="sxs-lookup"><span data-stu-id="47b77-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![オンプレミスのトラフィック マネージャー](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="47b77-241">Azure で、アプリケーションの 2 つの異なるインスタンスをポイントするように Traffic Manager を構成します。</span><span class="sxs-lookup"><span data-stu-id="47b77-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="47b77-242">[![TM エンドポイント プロファイル](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="47b77-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="47b77-243">ご覧のように、2 つのエンドポイントによって、[前のセクション](#deploy-the-application)でデプロイされたアプリケーションの 2 つのインスタンスがポイントされています。</span><span class="sxs-lookup"><span data-stu-id="47b77-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="47b77-244">この時点で:</span><span class="sxs-lookup"><span data-stu-id="47b77-244">At this point:</span></span>
- <span data-ttu-id="47b77-245">Kubernetes インフラストラクチャが作成されています (イングレス コントローラーが含まれています)。</span><span class="sxs-lookup"><span data-stu-id="47b77-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="47b77-246">クラスターは 2 つの Azure Stack Hub インスタンスにデプロイされています。</span><span class="sxs-lookup"><span data-stu-id="47b77-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="47b77-247">監視が構成されています。</span><span class="sxs-lookup"><span data-stu-id="47b77-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="47b77-248">Azure Traffic Manager によって、2 つの Azure Stack Hub インスタンスにトラフィックが負荷分散されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="47b77-249">インフラストラクチャの上部に、3 層のサンプル アプリケーションが、Helm Charts を使用して自動化された方法でデプロイされています。</span><span class="sxs-lookup"><span data-stu-id="47b77-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="47b77-250">これで、ソリューションが起動し、ユーザーがアクセスできるようになります。</span><span class="sxs-lookup"><span data-stu-id="47b77-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="47b77-251">検討する価値があるデプロイ後の運用上の考慮事項がいくつかあります。それらについて、次の 2 つのセクションで説明します。</span><span class="sxs-lookup"><span data-stu-id="47b77-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="47b77-252">Kubernetes のアップグレード</span><span class="sxs-lookup"><span data-stu-id="47b77-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="47b77-253">Kubernetes クラスターをアップグレードするときは、次のトピックを考慮してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="47b77-254">Kubernetes クラスターのアップグレードは、AKS Engine を使用して実行できる複雑な運用段階の操作です。</span><span class="sxs-lookup"><span data-stu-id="47b77-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="47b77-255">詳細については、「[Azure Stack Hub で Kubernetes クラスターをアップグレードする](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="47b77-256">AKS Engine を使用して、クラスターの Kubernetes とベース OS イメージを新しいバージョンにアップグレードできます。</span><span class="sxs-lookup"><span data-stu-id="47b77-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="47b77-257">詳細については、「[新しい Kubernetes バージョンにアップグレードする手順](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="47b77-258">基礎になるノードだけをベース OS イメージの新しいバージョンにアップグレードすることもできます。</span><span class="sxs-lookup"><span data-stu-id="47b77-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="47b77-259">詳細については、「[OS イメージのみをアップグレードする手順](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="47b77-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="47b77-260">新しいベース OS イメージには、セキュリティとカーネルの更新プログラムが含まれています。</span><span class="sxs-lookup"><span data-stu-id="47b77-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="47b77-261">新しい Kubernetes バージョンと OS イメージの可用性を監視するのはクラスター オペレーターの責任です。</span><span class="sxs-lookup"><span data-stu-id="47b77-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="47b77-262">オペレーターは、AKS Engine を使用するこれらのアップグレードを計画して実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="47b77-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="47b77-263">ベース OS イメージは、Azure Stack Hub オペレーターが Azure Stack Hub Marketplace からダウンロードする必要があります。</span><span class="sxs-lookup"><span data-stu-id="47b77-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="47b77-264">Kubernetes をスケーリングする</span><span class="sxs-lookup"><span data-stu-id="47b77-264">Scale Kubernetes</span></span>

<span data-ttu-id="47b77-265">スケーリングは、AKS Engine を使用して調整できる別の運用段階の操作です。</span><span class="sxs-lookup"><span data-stu-id="47b77-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="47b77-266">スケーリング コマンドでは、出力ディレクトリ内のクラスター構成ファイル (apimodel.json) が、新しい Azure Resource Manager デプロイの入力として再利用されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="47b77-267">AKS Engine により、特定のエージェント プールに対してスケーリング操作が実行されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="47b77-268">スケーリング操作が完了すると、AKS Engine により、同じ apimodel. json ファイル内のクラスター定義が更新されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="47b77-269">クラスター定義には、更新された最新のクラスター構成を反映するために、新しいノード数が反映されます。</span><span class="sxs-lookup"><span data-stu-id="47b77-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="47b77-270">Azure Stack Hub で Kubernetes クラスターをスケールする</span><span class="sxs-lookup"><span data-stu-id="47b77-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="47b77-271">次のステップ</span><span class="sxs-lookup"><span data-stu-id="47b77-271">Next steps</span></span>

- <span data-ttu-id="47b77-272">[ハイブリッド アプリの設計上の考慮事項](overview-app-design-considerations.md)の詳細を確認する</span><span class="sxs-lookup"><span data-stu-id="47b77-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="47b77-273">[GitHub でこのサンプルのコード](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)を確認し、改善点を提案する</span><span class="sxs-lookup"><span data-stu-id="47b77-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>