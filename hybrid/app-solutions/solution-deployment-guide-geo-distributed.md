---
title: Azure と Azure Stack Hub を利用し、地理的分散アプリでトラフィックを転送する
description: Azure と Azure Stack Hub を使用する地理的に分散されたアプリ ソリューションで、トラフィックを特定のエンドポイントに転送する方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 741ddf2c3ed234788af359dd233f6a656fbea13c
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477356"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a>Azure と Azure Stack Hub を利用し、地理的分散アプリでトラフィックを転送する

地理的分散アプリ パターンを使用して、さまざまなメトリックに基づき、特定のエンドポイントにトラフィックを送信する方法について説明します。 地理的ベースのルーティングとエンドポイント構成で Traffic Manager プロファイルを作成すると、リージョンの要件、企業および国際的な規制、およびデータ ニーズに基づいて、情報がエンドポイントにルーティングされます。

このソリューションでは、以下を実現するためのサンプル環境を構築します。

> [!div class="checklist"]
> - 地理的分散アプリを作成する
> - Traffic Manager を使用して、アプリを対象にします。

## <a name="use-the-geo-distributed-apps-pattern"></a>地理的分散アプリ パターンを使用する

地理的分散パターンを使用すると、アプリは複数のリージョンにまたがります。 パブリック クラウドを既定として設定できますが、一部のユーザーは、場合によっては、自分のリージョンにデータを残す必要がある場合があります。 ユーザーをその要件に基づいて最も適したクラウドに送信できます。

### <a name="issues-and-considerations"></a>問題と注意事項

#### <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

この記事で作成するソリューションは、スケーラビリティに対応しません。 ただし、他の Azure とオンプレミスのソリューションと組み合わせて使用すれば、スケーラビリティ要件に対応できます。 Traffic Manager を介した自動スケーリングを伴うハイブリッド ソリューションの作成に関する詳細については、「[Azure でクラウド間スケーリング ソリューションを作成する](solution-deployment-guide-cross-cloud-scaling.md)」を参照してください。

#### <a name="availability-considerations"></a>可用性に関する考慮事項

スケーラビリティに関する考慮事項と同様に、このソリューションでは可用性を直接扱いません。 ただし、Azure とオンプレミスのソリューションをこのソリューション内に実装して、関連するすべてのコンポーネントの高可用性を確保できます。

### <a name="when-to-use-this-pattern"></a>このパターンを使用する状況

- 組織には、リージョンのセキュリティおよび分散に関するカスタム ポリシーが必要な支店が世界中にあります。

- 組織の支店それぞれは、従業員、ビジネス、施設のデータをプルしますが、これには地域の規制やタイム ゾーンに従ったレポート アクティビティが必要になります。

- 高スケール要件を満たすには、極端に負荷の大きい要件に対応するように、単一のリージョン内、または複数のリージョンにわたって、複数のアプリ デプロイを使用してアプリを水平方向に拡張します。

### <a name="planning-the-topology"></a>トポロジの計画

分散アプリのフットプリントを構築する前に、次を知っておくと役立ちます。

- **アプリのカスタム ドメイン:** 顧客がアプリへのアクセスに使用するカスタム ドメイン名が必要です。 サンプル アプリでは、カスタム ドメイン名は *www\.scalableasedemo.com* です。

- **Traffic Manager ドメイン:** [Azure Traffic Manager プロファイル](/azure/traffic-manager/traffic-manager-manage-profiles)の作成時に、ドメイン名が選択されます。 この名前は、Traffic Manager が管理するドメイン エントリを登録する際に、*trafficmanager.net* サフィックスと組み合わされます。 サンプル アプリでは、選択される名前は *scalable-ase-demo*です。 そのため、Traffic Manager で管理される完全なドメイン名は、*scalable-ase-demo.trafficmanager.net* になります。

- **アプリ フットプリントのスケーリングに関する戦略:** アプリのフットプリントは単一リージョン内の複数の App Service 環境に分散されるのか、リージョンは複数なのか、あるいは両方の手法の混在になるのかを決定します。 この決定は、顧客のトラフィックが発生する場所に加えて、アプリをサポートするバックエンド インフラストラクチャの他の要素のスケーラビリティに関する期待事項に基づく必要があります。 たとえば、完全にステートレスなアプリでは、各 Azure リージョンで複数の App Service 環境を組み合わせ、さらに複数の Azure リージョンにデプロイされた App Service 環境を掛け合わせることで、大規模なスケーリングを実施できます。 選択できるグローバルな Azure リージョンは 15 以上あるため、顧客はスケーラビリティのきわめて高いアプリ フットプリントを世界規模で構築できます。 ここで使用されるサンプル アプリでは、単一の Azure リージョン (米国中南部) に 3 つの App Service 環境が作成されています。

