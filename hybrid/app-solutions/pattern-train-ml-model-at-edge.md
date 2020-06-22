---
title: エッジ パターンで機械学習モデルをトレーニングする
description: Azure と Azure Stack Hub を使用してエッジで機械学習モデルをトレーニングする方法について確認してください。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911100"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="0d40d-103">エッジ パターンで機械学習モデルをトレーニングする</span><span class="sxs-lookup"><span data-stu-id="0d40d-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="0d40d-104">オンプレミスにのみ存在するデータから移植可能な機械学習 (ML) モデルを生成します。</span><span class="sxs-lookup"><span data-stu-id="0d40d-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="0d40d-105">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="0d40d-105">Context and problem</span></span>

<span data-ttu-id="0d40d-106">データ サイエンティストが理解するツールを利用し、オンプレミスまたはレガシ データから分析情報を解き明かすことを望む組織はたくさん存在します。</span><span class="sxs-lookup"><span data-stu-id="0d40d-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="0d40d-107">[Azure Machine Learning](/azure/machine-learning/) では、ML とディープ ラーニング モデルをトレーニング、チューニング、デプロイするためのクラウドネイティブ ツールが提供されます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="0d40d-108">しかしながら、一部のデータは大きすぎて、あるいは、規制上の理由から、クラウドに送信できません。</span><span class="sxs-lookup"><span data-stu-id="0d40d-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="0d40d-109">このパターンを使用すると、データ サイエンティストは Azure Machine Learning を利用し、オンプレミス データとコンピューティングでモデルをトレーニングできます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="0d40d-110">解決策</span><span class="sxs-lookup"><span data-stu-id="0d40d-110">Solution</span></span>

<span data-ttu-id="0d40d-111">エッジ パターンのトレーニングでは、Azure Stack Hub で実行されている仮想マシン (VM) が使用されます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="0d40d-112">VM は Azure ML でコンピューティング先として登録され、オンプレミスでのみ利用できるデータにアクセスすることが許可されます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="0d40d-113">この場合、データは Azure Stack Hub の BLOB ストレージに保存されます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="0d40d-114">モデルがトレーニングされると、Azure ML に登録され、コンテナー化され、Azure Container Registry にデプロイのために追加されます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="0d40d-115">このパターンを反復するためには、Azure Stack Hub トレーニング VM に公共のインターネット経由でアクセスできる必要があります。</span><span class="sxs-lookup"><span data-stu-id="0d40d-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="0d40d-116">[![エッジ アーキテクチャで ML モデルをトレーニングする](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="0d40d-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="0d40d-117">パターンのしくみは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="0d40d-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="0d40d-118">Azure Stack Hub VM がデプロイされ、Azure ML にコンピューティング先として登録されます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="0d40d-119">Azure ML で、コンピューティング先として Azure Stack Hub VM を使用する実験が作成されます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="0d40d-120">トレーニングされたモデルは登録され、コンテナー化されます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="0d40d-121">これで、オンプレミスまたはクラウドにある場所にモデルをデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="0d40d-122">Components</span><span class="sxs-lookup"><span data-stu-id="0d40d-122">Components</span></span>

<span data-ttu-id="0d40d-123">このソリューションでは、次のコンポーネントを使用します。</span><span class="sxs-lookup"><span data-stu-id="0d40d-123">This solution uses the following components:</span></span>

