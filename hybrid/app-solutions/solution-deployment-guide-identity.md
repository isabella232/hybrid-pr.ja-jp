---
title: Azure および Azure Stack Hub アプリのハイブリッド クラウド ID を構成する
description: Azure および Azure Stack Hub アプリのハイブリッド クラウド ID を構成する方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 1b3c683dd3e4a68413f83fd3cc129d6e6f594e1b
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911315"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="f7a66-103">Azure および Azure Stack Hub アプリのハイブリッド クラウド ID を構成する</span><span class="sxs-lookup"><span data-stu-id="f7a66-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="f7a66-104">Azure および Azure Stack Hub アプリのハイブリッド クラウド ID を構成する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="f7a66-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="f7a66-105">グローバル Azure と Azure Stack Hub の両方でアプリへのアクセスを許可するには、2 つのオプションがあります。</span><span class="sxs-lookup"><span data-stu-id="f7a66-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="f7a66-106">Azure Stack Hub がインターネットに継続的に接続している場合、Azure Active Directory (Azure AD) を使用できます。</span><span class="sxs-lookup"><span data-stu-id="f7a66-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="f7a66-107">Azure Stack Hub がインターネットから切断されているときは、Active Directory フェデレーション サービス (AD FS) を使用できます。</span><span class="sxs-lookup"><span data-stu-id="f7a66-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="f7a66-108">Azure Stack Hub 内の Azure Resource Manager を使用したデプロイまたは構成のために、サービス プリンシパルを使用し、Azure Stack Hub アプリへのアクセスを許可します。</span><span class="sxs-lookup"><span data-stu-id="f7a66-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="f7a66-109">このソリューションでは、以下を実現するためのサンプル環境を構築します。</span><span class="sxs-lookup"><span data-stu-id="f7a66-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="f7a66-110">グローバル Azure および Azure Stack Hub でハイブリッド ID を確立する</span><span class="sxs-lookup"><span data-stu-id="f7a66-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="f7a66-111">Azure Stack Hub API にアクセスするためのトークンを取得します。</span><span class="sxs-lookup"><span data-stu-id="f7a66-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="f7a66-112">このソリューションの手順を行うには、Azure Stack Hub オペレーターのアクセス許可が必要です。</span><span class="sxs-lookup"><span data-stu-id="f7a66-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="f7a66-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="f7a66-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="f7a66-114">Microsoft Azure Stack Hub は Azure の拡張機能です。</span><span class="sxs-lookup"><span data-stu-id="f7a66-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="f7a66-115">Azure Stack Hub はオンプレミス環境にクラウド コンピューティングの機敏性とイノベーションをもたらし、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドを可能にします。</span><span class="sxs-lookup"><span data-stu-id="f7a66-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="f7a66-116">[ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。</span><span class="sxs-lookup"><span data-stu-id="f7a66-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="f7a66-117">これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="f7a66-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="f7a66-118">ポータルで Azure AD のサービス プリンシパルを作成する</span><span class="sxs-lookup"><span data-stu-id="f7a66-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="f7a66-119">Azure AD を ID ストアとして使用して Azure Stack Hub をデプロイした場合は、Azure での手順と同様の方法でサービス プリンシパルを作成できます。</span><span class="sxs-lookup"><span data-stu-id="f7a66-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="f7a66-120">[アプリ ID を使用したリソースへのアクセスの提供](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity)に関するページには、ポータルから手順を実行する方法が紹介されています。</span><span class="sxs-lookup"><span data-stu-id="f7a66-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="f7a66-121">始める前に、[Azure AD で必要なアクセス許可](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions)があることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="f7a66-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="f7a66-122">PowerShell を使用して AD FS のサービス プリンシパルを作成する</span><span class="sxs-lookup"><span data-stu-id="f7a66-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="f7a66-123">AD FS を使用して Azure Stack Hub をデプロイした場合は、PowerShell を使ってサービス プリンシパルを作成し、アクセスのロールを割り当てて、その ID を使用して PowerShell からサインインできます。</span><span class="sxs-lookup"><span data-stu-id="f7a66-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="f7a66-124">「[アプリ ID を使用してリソースにアクセスする](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity)」には、PowerShell で必要な手順を実行する方法が紹介されています。</span><span class="sxs-lookup"><span data-stu-id="f7a66-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="f7a66-125">Azure Stack Hub API を使用する</span><span class="sxs-lookup"><span data-stu-id="f7a66-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="f7a66-126">Azure Stack Hub API にアクセスするためのトークンを取得するプロセスについては、[Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md) に関するソリューションで説明されています。</span><span class="sxs-lookup"><span data-stu-id="f7a66-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="f7a66-127">PowerShell を使用して Azure Stack Hub に接続する</span><span class="sxs-lookup"><span data-stu-id="f7a66-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="f7a66-128">[Azure Stack Hub での PowerShell の設定と実行](/azure-stack/operator/azure-stack-powershell-install.md)に関するクイック スタートでは、Azure PowerShell をインストールして、Azure Stack Hub のインストールに接続するために必要な手順が説明されています。</span><span class="sxs-lookup"><span data-stu-id="f7a66-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f7a66-129">前提条件</span><span class="sxs-lookup"><span data-stu-id="f7a66-129">Prerequisites</span></span>

<span data-ttu-id="f7a66-130">アクセスできるサブスクリプションの Azure AD に接続された Azure Stack Hub インストールが必要です。</span><span class="sxs-lookup"><span data-stu-id="f7a66-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="f7a66-131">Azure Stack Hub のインストールがない場合は、この手順に従って [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md) を設定できます。</span><span class="sxs-lookup"><span data-stu-id="f7a66-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="f7a66-132">コードを使用して Azure Stack Hub に接続する</span><span class="sxs-lookup"><span data-stu-id="f7a66-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="f7a66-133">コードを使用して Azure Stack Hub に接続するには、Azure Resource Manager エンドポイント API を使用して、Azure Stack Hub インストールの認証およびグラフ エンドポイントを取得します。</span><span class="sxs-lookup"><span data-stu-id="f7a66-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="f7a66-134">次に、REST 要求を使用して認証します。</span><span class="sxs-lookup"><span data-stu-id="f7a66-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="f7a66-135">サンプル クライアント アプリケーションは [GitHub](https://github.com/shriramnat/HybridARMApplication) で見つかります。</span><span class="sxs-lookup"><span data-stu-id="f7a66-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="f7a66-136">選択した言語の Azure SDK で Azure API プロファイルがサポートされていない限り、SDK は Azure Stack Hub で動作しません。</span><span class="sxs-lookup"><span data-stu-id="f7a66-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="f7a66-137">Azure API プロファイルについて詳しくは、「[Azure Stack での API バージョンのプロファイルの管理](/azure-stack/user/azure-stack-version-profiles.md)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="f7a66-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles.md) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="f7a66-138">次のステップ</span><span class="sxs-lookup"><span data-stu-id="f7a66-138">Next steps</span></span>

- <span data-ttu-id="f7a66-139">Azure Stack Hub での ID の処理方法の詳細については、「[Azure Stack Hub の ID アーキテクチャ](/azure-stack/operator/azure-stack-identity-architecture.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="f7a66-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span></span>
- <span data-ttu-id="f7a66-140">Azure のクラウド パターンの詳細については、「[Cloud Design Pattern (クラウド設計パターン)](https://docs.microsoft.com/azure/architecture/patterns)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="f7a66-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