- **App Service 環境の名前付け規則:** 各 App Service 環境には一意の名前が必要です。 App Service 環境が 1 つか 2 つ以上になるとき、各 App Service 環境を識別する目的で名前付け規則を用意すると便利です。 ここで使用されるサンプル アプリには、簡単な名前付け規則が使用されました。 3 つの App Service 環境の名前は *fe1ase*、*fe2ase*、*fe3ase* です。

- **アプリの名前付け規則**:アプリのインスタンスが複数デプロイされるため、デプロイされたアプリの各インスタンスに名前が必要です。 Power Apps 用の App Service Environment では、同じアプリ名を複数の環境で使用できます。 App Service 環境ごとに一意のドメイン サフィックスがあるため、開発者は各環境でまったく同じアプリ名を再利用できます。 たとえば、開発者はアプリに *myapp.foo1.p.azurewebsites.net*、*myapp.foo2.p.azurewebsites.net*、*myapp.foo3.p.azurewebsites.net* のように名前を付けることができます。 ここで使用されるアプリについては、アプリ インスタンスごとに一意の名前が付けられています。 使用されているアプリ インスタンス名は *webfrontend1*、*webfrontend2*、*webfrontend3* です。

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub は Azure の拡張機能です。 Azure Stack Hub により、オンプレミス環境にクラウド コンピューティングの機敏性とイノベーションがもたらされ、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドが可能になります。  
> 
> [ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。 これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。

## <a name="part-1-create-a-geo-distributed-app"></a>パート 1: 地理的分散アプリを作成する

このパートでは、Web アプリを作成します。

> [!div class="checklist"]
> - Web アプリを作成し、公開します。
> - Azure Repos にコードを追加する
> - アプリ ビルドを複数のクラウド ターゲットにポイントします。
> - CD プロセスを管理し、構成します。

### <a name="prerequisites"></a>前提条件

Azure サブスクリプションと Azure Stack Hub のインストールが必要です。

### <a name="geo-distributed-app-steps"></a>地理的に分散されたアプリの手順

### <a name="obtain-a-custom-domain-and-configure-dns"></a>カスタム ドメインを取得し DNS を構成する

ドメインの DNS ゾーン ファイルを更新します。 Azure AD は続いて、カスタム ドメイン名の所有権を確認できます。 Azure 内の Azure/Office 365/外部 DNS レコードに [Azure DNS](/azure/dns/dns-getstarted-portal) を使用するか、または[別の DNS レジストラー](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)で DNS エントリを追加します。

1. パブリック レジストラーでカスタム ドメインを登録します。

2. ドメインのドメイン名レジストラーにサインインします。 DNS の更新を行うには、承認された管理者が必要になることがあります。

3. Azure AD から提供された DNS エントリを追加して、ドメインの DNS ゾーン ファイルを更新します。 メール ルーティングや Web ホスティングなどの動作は変更されません。

### <a name="create-web-apps-and-publish"></a>Web アプリを作成し発行する

ハイブリッドの継続的インテグレーション/継続的デリバリー (CI/CD) を設定し、Web アプリを Azure と Azure Stack Hub にデプロイし、両方のクラウドに変更を自動プッシュします。

> [!Note]  
> (Windows Server と SQL の) 実行および App Service のデプロイには、適切なイメージがシンジケート化された Azure Stack Hub が必要です。 詳細については、「[App Service on Azure Stack Hub のデプロイの前提条件](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)」を参照してください。

#### <a name="add-code-to-azure-repos"></a>Azure Repos にコードを追加する

1. Azure Repos 上で**プロジェクト作成権限が付与されているアカウント**を使用して、Visual Studio にサインインします。

    CI/CD は、アプリ コードとインフラストラクチャ コードの両方に適用できます。 プライベート クラウド開発とホステッド クラウド開発の両方に、[Azure Resource Manager テンプレート](https://azure.microsoft.com/resources/templates/)を使用します。

    ![Visual Studio でプロジェクトに接続する](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. 既定の Web アプリを作成して開くことで、**リポジトリを複製**します。

    ![Visual Studio でリポジトリを複製する](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>両方のクラウドで Web アプリ デプロイを作成する

1. **WebApplication.csproj** ファイルを編集します。`Runtimeidentifier` を選択し、`win10-x64` を追加します。 (「[自己完結型デプロイ](/dotnet/core/deploying/deploy-with-vs#simpleSelf)」に関するドキュメントを参照してください)。

    ![Visual Studio で Web アプリ プロジェクト ファイルを編集する](media/solution-deployment-guide-geo-distributed/image3.png)

2. チーム エクスプローラーを使用して、**コードを Azure Repos にチェックインします**。

3. **アプリケーション コード**が Azure Repos にチェックインされたことを確認します。

### <a name="create-the-build-definition"></a>ビルド定義を作成する

1. **Azure Pipelines にサインイン**し、ビルド定義を作成する機能を確認します。

2. `-r win10-x64` コードを追加します。 この追加は、.NET Core を使用して自己完結型のデプロイをトリガーするために必要です。

    ![Azure Pipelines でビルド定義にコードを追加する](media/solution-deployment-guide-geo-distributed/image4.png)

3. **ビルドを実行します**。 [自己完結型のデプロイ ビルド](/dotnet/core/deploying/deploy-with-vs#simpleSelf)のプロセスにより、Azure および Azure Stack Hub 上で実行できる成果物が発行されます。

#### <a name="using-an-azure-hosted-agent"></a>Azure ホステッド エージェントを使用する

Web アプリをビルドおよびデプロイする場合、Azure Pipelines でホステッド エージェントを使用すると便利です。 Microsoft Azure によってメンテナンスやアップグレードが自動的に実施され、開発、テスト、デプロイには中断がありません。

### <a name="manage-and-configure-the-cd-process"></a>CD プロセスを管理および構成する

Azure DevOps Services が提供するパイプラインは自由に構成でき、管理性にも優れ、開発、ステージング、QA、運用など、さまざまな環境へのリリースに使用できます。また、特定のステージで承認を要求できます。

## <a name="create-release-definition"></a>リリース定義の作成

1. Azure DevOps Services の **[ビルドとリリース]** セクションの **[リリース]** タブで **[+]** ボタンを選択して、新しいリリースを追加します。

    ![Azure DevOps Services でリリース定義を作成する](media/solution-deployment-guide-geo-distributed/image5.png)

2. Azure App Service Deployment テンプレートを適用します。

   ![Azure DevOps Services で [Azure App Service の配置] テンプレートを適用する](media/solution-deployment-guide-geo-distributed/image6.png)

3. **[成果物の追加]** で、Azure Cloud ビルド アプリに対して成果物を追加します。

   ![Azure DevOps Services で Azure Cloud ビルドに成果物を追加する](media/solution-deployment-guide-geo-distributed/image7.png)

4. [パイプライン] タブで、環境の**フェーズ、タスク** リンクを選択し、Azure のクラウド環境の値を設定します。

   ![Azure DevOps Services で Azure クラウド環境の値を設定する](media/solution-deployment-guide-geo-distributed/image8.png)

5. **環境名**を設定し、Azure Cloud エンドポイントに対して **Azure サブスクリプション**を選択します。

      ![Azure DevOps Services で Azure Cloud エンドポイントの Azure サブスクリプションを選択する](media/solution-deployment-guide-geo-distributed/image9.png)

6. **[App Service の名前]** で、必須の Azure アプリ サービス名を設定します。

      ![Azure DevOps Services で Azure アプリ サービス名を設定する](media/solution-deployment-guide-geo-distributed/image10.png)

7. Azure クラウドでホストされる環境の **[エージェント キュー]** に「Hosted VS2017」と入力します。

      ![Azure DevOps Services で Azure クラウド ホスト環境のエージェント キューを設定する](media/solution-deployment-guide-geo-distributed/image11.png)

8. [Azure App Service 配置] メニューで、環境に対して有効な**パッケージまたはフォルダー**を選択します。 **[OK]** を選択して、**フォルダーの場所**を選択します。
  
      ![Azure DevOps Services で Azure App Service 環境のパッケージまたはフォルダーを選択する](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Azure DevOps Services で Azure App Service 環境のパッケージまたはフォルダーを選択する](media/solution-deployment-guide-geo-distributed/image13.png)

9. すべての変更を保存し、**リリース パイプライン**に戻ります。

    ![Azure DevOps Services でリリース パイプラインの変更を保存する](media/solution-deployment-guide-geo-distributed/image14.png)

10. Azure Stack Hub アプリのビルドを選択して、新しい成果物を追加します。

    ![Azure DevOps Services で Azure Stack Hub アプリの新しい成果物を追加する](media/solution-deployment-guide-geo-distributed/image15.png)


11. [Azure App Service の配置] を適用し、環境をもう 1 つ追加します。

    ![Azure DevOps Services で [Azure App Service の配置] に環境を追加する](media/solution-deployment-guide-geo-distributed/image16.png)

12. 新しい環境に Azure Stack Hub という名前を付けます。

    ![Azure DevOps Services の [Azure App Service の配置] の環境に名前を付ける](media/solution-deployment-guide-geo-distributed/image17.png)

13. **[タスク]** タブで Azure Stack Hub 環境を見つけます。

    ![Azure DevOps Services の Azure DevOps Services の Azure Stack Hub 環境](media/solution-deployment-guide-geo-distributed/image18.png)

14. Azure Stack Hub エンドポイントのサブスクリプションを選択します。

    ![Azure DevOps Services で Azure Stack Hub エンドポイントのサブスクリプションを選択する](media/solution-deployment-guide-geo-distributed/image19.png)

15. [App Service の名前] として Azure Stack Hub Web アプリの名前を設定します。

    ![Azure DevOps Services で Azure Stack Hub Web アプリ名を設定する](media/solution-deployment-guide-geo-distributed/image20.png)

16. Azure Stack Hub エージェントを選択します。

    ![Azure DevOps Services で Azure Stack Hub エージェントを選択する](media/solution-deployment-guide-geo-distributed/image21.png)

17. [Azure App Service 配置] セクションで、環境に対して有効な**パッケージまたはフォルダー**を選択します。 **[OK]** を選択して、フォルダーの場所を選択します。

    ![Azure DevOps Services で [Azure App Service の配置] のフォルダーを選択する](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Azure DevOps Services で [Azure App Service の配置] のフォルダーを選択する](media/solution-deployment-guide-geo-distributed/image23.png)

18. [変数] タブで、`VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` という名前の変数を追加し、その値を **true** に設定し、スコープを Azure Stack Hub に設定します。

    ![Azure DevOps Services で [Azure App の配置] に変数を追加する](media/solution-deployment-guide-geo-distributed/image24.png)

19. 両方の成果物で**継続的**配置トリガー アイコンを選択し、**継続的**配置トリガーを有効にします。

    ![Azure DevOps Services で継続的配置トリガーを選択する](media/solution-deployment-guide-geo-distributed/image25.png)

20. Azure Stack Hub 環境で**配置前**条件アイコンを選択し、トリガーを**リリース後**に設定します。

    ![Azure DevOps Services で配置前の条件を選択する](media/solution-deployment-guide-geo-distributed/image26.png)

21. すべての変更を保存します。

> [!Note]  
> タスクの一部の設定は、テンプレートからリリース定義を作成したときに、[環境変数](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables)として自動的に定義されている可能性があります。 こうした設定は、タスクの設定では変更できません。これらの設定を編集するには、親環境項目を選択する必要があります。

## <a name="part-2-update-web-app-options"></a>パート 2: Web アプリ オプションを更新する

[Azure App Service](/azure/app-service/overview) は、非常にスケーラブルな、自己適用型の Web ホスティング サービスを提供します。

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - 既存のカスタム DNS 名を Azure Web Apps にマップします。
> - **CNAME レコード**と **A レコード**を使用し、カスタム DNS 名を App Service にマップします。

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>既存のカスタム DNS 名を Azure Web Apps にマップする

> [!Note]  
> ルート ドメイン (northwind.com など) を除くすべてのカスタム DNS 名に CNAME を使用します。

ライブ サイトとその DNS ドメイン名を App Service に移行する方法については、「[Azure App Service へのアクティブな DNS 名の移行](/azure/app-service/manage-custom-dns-migrate-domain)」をご覧ください。

### <a name="prerequisites"></a>前提条件

このソリューションを完了するには:

- [App Service アプリを作成する](/azure/app-service/)か、別のソリューションで作成したアプリを使用します。

- ドメイン名を購入し、ドメイン プロバイダーの DNS レジストリへのアクセスを確認します。

ドメインの DNS ゾーン ファイルを更新します。 Azure AD は、カスタム ドメイン名の所有権を確認します。 Azure 内の Azure/Office 365/外部 DNS レコードに [Azure DNS](/azure/dns/dns-getstarted-portal) を使用するか、または[別の DNS レジストラー](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)で DNS エントリを追加します。

- パブリック レジストラーでカスタム ドメインを登録します。

- ドメインのドメイン名レジストラーにサインインします。 (DNS の更新を行うには、承認された管理者が必要になることがあります)。

- Azure AD から提供された DNS エントリを追加して、ドメインの DNS ゾーン ファイルを更新します。

たとえば、northwindcloud.com と www\.northwindcloud.com の DNS エントリを追加するには、northwindcloud.com ルート ドメインの DNS 設定を構成します。

> [!Note]  
> ドメイン名は [Microsoft Azure portal](/azure/app-service/manage-custom-dns-buy-domain) を使用して購入できます。 Web アプリにカスタム DNS 名をマップするには、Web アプリの [App Service プラン](https://azure.microsoft.com/pricing/details/app-service/)が有料レベル (**Shared**、**Basic**、**Standard**、または **Premium**) である必要があります。

### <a name="create-and-map-cname-and-a-records"></a>CNAME および A レコードを作成してマップする

#### <a name="access-dns-records-with-domain-provider"></a>ドメイン プロバイダーで DNS レコードにアクセスする

> [!Note]  
>  Azure DNS を使用して、Azure Web Apps のカスタム DNS 名を構成します。 詳細については、「[Azure DNS を使用して Azure サービス用のカスタム ドメイン設定を提供する](/azure/dns/dns-custom-domain)」をご覧ください。

1. ドメイン プロバイダーの Web サイトにサインインします。

2. DNS レコードの管理ページを探します。 各ドメイン プロバイダーは、独自の DNS レコード インターフェイスを保有します。 **[ドメイン名]** 、 **[DNS]** 、 **[ネーム サーバー管理]** というラベルが付いたサイトの領域を探します。

DNS レコード ページは、 **[My domains] (マイ ドメイン)** で表示できます。 **[ゾーン ファイル]** 、 **[DNS レコード]** 、または **[詳細構成]** という名前のリンクを見つけます。

以下のスクリーンショットは、DNS レコード ページの例です。

![DNS レコード ページの例](media/solution-deployment-guide-geo-distributed/image28.png)

1. ドメイン名レジストラーで、 **[Add or Create] (追加または作成)** を選択してレコードを作成します。 プロバイダーによっては、追加するレコード タイプごとに異なるリンクが用意されています。 プロバイダーのドキュメントを参照してください。

2. CNAME レコードを追加して、サブドメインをアプリの既定のホスト名にマップします。

   www\.northwindcloud.com ドメインの例では、名前を `<app_name>.azurewebsites.net` にマップする CNAME レコードを追加します。

CNAME を追加した後の DNS レコード ページは次の例のようになります。

![Azure アプリへのポータル ナビゲーション](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>Azure で CNAME レコード マッピングを有効にする

1. 新しいタブで Azure portal にサインインします。

2. [App Services] に移動します。

3. Web アプリを選択します。

4. Azure Portal のアプリ ページの左側のナビゲーションで、 **[カスタム ドメイン]** を選択します。

5. **[ホスト名の追加]** の横の **+** アイコンを選択します。

6. `www.northwindcloud.com` のように完全修飾ドメイン名を入力します。

7. **[検証]** を選択します。

8. 指示された場合は、他の種類 (`A` または `TXT`) の追加レコードをドメイン名レジストラー DNS レコードに追加します。 Azure は、これらのレコードの値と種類を提供します。

   a.  アプリの IP アドレスにマップするための **A** レコード。

   b.  アプリの既定のホスト名 `<app_name>.azurewebsites.net` にマップするための **TXT** レコード。 App Service は、このレコードを、カスタム ドメインの所有者を検証するために構成時にのみ使用します。 検証後、TXT レコードを削除してください。

9. ドメイン レジスター タブでこのタスクを完了し、 **[ホスト名の追加]** ボタンがアクティブになるまで、再検証します。

10. **[ホスト名レコード タイプ]** が **[CNAME]** (www.example.com または任意のサブドメイン) に設定されていることを確認します。

11. **[ホスト名の追加]** を選択します。

12. `northwindcloud.com` のように完全修飾ドメイン名を入力します。

13. **[検証]** を選択します。 **[追加]** がアクティブになります。

14. **[ホスト名レコード タイプ]** が **[A レコード]** (example.com) に設定されていることを確認します。

15. **ホスト名を追加します**。

    アプリの **[カスタム ドメイン]** ページに新しいホスト名が反映されるまで時間がかかることがあります。 データを更新するために、ブラウザーの表示を更新してみてください。
  
    ![カスタム ドメイン](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    エラーが発生した場合は、検証エラーの通知がページの下部に表示されます。 ![ドメインの検証エラー](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  上記の手順を繰り返して、ワイルドカード ドメイン (\*northwindcloud.com) をマップできます。 これにより、それぞれに個別の CNAME レコードを作成せずに、このアプリ サービスに別のサブドメインを追加できます。 レジストラーの指示に従って、この設定を構成します。

#### <a name="test-in-a-browser"></a>ブラウザーでテストする

先ほど構成された DNS 名 (`northwindcloud.com`、`www.northwindcloud.com` など) を参照します。

## <a name="part-3-bind-a-custom-ssl-cert"></a>パート 3: カスタム SSL 証明書をバインドする

このパートでの作業:

> [!div class="checklist"]
> - カスタム SSL 証明書を App Service にバインドします。
> - アプリに HTTPS を適用します。
> - スクリプトで SSL 証明書のバインドを自動化します。

> [!Note]  
> 必要に応じて、Microsoft Azure portal で顧客の SSL 証明書を取得し、それを Web アプリにバインドします。 詳細については、[Azure App Service 証明書のチュートリアル](/azure/app-service/web-sites-purchase-ssl-web-site)を参照してください。

### <a name="prerequisites"></a>前提条件

このソリューションを完了するには:

- [App Service アプリを作成します。](/azure/app-service/)
- [カスタム DNS 名を Web アプリにマップします。](/azure/app-service/app-service-web-tutorial-custom-domain)
- 信頼された証明機関から SSL 証明書を取得し、キーを使用して要求に署名します。

### <a name="requirements-for-your-ssl-certificate"></a>SSL 証明書の必要条件

App Service で証明書を使用するには、証明書が次のすべての要件を満たしている必要があります。

- 信頼された証明機関によって署名されています。

- パスワードで保護された PFX ファイルとしてエクスポートされています。

- 少なくとも 2048 ビット長の秘密キーが含まれています。

- 証明書チェーン内のすべての中間証明書が含まれています。

> [!Note]  
> **楕円曲線暗号 (ECC) 証明書**は、App Service で有効ですが、このガイドでは取り上げていません。 ECC 証明書の作成でサポートが必要なときは、証明機関に問い合わせてください。

#### <a name="prepare-the-web-app"></a>Web アプリを用意する

カスタム SSL 証明書を Web アプリにバインドするには、[App Service プラン](https://azure.microsoft.com/pricing/details/app-service/)が **Basic**、**Standard** または **Premium** のいずれかのレベルである必要があります。

#### <a name="sign-in-to-azure"></a>Azure へのサインイン

1. [Azure portal](https://portal.azure.com/) を開き、Web アプリに移動します。

2. 左側のメニューで、 **[App Services]** を選択し、Web アプリ名を選択します。

![Azure portal で Web アプリを選択する](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a>価格レベルの確認

1. Web アプリ ページの左側のナビゲーションで **[設定]** セクションまでスクロールし、 **[スケール アップ (App Service プラン)]** を選択します。

    ![Web アプリのスケールアップ メニュー](media/solution-deployment-guide-geo-distributed/image34.png)

1. Web アプリが **Free** レベルまたは **Shared** レベルに含まれていないことを確認します。 Web アプリの現在の層が濃青色のボックスに強調表示されます。

    ![Web アプリで価格レベルを確認する](media/solution-deployment-guide-geo-distributed/image35.png)

カスタム SSL は、**Free** レベルまたは **Shared** レベルではサポートされていません。 アップスケールするには、次のセクション、 **[価格レベルの選択]** ページの手順に従い、[[Upload and bind your SSL certificate]\(SSL 証明書のアップロードおよびバインド\)](/azure/app-service/app-service-web-tutorial-custom-ssl) にスキップします。

#### <a name="scale-up-your-app-service-plan"></a>App Service プランのスケール アップ

1. **Basic**、**Standard**、**Premium** のいずれかのレベルを選択します。

2. **[選択]** を選択します。

![Web アプリの価格レベルを選択する](media/solution-deployment-guide-geo-distributed/image36.png)

通知が表示されたら、スケール操作は完了です。

![スケール アップの通知](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>SSL 証明書をバインドし、中間証明書を結合する

チェーン内の複数の証明書を結合します。

1. テキストエディタで、受信した**それぞれの証明書を開きます**。

2. 結合した証明書用に *mergedcertificate.crt* という名前のファイルを作成します。 テキスト エディターで、このファイルに各証明書の内容をコピーします。 証明書の順序は、証明書チェーンの順番に従う必要があります。自分の証明書から始まり、ルート証明書で終わります。 次の例のようになります。

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

#### <a name="export-certificate-to-pfx"></a>PFX への証明書のエクスポート

証明書で生成された秘密キーを使用して、結合した SSL 証明書をエクスポートします。

秘密キー ファイルは OpenSSL 経由で作成されます。 証明書を PFX にエクスポートするには、次のコマンドを実行し、プレースホルダー `<private-key-file>` と `<merged-certificate-file>` を秘密キーのパスとマージされた証明書ファイルに置き換えます。

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

プロンプトが表示されたら、後から SSL 証明書を App Service にアップロードするためのエクスポート パスワードを定義します。

IIS または **Certreq.exe** を使用して証明書の要求を生成した場合は、ローカル コンピューターに証明書をインストールした後で[証明書を PFX にエクスポート](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11))します。

#### <a name="upload-the-ssl-certificate"></a>SSL 証明書をアップロードする

1. Web アプリの左側のナビゲーションで **[SSL 設定]** を選択します。

2. **[証明書のアップロード]** を選択します。

3. **[PFX 証明書ファイル]** で、PFX ファイルを選択します。

4. **証明書のパスワード**、PFX ファイルをエクスポートするときに作成したパスワードを入力します。

5. **[アップロード]** を選択します。

    ![SSL 証明書をアップロードする](media/solution-deployment-guide-geo-distributed/image38.png)

App Service による証明書のアップロードが完了すると、 **[SSL 設定]** ページにアップロードした証明書が表示されます。

![SSL 設定](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a>SSL 証明書のバインド

1. **[SSL バインド]** セクションで **[バインドの追加]** を選択します。

    > [!Note]  
    >  証明書をアップロードしたのに **[ホスト名]** ドロップダウンにドメイン名が表示されない場合は、ブラウザのページを最新の情報に更新してみてください。

2. **[SSL バインディングの追加]** ページで、ドロップダウンから保護するドメインの名前と使用する証明書を選択します。

3. **[SSL Type] \(SSL の種類)** で、[**Server Name Indication (SNI)** ](https://en.wikipedia.org/wiki/Server_Name_Indication) ベースの SSL を使用するか IP ベースの SSL を使用するかを選択します。

    - **SNI ベースの SSL**:複数の SNI ベースの SSL バインディングを追加できます。 このオプションでは、複数の SSL 証明書を使用して、同一の IP アドレス上の複数のドメインを保護できます。 最新のブラウザーのほとんど (Inernet Explorer、Chrome、Firefox、Opera など) が SNI をサポートしています (ブラウザーのサポートに関するより包括的な情報については、「[Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)」を参照してください)。

    - **IP ベースの SSL**:IP ベースの SSL バインディングを 1 つだけ追加できます。 このオプションでは、SSL 証明書を 1 つだけ使用して、専用のパブリック IP アドレスを保護します。 複数のドメインを保護するには、同じ SSL 証明書を使用してすべてのドメインを保護します。 IP ベースの SSL は、SSL バインドの従来のオプションです。

4. **[バインドの追加]** を選択します。

    ![SSL バインドの追加](media/solution-deployment-guide-geo-distributed/image40.png)

App Service による証明書のアップロードが完了すると、 **[SSL バインド]** セクションにアップロードした証明書が表示されます。

![SSL バインドのアップロードの完了](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>IP SSL の A レコードを再マップする

Web アプリで IP ベースの SSL を使用していない場合、[カスタム ドメインの HTTPS のテスト](/azure/app-service/app-service-web-tutorial-custom-ssl)に関するセクションにスキップしてください。

既定では、Web アプリは、共有のパブリック IP アドレスを使用します。 IP ベースの SSL で証明書をバインドすると、Web アプリ用の新規の専用 IP アドレスが App Service によって作成されます。

A レコードが Web アプリにマップされた場合、ドメイン レジストリを専用の IP アドレスで更新する必要があります。

**[カスタム ドメイン]** ページが、新規の専用 IP アドレスで更新されます。 [この IP アドレスをコピー](/azure/app-service/app-service-web-tutorial-custom-domain)して、この新しい IP アドレスに [A レコードを再マップ](/azure/app-service/app-service-web-tutorial-custom-domain)します。

#### <a name="test-https"></a>HTTPS のテスト

さまざまなブラウザーで `https://<your.custom.domain>` にアクセスして、Web アプリが提供されていることを確認します。

![Web アプリの参照](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> 証明書の検証エラーが発生した場合、自己署名証明書が原因であるか、PFX ファイルにエクスポートするときに中間証明書が除外された可能性があります。

#### <a name="enforce-https"></a>HTTPS の適用

既定では、どなたでも HTTP を使用して Web アプリにアクセスできます。 HTTPS ポートへのすべての HTTP 要求をリダイレクトできます。

Web アプリページで、 **[SSL 設定]** を選択します。 その後、 **[HTTPS のみ]** で、 **[On]** を選択します。

![HTTPS の適用](media/solution-deployment-guide-geo-distributed/image43.png)

操作が完了すると、アプリを指定する HTTP URL のいずれかに移動します。 次に例を示します。

- https://<app_name>.azurewebsites.net
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a>TLS 1.1/1.2 の適用

アプリでは既定で [TLS 1.0](https://wikipedia.org/wiki/Transport_Layer_Security) が有効です。これは、業界標準 ([PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard) など) によって安全であると見なされなくなっています。 より上位の TLS バージョンを適用するには、次の手順に従います。

1. Web アプリ ページで、左側のナビゲーションにある **[SSL 設定]** を選択します。

2. **[TLS version] (TLS バージョン)** で、最低限の TLS バージョンを選択します。

    ![TLS 1.1/1.2 の適用](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a>Traffic Manager プロファイルの作成

1. **[リソースの作成]**  >  **[ネットワーク]**  >  **[Traffic Manager プロファイル]**  >  **[作成]** の順に選択します。

2. **[Traffic Manager プロファイルの作成]** で、以下を実行します。

    1. **[名前]** に、プロファイルの名前を入力します。 この名前は trafficmanager.net ゾーン内で一意である必要があります。結果的に、Traffic Manager プロファイルへのアクセスに使用される DNS 名 trafficmanager.net になるためです。

    2. **[ルーティング方法]** で、**地理的ルーティング方法**を選択します。

    3. **[サブスクリプション]** で、このプロファイルを作成するサブスクリプションを選択します。

    4. **[リソース グループ]** で、このプロファイルを配置する新しいリソース グループを作成します。

    5. **[リソース グループの場所]** で、リソース グループの場所を選択します。 これはリソース グループの場所を指定する設定であり、グローバルにデプロイされる Traffic Manager プロファイルには影響しません。

    6. **［作成］** を選択します

    7. Traffic Manager プロファイルは、グローバルなデプロイが完了すると、それぞれのリソース グループ内にリソースの 1 つとして表示されます。

        ![Traffic Manager プロファイルのリソース グループ作成](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a>Traffic Manager エンドポイントの追加

1. ポータルの検索バーで、前のセクションで作成した **Traffic Manager プロファイル**の名前を検索し、表示された結果から Traffic Manager プロファイルを選択します。

2. **[Traffic Manager プロファイル]** の **[設定]** セクションで、 **[エンドポイント]** を選択します。

3. **[追加]** を選択します。

4. Azure Stack Hub エンドポイントを追加します。

5. **[Type] (種類)** で、 **[外部エンドポイント]** を選択します。

6. このエンドポイントの**名前**を、理想的には Azure Stack Hub の名前を入力します。

7. 完全修飾ドメイン名 (**FQDN**) については、Azure Stack Hub Web アプリの外部 URL を使用します。

8. 地理的マッピングで、リソースが置かれているリージョン/大陸を選択します。 たとえば**ヨーロッパ**を選択します。

9. 表示された [Country/Region]\(国/リージョン\) ドロップダウンで、このエンドポイントに適用される国を選択します。 たとえば**ドイツ**を選択します。

10. **[Add as disabled (無効として追加)]** はオフのままにします。

11. **[OK]** を選択します。

12. Azure エンドポイントの追加:

    1. **[Type] (種類)** で、 **[Azure エンドポイント]** を選択します。

    2. エンドポイントに **[名前]** を入力します。

    3. **[ターゲット リソースの種類]** で、 **[App Service]** を選択します。

    4. **[ターゲット リソース]** で、 **[アプリ サービスの選択]** を選択し、同じサブスクリプションにある Web Apps の一覧を表示します。 **[リソース]** で、最初のエンドポイントとして使用されるアプリ サービスを選択します。

13. 地理的マッピングで、リソースが置かれているリージョン/大陸を選択します。 たとえば、**北米/中央アメリカ/カリブ**を選択します。

14. 表示された [Country/Region]\(国/リージョン\) ドロップダウンで、このスポットを空のまま残すと、上のリージョン グループがすべて選択されます。

15. **[Add as disabled (無効として追加)]** はオフのままにします。

16. **[OK]** を選択します。

    > [!Note]  
    >  [All (World)] (すべて (世界)) の地理的範囲を持つ少なくとも 1 つのエンドポイントを作成して、リソースの既定のエンドポイントとして機能します。

17. 両方のエンドポイントが追加されると、 **[Traffic Manager プロファイル]** に表示され、監視ステータスが **[オンライン]** になります。

    ![Traffic Manager プロファイル エンドポイントの状態](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a>グローバル エンタープライズには Azure の地理的分散機能が必要

Azure Traffic Manager と地域固有のエンドポイントを利用してデータ トラフィックを転送することで、グローバル企業は地域の規制に準拠し、現地/遠隔地を問わず、ビジネスの成功に不可欠であるデータ コンプライアンスとデータ セキュリティを維持できます。

## <a name="next-steps"></a>次のステップ

- Azure のクラウド パターンの詳細については、「[Cloud Design Pattern (クラウド設計パターン)](/azure/architecture/patterns)」を参照してください。
