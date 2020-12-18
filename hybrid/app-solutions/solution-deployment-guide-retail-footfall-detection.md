---
title: Azure と Azure Stack Hub で AI ベースの足取り検出ソリューションをデプロイする
description: Azure と Azure Stack Hub を使用して小売店の訪問者トラフィックを分析するための AI ベースの足取り検出ソリューションをデプロイする方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2177b32474dea695967e197acbd4bc1e18422d7b
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901492"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="c5f74-103">Azure と Azure Stack Hub を使用して AI ベースの足取り検出ソリューションをデプロイする</span><span class="sxs-lookup"><span data-stu-id="c5f74-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="c5f74-104">この記事では、Azure、Azure Stack Hub、Custom Vision AI Dev Kit を使用して、実際のアクションから分析情報を生成する AI ベースのソリューションをデプロイする方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="c5f74-105">このソリューションでは、次の方法を学習します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="c5f74-106">エッジに Cloud Native Application Bundles (CNAB) をデプロイする。</span><span class="sxs-lookup"><span data-stu-id="c5f74-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="c5f74-107">クラウドの境界をまたぐアプリをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="c5f74-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="c5f74-108">エッジでの推論に Custom Vision AI Dev Kit を使用する。</span><span class="sxs-lookup"><span data-stu-id="c5f74-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="c5f74-109">![ハイブリッドの柱の図](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="c5f74-109">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="c5f74-110">Microsoft Azure Stack Hub は Azure の拡張機能です。</span><span class="sxs-lookup"><span data-stu-id="c5f74-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="c5f74-111">Azure Stack Hub により、オンプレミス環境にクラウド コンピューティングの機敏性とイノベーションがもたらされ、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドが可能になります。</span><span class="sxs-lookup"><span data-stu-id="c5f74-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="c5f74-112">[ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。</span><span class="sxs-lookup"><span data-stu-id="c5f74-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="c5f74-113">これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="c5f74-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="c5f74-114">前提条件</span><span class="sxs-lookup"><span data-stu-id="c5f74-114">Prerequisites</span></span>

