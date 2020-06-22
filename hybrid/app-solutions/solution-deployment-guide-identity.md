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
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Azure および Azure Stack Hub アプリのハイブリッド クラウド ID を構成する

Azure および Azure Stack Hub アプリのハイブリッド クラウド ID を構成する方法について説明します。

グローバル Azure と Azure Stack Hub の両方でアプリへのアクセスを許可するには、2 つのオプションがあります。

 * Azure Stack Hub がインターネットに継続的に接続している場合、Azure Active Directory (Azure AD) を使用できます。
 * Azure Stack Hub がインターネットから切断されているときは、Active Directory フェデレーション サービス (AD FS) を使用できます。

Azure Stack Hub 内の Azure Resource Manager を使用したデプロイまたは構成のために、サービス プリンシパルを使用し、Azure Stack Hub アプリへのアクセスを許可します。

このソリューションでは、以下を実現するためのサンプル環境を構築します。

> [!div class="checklist"]
> - グローバル Azure および Azure Stack Hub でハイブリッド ID を確立する
> - Azure Stack Hub API にアクセスするためのトークンを取得します。

このソリューションの手順を行うには、Azure Stack Hub オペレーターのアクセス許可が必要です。

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub は Azure の拡張機能です。 Azure Stack Hub はオンプレミス環境にクラウド コンピューティングの機敏性とイノベーションをもたらし、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドを可能にします。  
> 
> [ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。 これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>ポータルで Azure AD のサービス プリンシパルを作成する

Azure AD を ID ストアとして使用して Azure Stack Hub をデプロイした場合は、Azure での手順と同様の方法でサービス プリンシパルを作成できます。 [アプリ ID を使用したリソースへのアクセスの提供](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity)に関するページには、ポータルから手順を実行する方法が紹介されています。 始める前に、[Azure AD で必要なアクセス許可](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions)があることを確認してください。

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>PowerShell を使用して AD FS のサービス プリンシパルを作成する

AD FS を使用して Azure Stack Hub をデプロイした場合は、PowerShell を使ってサービス プリンシパルを作成し、アクセスのロールを割り当てて、その ID を使用して PowerShell からサインインできます。 「[アプリ ID を使用してリソースにアクセスする](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity)」には、PowerShell で必要な手順を実行する方法が紹介されています。

## <a name="using-the-azure-stack-hub-api"></a>Azure Stack Hub API を使用する

Azure Stack Hub API にアクセスするためのトークンを取得するプロセスについては、[Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md) に関するソリューションで説明されています。

## <a name="connect-to-azure-stack-hub-using-powershell"></a>PowerShell を使用して Azure Stack Hub に接続する

[Azure Stack Hub での PowerShell の設定と実行](/azure-stack/operator/azure-stack-powershell-install.md)に関するクイック スタートでは、Azure PowerShell をインストールして、Azure Stack Hub のインストールに接続するために必要な手順が説明されています。

### <a name="prerequisites"></a>前提条件

アクセスできるサブスクリプションの Azure AD に接続された Azure Stack Hub インストールが必要です。 Azure Stack Hub のインストールがない場合は、この手順に従って [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md) を設定できます。

#### <a name="connect-to-azure-stack-hub-using-code"></a>コードを使用して Azure Stack Hub に接続する

コードを使用して Azure Stack Hub に接続するには、Azure Resource Manager エンドポイント API を使用して、Azure Stack Hub インストールの認証およびグラフ エンドポイントを取得します。 次に、REST 要求を使用して認証します。 サンプル クライアント アプリケーションは [GitHub](https://github.com/shriramnat/HybridARMApplication) で見つかります。

>[!Note]
>選択した言語の Azure SDK で Azure API プロファイルがサポートされていない限り、SDK は Azure Stack Hub で動作しません。 Azure API プロファイルについて詳しくは、「[Azure Stack での API バージョンのプロファイルの管理](/azure-stack/user/azure-stack-version-profiles.md)」をご覧ください。

## <a name="next-steps"></a>次のステップ

- Azure Stack Hub での ID の処理方法の詳細については、「[Azure Stack Hub の ID アーキテクチャ](/azure-stack/operator/azure-stack-identity-architecture.md)」を参照してください。
- Azure のクラウド パターンの詳細については、「[Cloud Design Pattern (クラウド設計パターン)](https://docs.microsoft.com/azure/architecture/patterns)」を参照してください。