| <span data-ttu-id="0d40d-124">レイヤー</span><span class="sxs-lookup"><span data-stu-id="0d40d-124">Layer</span></span> | <span data-ttu-id="0d40d-125">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="0d40d-125">Component</span></span> | <span data-ttu-id="0d40d-126">説明</span><span class="sxs-lookup"><span data-stu-id="0d40d-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="0d40d-127">Azure</span><span class="sxs-lookup"><span data-stu-id="0d40d-127">Azure</span></span> | <span data-ttu-id="0d40d-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="0d40d-128">Azure Machine Learning</span></span> | <span data-ttu-id="0d40d-129">[Azure Machine Learning](/azure/machine-learning/) により ML モデルのトレーニングが調整されます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="0d40d-130">Azure Container Registry</span><span class="sxs-lookup"><span data-stu-id="0d40d-130">Azure Container Registry</span></span> | <span data-ttu-id="0d40d-131">Azure ML によりモデルがパッケージ化され、コンテナーが作られ、デプロイのために [Azure Container Registry](/azure/container-registry/) に格納されます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="0d40d-132">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="0d40d-132">Azure Stack Hub</span></span> | <span data-ttu-id="0d40d-133">App Service</span><span class="sxs-lookup"><span data-stu-id="0d40d-133">App Service</span></span> | <span data-ttu-id="0d40d-134">[Azure Stack Hub と App Service](/azure-stack/operator/azure-stack-app-service-overview) は末端のコンポーネントの基礎となります。</span><span class="sxs-lookup"><span data-stu-id="0d40d-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="0d40d-135">Compute</span><span class="sxs-lookup"><span data-stu-id="0d40d-135">Compute</span></span> | <span data-ttu-id="0d40d-136">Ubuntu と Docker を実行する Azure Stack Hub VM は ML モデルのトレーニングに使用されます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="0d40d-137">ストレージ</span><span class="sxs-lookup"><span data-stu-id="0d40d-137">Storage</span></span> | <span data-ttu-id="0d40d-138">非公開データは Azure Stack Hub BLOB ストレージでホストできます。</span><span class="sxs-lookup"><span data-stu-id="0d40d-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="0d40d-139">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="0d40d-139">Issues and considerations</span></span>

<span data-ttu-id="0d40d-140">このソリューションの実装方法を決めるときには、以下の点を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="0d40d-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="0d40d-141">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="0d40d-141">Scalability</span></span>

<span data-ttu-id="0d40d-142">このソリューションをスケーリングするには、トレーニングのために Azure Stack Hub で適切なサイズの VM を作成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="0d40d-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="0d40d-143">可用性</span><span class="sxs-lookup"><span data-stu-id="0d40d-143">Availability</span></span>

<span data-ttu-id="0d40d-144">トレーニング スクリプトと Azure Stack Hub VM でトレーニングに使用されるオンプレミス データにアクセスできることを確認します。</span><span class="sxs-lookup"><span data-stu-id="0d40d-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="0d40d-145">管理の容易性</span><span class="sxs-lookup"><span data-stu-id="0d40d-145">Manageability</span></span>

<span data-ttu-id="0d40d-146">モデル デプロイ中の混乱を避けるため、モデルと実験が正しく登録され、バージョン管理され、タグ付けされていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="0d40d-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="0d40d-147">Security</span><span class="sxs-lookup"><span data-stu-id="0d40d-147">Security</span></span>

<span data-ttu-id="0d40d-148">このパターンを使用することで、Azure ML からオンプレミスの機密性が高い可能性のあるデータにアクセスできるようになります。</span><span class="sxs-lookup"><span data-stu-id="0d40d-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="0d40d-149">Azure Stack Hub VM に SSH 接続するためのアカウントに強固なパスワードが与えられていることとトレーニング スクリプトではデータが保存されず、クラウドにアップロードされないことを確認します。</span><span class="sxs-lookup"><span data-stu-id="0d40d-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="0d40d-150">次のステップ</span><span class="sxs-lookup"><span data-stu-id="0d40d-150">Next steps</span></span>

<span data-ttu-id="0d40d-151">この記事で紹介したトピックの関連情報:</span><span class="sxs-lookup"><span data-stu-id="0d40d-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="0d40d-152">ML と関連トピックの概要については、[Azure Machine Learning のドキュメント](/azure/machine-learning)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0d40d-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="0d40d-153">コンテナー デプロイ向けのイメージを作成、格納、管理する方法については、「[Azure Container Registry](/azure/container-registry/)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0d40d-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="0d40d-154">リソース プロバイダーとデプロイ方法の詳細については、「[Azure Stack 上の App Service の概要](/azure-stack/operator/azure-stack-app-service-overview)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0d40d-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="0d40d-155">ベスト プラクティスの詳細を確認し、その他の疑問の回答を取得するには、[ハイブリッド アプリケーションの設計上の考慮事項](overview-app-design-considerations.md)に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0d40d-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="0d40d-156">製品とソリューションのポートフォリオ全体の詳細について、[Azure Stack ファミリの製品とソリューション](/azure-stack)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0d40d-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="0d40d-157">ソリューションの例をテストする準備ができたら、[エッジ デプロイの ML モデル トレーニング ガイド](https://aka.ms/edgetrainingdeploy)に進んでください。</span><span class="sxs-lookup"><span data-stu-id="0d40d-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="0d40d-158">デプロイ ガイドでは、コンポーネントをデプロイしてテストするための詳細な手順について説明します。</span><span class="sxs-lookup"><span data-stu-id="0d40d-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