<span data-ttu-id="c5f74-115">このデプロイ ガイドの使用を開始する前に、必ず次のことを行ってください。</span><span class="sxs-lookup"><span data-stu-id="c5f74-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="c5f74-116">「[足取り検出パターン](pattern-retail-footfall-detection.md)」トピックを確認します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="c5f74-117">次のものがある、Azure Stack Development Kit (ASDK) または Azure Stack Hub 統合システム インスタンスへのユーザー アクセス権を取得します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="c5f74-118">[Azure App Service on Azure Stack Hub リソース プロバイダー](/azure-stack/operator/azure-stack-app-service-overview.md)がインストールされている。</span><span class="sxs-lookup"><span data-stu-id="c5f74-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview.md) installed.</span></span> <span data-ttu-id="c5f74-119">インストールには、Azure Stack Hub インスタンスへのオペレーター アクセス、または管理者と協力する必要があります。</span><span class="sxs-lookup"><span data-stu-id="c5f74-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="c5f74-120">App Service とストレージ クォータを提供するオファーに対するサブスクリプション。</span><span class="sxs-lookup"><span data-stu-id="c5f74-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="c5f74-121">オファーを作成するには、オペレーター アクセスが必要です。</span><span class="sxs-lookup"><span data-stu-id="c5f74-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="c5f74-122">Azure サブスクリプションへのアクセスを取得します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="c5f74-123">Azure サブスクリプションがない場合は、始める前に[無料試用版アカウント](https://azure.microsoft.com/free/)にサインアップしてください。</span><span class="sxs-lookup"><span data-stu-id="c5f74-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="c5f74-124">ディレクトリに 2 つのサービス プリンシパルを作成する</span><span class="sxs-lookup"><span data-stu-id="c5f74-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="c5f74-125">1 つは、Azure サブスクリプションのスコープでアクセスする、Azure リソースで使用するように設定します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="c5f74-126">1 つは、Azure Stack Hub サブスクリプションのスコープでアクセスする、Azure Stack Hub リソースで使用するように設定します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="c5f74-127">サービス プリンシパルの作成とアクセスの承認の詳細については、[アプリ ID を使用してリソースにアクセスする](/azure-stack/operator/azure-stack-create-service-principals.md)に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="c5f74-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md).</span></span> <span data-ttu-id="c5f74-128">Azure CLI を使用する場合は、「[Azure CLI で Azure サービス プリンシパルを作成する](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="c5f74-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span></span>
- <span data-ttu-id="c5f74-129">Azure Cognitive Services を Azure または Azure Stack Hub にデプロイします。</span><span class="sxs-lookup"><span data-stu-id="c5f74-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="c5f74-130">まず、[Cognitive Services の詳細について説明します](https://azure.microsoft.com/services/cognitive-services/)。</span><span class="sxs-lookup"><span data-stu-id="c5f74-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="c5f74-131">次に、「[Azure Cognitive Services を Azure Stack にデプロイする](/azure-stack/user/azure-stack-solution-template-cognitive-services.md)」にアクセスして、Azure Stack Hub 上に Cognitive Services をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="c5f74-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="c5f74-132">プレビューにアクセスするには、まず、サインアップする必要があります。</span><span class="sxs-lookup"><span data-stu-id="c5f74-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="c5f74-133">未構成の Azure Custom Vision AI Dev Kit をクローンまたはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="c5f74-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="c5f74-134">詳細については、[Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="c5f74-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="c5f74-135">Power BI アカウントにサインアップする。</span><span class="sxs-lookup"><span data-stu-id="c5f74-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="c5f74-136">Azure Cognitive Services Face API サブスクリプション キーとエンドポイント URL。</span><span class="sxs-lookup"><span data-stu-id="c5f74-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="c5f74-137">どちらも「[Cognitive Services を試す](https://azure.microsoft.com/try/cognitive-services/?api=face-api)」の無料試用版を使用して取得できます。</span><span class="sxs-lookup"><span data-stu-id="c5f74-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="c5f74-138">または、[Cognitive Services アカウントの作成](/azure/cognitive-services/cognitive-services-apis-create-account)の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="c5f74-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="c5f74-139">次の開発リソースをインストールする。</span><span class="sxs-lookup"><span data-stu-id="c5f74-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="c5f74-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="c5f74-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [<span data-ttu-id="c5f74-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="c5f74-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="c5f74-142">[Porter](https://porter.sh/)。</span><span class="sxs-lookup"><span data-stu-id="c5f74-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="c5f74-143">提供されている CNAB バンドル マニフェストを使用してクラウド アプリをデプロイするには、Porter を使用します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="c5f74-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="c5f74-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="c5f74-145">Visual Studio Code 用の Azure IoT Tools</span><span class="sxs-lookup"><span data-stu-id="c5f74-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="c5f74-146">Visual Studio Code 用の Python 拡張機能</span><span class="sxs-lookup"><span data-stu-id="c5f74-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="c5f74-147">Python</span><span class="sxs-lookup"><span data-stu-id="c5f74-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="c5f74-148">ハイブリッド クラウド アプリをデプロイする</span><span class="sxs-lookup"><span data-stu-id="c5f74-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="c5f74-149">まず、Porter CLI を使用して資格情報セットを生成してから、クラウド アプリをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="c5f74-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="c5f74-150">[https://github.com/azure-samples/azure-intelligent-edge-patterns](https://github.com/azure-samples/azure-intelligent-edge-patterns ) からソリューション サンプル コードを複製するか、ダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="c5f74-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="c5f74-151">Porter によって、アプリのデプロイを自動化する一連の資格情報が生成されます。</span><span class="sxs-lookup"><span data-stu-id="c5f74-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="c5f74-152">資格情報の生成コマンドを実行する前に、次のものを使用できるようにしておく必要があります。</span><span class="sxs-lookup"><span data-stu-id="c5f74-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="c5f74-153">Azure リソースにアクセスするためのサービス プリンシパル (サービス プリンシパル ID、キー、テナント DNS など)。</span><span class="sxs-lookup"><span data-stu-id="c5f74-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="c5f74-154">ご利用の Azure サブスクリプションのサブスクリプション ID。</span><span class="sxs-lookup"><span data-stu-id="c5f74-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="c5f74-155">Azure Stack Hub リソースにアクセスするためのサービス プリンシパル (サービス プリンシパル ID、キー、テナント DNS など)。</span><span class="sxs-lookup"><span data-stu-id="c5f74-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="c5f74-156">ご利用の Azure Stack Hub サブスクリプションのサブスクリプション ID。</span><span class="sxs-lookup"><span data-stu-id="c5f74-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="c5f74-157">ご利用の Azure Cognitive Services Face API キーとリソース エンドポイント URL。</span><span class="sxs-lookup"><span data-stu-id="c5f74-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="c5f74-158">Porter 資格情報生成プロセスを実行し、画面の指示に従います。</span><span class="sxs-lookup"><span data-stu-id="c5f74-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="c5f74-159">Porter では、実行するパラメーターのセットも必要です。</span><span class="sxs-lookup"><span data-stu-id="c5f74-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="c5f74-160">パラメーター テキスト ファイルを作成し、次の名前と値のペアを入力します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="c5f74-161">必要な値についてサポートが必要な場合は、Azure Stack Hub 管理者に問い合わせてください。</span><span class="sxs-lookup"><span data-stu-id="c5f74-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="c5f74-162">`resource suffix` 値は、デプロイのリソースが Azure 全体で一意の名前になることを保証するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="c5f74-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="c5f74-163">これは、8 文字以内の文字と数字の一意の文字列である必要があります。</span><span class="sxs-lookup"><span data-stu-id="c5f74-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   <span data-ttu-id="c5f74-164">テキスト ファイルを保存し、パスをメモしておきます。</span><span class="sxs-lookup"><span data-stu-id="c5f74-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="c5f74-165">これで、Porter を使用してハイブリッド クラウド アプリをデプロイする準備ができました。</span><span class="sxs-lookup"><span data-stu-id="c5f74-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="c5f74-166">インストール コマンドを実行し、Azure と Azure Stack Hub にリソースがデプロイされる様子を確認します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="c5f74-167">デプロイが完了したら、次の値をメモしておきます。</span><span class="sxs-lookup"><span data-stu-id="c5f74-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="c5f74-168">カメラの接続文字列。</span><span class="sxs-lookup"><span data-stu-id="c5f74-168">The camera's connection string.</span></span>
    - <span data-ttu-id="c5f74-169">画像ストレージ アカウントの接続文字列。</span><span class="sxs-lookup"><span data-stu-id="c5f74-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="c5f74-170">リソース グループ名。</span><span class="sxs-lookup"><span data-stu-id="c5f74-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="c5f74-171">Custom Vision AI DevKit を準備する</span><span class="sxs-lookup"><span data-stu-id="c5f74-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="c5f74-172">次に、[Vision AI DevKit のクイックスタート](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/)に示されているように、Custom Vision AI Dev Kit を設定します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="c5f74-173">また、前の手順で指定した接続文字列を使用して、カメラを設定し、テストします。</span><span class="sxs-lookup"><span data-stu-id="c5f74-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="c5f74-174">カメラ アプリをデプロイする</span><span class="sxs-lookup"><span data-stu-id="c5f74-174">Deploy the camera app</span></span>

<span data-ttu-id="c5f74-175">Porter CLI を使用して資格情報セットを生成してから、カメラ アプリをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="c5f74-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="c5f74-176">Porter によって、アプリのデプロイを自動化する一連の資格情報が生成されます。</span><span class="sxs-lookup"><span data-stu-id="c5f74-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="c5f74-177">資格情報の生成コマンドを実行する前に、次のものを使用できるようにしておく必要があります。</span><span class="sxs-lookup"><span data-stu-id="c5f74-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="c5f74-178">Azure リソースにアクセスするためのサービス プリンシパル (サービス プリンシパル ID、キー、テナント DNS など)。</span><span class="sxs-lookup"><span data-stu-id="c5f74-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="c5f74-179">ご利用の Azure サブスクリプションのサブスクリプション ID。</span><span class="sxs-lookup"><span data-stu-id="c5f74-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="c5f74-180">クラウド アプリをデプロイしたときに指定した画像ストレージ アカウントの接続文字列。</span><span class="sxs-lookup"><span data-stu-id="c5f74-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="c5f74-181">Porter 資格情報生成プロセスを実行し、画面の指示に従います。</span><span class="sxs-lookup"><span data-stu-id="c5f74-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="c5f74-182">Porter では、実行するパラメーターのセットも必要です。</span><span class="sxs-lookup"><span data-stu-id="c5f74-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="c5f74-183">パラメーター テキスト ファイルを作成し、次のテキストを入力します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="c5f74-184">わからない必要な値がいくつかある場合は、Azure Stack Hub 管理者に問い合わせてください。</span><span class="sxs-lookup"><span data-stu-id="c5f74-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="c5f74-185">`deployment suffix` 値は、デプロイのリソースが Azure 全体で一意の名前になることを保証するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="c5f74-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="c5f74-186">これは、8 文字以内の文字と数字の一意の文字列である必要があります。</span><span class="sxs-lookup"><span data-stu-id="c5f74-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="c5f74-187">テキスト ファイルを保存し、パスをメモしておきます。</span><span class="sxs-lookup"><span data-stu-id="c5f74-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="c5f74-188">これで、Porter を使用してカメラ アプリをデプロイする準備ができました。</span><span class="sxs-lookup"><span data-stu-id="c5f74-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="c5f74-189">インストール コマンドを実行し、IoT Edge デプロイが作成される様子を確認します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="c5f74-190">`https://<camera-ip>:3000/` でカメラ フィードを表示して、カメラのデプロイが完了したことを確認します。ここで、`<camara-ip>` はカメラの IP アドレスです。</span><span class="sxs-lookup"><span data-stu-id="c5f74-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="c5f74-191">この手順には最大で 10 分かかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="c5f74-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="c5f74-192">Azure Stream Analytics の構成</span><span class="sxs-lookup"><span data-stu-id="c5f74-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="c5f74-193">これでデータがカメラから Azure Stream Analytics に流れるようになりました。Power BI と通信するには、これを手動で承認する必要があります。</span><span class="sxs-lookup"><span data-stu-id="c5f74-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="c5f74-194">Azure portal から、 **[すべてのリソース]** を開き、*process-footfall\[yoursuffix\]* ジョブを開きます。</span><span class="sxs-lookup"><span data-stu-id="c5f74-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="c5f74-195">[Stream Analytics ジョブ] ウィンドウの **[ジョブ トポロジ]** セクションで、 **[出力]** オプションを選択します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="c5f74-196">**traffic-output** 出力シンクを選択します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="c5f74-197">**[承認の更新]** を選択して、自分の Power BI アカウントにサインインします。</span><span class="sxs-lookup"><span data-stu-id="c5f74-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Power BI で承認プロンプトを更新する](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="c5f74-199">出力の設定を保存します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-199">Save the output settings.</span></span>

6. <span data-ttu-id="c5f74-200">**[概要]** ウィンドウにアクセスし、 **[開始]** を選択して Power BI へのデータの送信を開始します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="c5f74-201">ジョブ出力の開始時刻に **[現在]** を選択し、 **[開始]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="c5f74-202">通知バーでジョブの状態を確認できます。</span><span class="sxs-lookup"><span data-stu-id="c5f74-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="c5f74-203">Power BI ダッシュボードを作成する</span><span class="sxs-lookup"><span data-stu-id="c5f74-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="c5f74-204">ジョブが成功したら [Power BI](https://powerbi.com/) に移動し、職場または学校アカウントを使用してサインインします。</span><span class="sxs-lookup"><span data-stu-id="c5f74-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="c5f74-205">Stream Analytics ジョブ クエリによる結果の出力が進行中の場合、作成した *footfall-dataset* データセットは **[データセット]** タブにあります。</span><span class="sxs-lookup"><span data-stu-id="c5f74-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="c5f74-206">Power BI ワークスペースで **[+ 作成]** を選択し、*Footfall Analysis* という名前の新しいダッシュボードを作成します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="c5f74-207">ウィンドウの上部にある **[タイルの追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="c5f74-208">次に、 **[カスタム ストリーミング データ]** と **[次に]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="c5f74-209">**[データセット]** の下で **footfall-dataset** を選択します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="c5f74-210">**[視覚化タイプ]** ドロップダウンで **[カード]** を選択し、**年齢** を **[フィールド]** に追加します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="c5f74-211">**[次へ]** を選択してタイルに名前を入力し、 **[適用]** を選択してタイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="c5f74-212">必要に応じて、フィールドとカードを追加できます。</span><span class="sxs-lookup"><span data-stu-id="c5f74-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="c5f74-213">ソリューションをテストする</span><span class="sxs-lookup"><span data-stu-id="c5f74-213">Test Your Solution</span></span>

<span data-ttu-id="c5f74-214">さまざまな人がカメラの前を歩いたときに、Power BI で作成したカードのデータがどのように変化するかを観察します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="c5f74-215">推論は、記録されてから表示されるまで最大 20 秒かかることがあります。</span><span class="sxs-lookup"><span data-stu-id="c5f74-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="c5f74-216">ソリューションを削除する</span><span class="sxs-lookup"><span data-stu-id="c5f74-216">Remove Your Solution</span></span>

<span data-ttu-id="c5f74-217">ソリューションを削除する場合は、Porter を使用して、デプロイ用に作成したのと同じパラメーター ファイルを使用して次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="c5f74-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="c5f74-218">次のステップ</span><span class="sxs-lookup"><span data-stu-id="c5f74-218">Next steps</span></span>

- <span data-ttu-id="c5f74-219">[ハイブリッド アプリの設計上の考慮事項](overview-app-design-considerations.md)の詳細を確認する</span><span class="sxs-lookup"><span data-stu-id="c5f74-219">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="c5f74-220">[GitHub でこのサンプルのコード](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis)を確認し、改善点を提案する</span><span class="sxs-lookup"><span data-stu-id="c5f74-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
