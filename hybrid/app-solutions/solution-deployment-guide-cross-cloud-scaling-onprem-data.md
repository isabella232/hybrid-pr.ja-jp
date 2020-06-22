---
title: オンプレミス データを使用してクロスクラウドをスケーリングするハイブリッド アプリをデプロイする
description: オンプレミス データを使用し、Azure と Azure Stack Hub でクロスクラウドをスケーリングするアプリをデプロイする方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 75289eae902c5363862e345bdedb97cbcee0476e
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910890"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a>オンプレミス データを使用してクロスクラウドをスケーリングするハイブリッド アプリをデプロイする

このソリューション ガイドでは、Azure と Azure Stack Hub の両方にまたがり、1 つのオンプレミス データ ソースを使用するハイブリッド アプリをデプロイする方法について説明します。

ハイブリッド クラウド ソリューションを使用することで、プライベート クラウドが持つコンプライアンス面でのメリットとパブリック クラウドが持つスケーラビリティとを融合することができます。 また、開発者は、Microsoft デベロッパーのエコシステムを活用し、クラウド環境とオンプレミス環境にそのスキルを活かすこともできます。

## <a name="overview-and-assumptions"></a>概要と前提条件

このチュートリアルに従うことにより、開発者がパブリック クラウドとプライベート クラウドにまったく同じ Web アプリをデプロイできるようにするワークフローを構築します。 このアプリは、プライベート クラウド上にホストされた、インターネットを介したルーティングが不可能なネットワークにアクセスできます。 これらの Web アプリは監視され、トラフィックが急増すると、トラフィックをパブリック クラウドにリダイレクトするように DNS レコードがプログラムによって書き換えられます。 急増前の水準までトラフィックが減少すると、トラフィックは再びプライベート クラウドへルーティングされます。

このチュートリアルに含まれるタスクは次のとおりです。

> [!div class="checklist"]
> - ハイブリッド接続の SQL Server データベース サーバーをデプロイする。
> - グローバル Azure 内の Web アプリをハイブリッド ネットワークに接続する。
> - クラウド間スケーリング向けに DNS を構成する。
> - クラウド間スケーリング向けに SSL 証明書を構成する。
> - Web アプリを構成してデプロイする。
> - Traffic Manager プロファイルを作成し、クラウド間スケーリング向けに構成する。
> - トラフィックの増加に関して Application Insights の監視とアラートを設定する。
> - グローバル Azure と Azure Stack Hub の間で自動トラフィック切り替えを構成する。

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub は Azure の拡張機能です。 Azure Stack Hub により、オンプレミス環境にクラウド コンピューティングの機敏性とイノベーションがもたらされ、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドが可能になります。  
> 
> [ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。 これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。

### <a name="assumptions"></a>前提条件

このチュートリアルは、グローバル Azure と Azure Stack Hub についての基本知識があることを前提にしています。 チュートリアルを開始する前に、より詳しい情報を確認しておきたい場合は、以下の記事をお読みください。

- [Azure 入門](https://azure.microsoft.com/overview/what-is-azure/)
- [Azure Stack Hub の主要概念](/azure-stack/operator/azure-stack-overview.md)

このチュートリアルは、Azure サブスクリプションをお持ちであることも前提としています。 サブスクリプションをお持ちでない場合は、開始する前に[無料アカウントを作成](https://azure.microsoft.com/free/)してください。

## <a name="prerequisites"></a>前提条件

このソリューションを開始する前に、次の要件を満たしてください。

- Azure Stack Development Kit (ASDK) または Azure Stack Hub 統合システムのサブスクリプション。 ASDK をデプロイするには、[インストーラーを使用して ASDK をデプロイする](/azure-stack/asdk/asdk-install.md)方法の手順に従います。
- ご利用の Azure Stack Hub 環境に次のものがインストールされている必要があります。
  - Azure App Service。 Azure Stack Hub のオペレーターと協力して、Azure App Service をご自分の環境にデプロイし、構成してください。 このチュートリアルでは、App Service で専用の worker ロールを少なくとも 1 つ利用できるようにすることが求められます。
  - Windows Server 2016 イメージ。
  - Windows Server 2016 と Microsoft SQL Server イメージ。
  - 適切なプランとオファー。
  - Web アプリのドメイン名。 ドメイン名を持っていない場合、GoDaddy、Bluehost、InMotion などのドメイン プロバイダーから 1 つ購入できます。
- 信頼のおける証明機関 (LetsEncrypt など) から取得したドメインの SSL 証明書。
- SQL Server データベースと通信し、Application Insights をサポートする Web アプリ。 GitHub から [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) サンプル アプリをダウンロードできます。
- Azure 仮想ネットワークと Azure Stack Hub 仮想ネットワークの間のハイブリッド ネットワーク。 詳細な手順については、[Azure と Azure Stack Hub を使用したハイブリッド クラウド接続の構成](solution-deployment-guide-connectivity.md)に関するページを参照してください。

- Azure Stack Hub にプライベート ビルド エージェントが存在する、継続的インテグレーション/継続的デプロイ (CI/CD) のハイブリッド パイプライン。 詳細な手順については、「[Azure および Azure Stack Hub アプリケーションのハイブリッド クラウド ID を構成する](solution-deployment-guide-identity.md)」を参照してください。

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>ハイブリッド接続の SQL Server データベース サーバーをデプロイする

1. Azure Stack Hub ユーザー ポータルにサインインします。

2. **[ダッシュボード]** の **[Marketplace]** を選択します。

    ![Azure Stack Hub Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. **[Marketplace]** で **[Compute]\(計算\)** を選択し、 **[More]\(その他\)** を選択します。 **[More]\(その他\)** から **[Free SQL Server License: SQL Server 2017 Developer on Windows Server]** イメージを選択します。

    ![Azure Stack Hub ユーザー ポータルで仮想マシン イメージを選択する](media/solution-deployment-guide-hybrid/image2.png)

4. **[Free SQL Server License: SQL Server 2017 Developer on Windows Server]** で、 **[作成]** を選択します。

5. **[基本] の [基本設定の構成]** で、仮想マシン (VM) の **[名前]** 、SQL Server SA の **[ユーザー名]** 、SA の **[パスワード]** を入力します。  **[サブスクリプション]** ドロップダウン リストから、デプロイ先のサブスクリプションを選択します。 **[リソース グループ]** では **[Choose existing]\(既存の選択\)** を使用し、Azure Stack Hub Web アプリと同じリソース グループに VM を配置します。

    ![Azure Stack Hub ユーザー ポータルで VM の基本設定を構成する](media/solution-deployment-guide-hybrid/image3.png)

6. **[サイズ]** で VM のサイズを選択します。 このチュートリアルでは、A2_Standard または DS2_V2_Standard をお勧めします。

7. **[設定] の [オプション機能の構成]** で、次の設定を構成します。

   - **ストレージ アカウント**: 新しいアカウントが必要な場合は、作成します。
   - **仮想ネットワーク**:

     > [!Important]  
     > SQL Server VM が VPN ゲートウェイと同じ仮想ネットワーク上にデプロイされていることを確認してください。

   - **[パブリック IP アドレス]** : 既定の設定を使用します。
   - **[ネットワーク セキュリティ グループ]** : (NSG)。 新しい NSG を作成します。
   - **[拡張機能] と [監視]** :既定の設定のままにします。
   - **[診断ストレージ アカウント]** :新しいアカウントが必要な場合は、作成します。
   - **[OK]** を選択して構成を保存します。

     ![Azure Stack Hub ユーザー ポータルでオプションの VM 機能を構成する](media/solution-deployment-guide-hybrid/image4.png)

8. **[SQL Server の設定]** で、次の設定を構成します。

   - **[SQL 接続]** で **[パブリック (インターネット)]** を選択します。
   - **[ポート]** は、既定値 (**1433**) のままにします。
   - **[SQL 認証]** には **[有効]** を選択します。

     > [!Note]  
     > SQL 認証を有効にすると、 **[基本]** で構成した "SQLAdmin" の情報が自動設定されます。

   - その他の設定は、既定値のままにしてください。 **[OK]** を選択します。

     ![Azure Stack Hub ユーザー ポータルで SQL Server 設定を構成する](media/solution-deployment-guide-hybrid/image5.png)

9. **[概要]** で VM の構成を確認し、 **[OK]** を選択してデプロイを開始します。

    ![Azure Stack Hub ユーザー ポータルの構成の概要](media/solution-deployment-guide-hybrid/image6.png)

10. 新しい VM の作成には時間がかかります。 VM の状態は、 **[仮想マシン]** で確認できます。

    ![Azure Stack Hub ユーザー ポータルの仮想マシンの状態](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a>Azure と Azure Stack Hub に Web アプリを作成する

Azure App Service は、Web アプリの実行と管理を簡単にします。 Azure Stack Hub は Azure と一貫性があるため、App Service はどちらの環境でも実行できます。 App Service を使用し、アプリをホストします。

### <a name="create-web-apps"></a>Web アプリを作成する

1. 「[Azure で App Service プランを管理する](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan)」の手順に従って、Azure で Web アプリを作成します。 Web アプリは、ご利用のハイブリッド ネットワークと同じサブスクリプションおよびリソース グループに配置してください。

2. 前の手順 (1) を Azure Stack Hub でも行います。

### <a name="add-route-for-azure-stack-hub"></a>Azure Stack Hub 用のルートを追加する

Azure Stack Hub 上の App Service は、ユーザーがアプリにアクセスできるよう、パブリック インターネットからルーティングできなければなりません。 Azure Stack Hub がインターネットからアクセスできる場合、Azure Stack Hub Web アプリの公開 IP アドレスまたは URL をメモします。

ASDK を使用している場合は、[静的 NAT のマッピングを構成](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal)することで、仮想環境の外部に App Service を公開することができます。

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Azure 内の Web アプリをハイブリッド ネットワークに接続する

Azure の Web フロントエンドと Azure Stack Hub の SQL Server データベースを接続するには、Azure と Azure Stack Hub の間のハイブリッド ネットワークに Web アプリを接続する必要があります。 接続を有効にするためには、次の作業が必要となります。

- ポイント対サイト接続を構成します。
- Web アプリを構成します。
- Azure Stack Hub 内のローカル ネットワーク ゲートウェイに変更を加える

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>ポイント対サイト接続のために Azure 仮想ネットワークを構成する

Azure App Service と統合するためには、ハイブリッド ネットワークの Azure 側にある仮想ネットワーク ゲートウェイでポイント対サイト接続を許可する必要があります。

1. Azure で仮想ネットワーク ゲートウェイ ページに移動します。 **[設定]** で、 **[ポイント対サイトの構成]** を選択します。

    ![Azure 仮想ネットワーク ゲートウェイのポイント対サイト オプション](media/solution-deployment-guide-hybrid/image8.png)

2. **[今すぐ構成]** を選択し、ポイント対サイトを構成します。

    ![Azure 仮想ネットワーク ゲートウェイでポイント対サイトの構成を開始する](media/solution-deployment-guide-hybrid/image9.png)

3. **[ポイント対サイト]** 構成ページで、使用するプライベート IP アドレス範囲を **[アドレス プール]** に入力します。

   > [!Note]  
   > 指定する範囲が、ハイブリッド ネットワークのグローバル Azure コンポーネントまたは Azure Stack Hub コンポーネントのサブネットによって既に使用されているアドレス範囲と重複しないようにしてください。

   **[トンネルの種類]** の **[IKEv2 VPN]** チェック ボックスをオフにします。 **[保存]** を選択して、ポイント対サイトの構成を完了します。

   ![Azure 仮想ネットワーク ゲートウェイのポイント対サイト設定](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a>Azure App Service アプリとハイブリッド ネットワークを統合する

1. Azure VNet にアプリを接続するには、「[ゲートウェイが必要な Vnet 統合](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)」の指示に従ってください。

2. Web アプリをホストしている App Service プランの **[設定]** に移動します。 **[設定]** の **[ネットワーク]** を選択します。

    ![App Service プランのネットワークを構成する](media/solution-deployment-guide-hybrid/image11.png)

3. **[VNET 統合]** の **[管理するにはここをクリック]** を選択します。

    ![App Service プランの VNET 統合を管理する](media/solution-deployment-guide-hybrid/image12.png)

4. 構成する VNET を選択します。 **[IP アドレスが VNET にルーティングされました]** で、Azure VNet、Azure Stack Hub VNet、ポイント対サイトのアドレス空間に使用する IP アドレス範囲を入力します。 **[保存]** を選択し、これらの設定を確認して保存します。

    ![Virtual Network 統合でルーティングする IP アドレス範囲](media/solution-deployment-guide-hybrid/image13.png)

App Service と Azure VNet の統合方法の詳細については、「[アプリを Azure 仮想ネットワークに統合する](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet)」を参照してください。

### <a name="configure-the-azure-stack-hub-virtual-network"></a>Azure Stack Hub 仮想ネットワークを構成する

App Service のポイント対サイト アドレス範囲からのトラフィックをルーティングするために、Azure Stack Hub 仮想ネットワーク内のローカル ネットワーク ゲートウェイを構成する必要があります。

1. Azure Stack Hub で **[ローカル ネットワーク ゲートウェイ]** に移動します。 **[設定]** で **[構成]** を選択します。

    ![Azure Stack Hub ローカル ネットワーク ゲートウェイのゲートウェイ構成オプション](media/solution-deployment-guide-hybrid/image14.png)

2. **[アドレス空間]** に、Azure の仮想ネットワーク ゲートウェイのポイント対サイト アドレス範囲を入力します。

    ![Azure Stack Hub ローカル ネットワーク ゲートウェイのポイント対サイト アドレス空間](media/solution-deployment-guide-hybrid/image15.png)

3. **[保存]** を選択し、構成を検証して保存します。

## <a name="configure-dns-for-cross-cloud-scaling"></a>クラウド間スケーリング向けに DNS を構成する

クラウド間アプリ向けに DNS を適切に構成することで、ユーザーはグローバル Azure と Azure Stack Hub の Web アプリ インスタンスにアクセスできます。 また、このチュートリアルの DNS 構成を使えば、負荷が増減したときに Azure Traffic Manager でトラフィックをルーティングすることも可能です。

App Service ドメインが機能しないため、このチュートリアルでは Azure DNS を使用して DNS を管理します。

### <a name="create-subdomains"></a>サブドメインを作成する

Traffic Manager は DNS の CNAME に依存しているため、エンドポイントに対して適切にトラフィックをルーティングするためには、サブドメインが必要となります。 DNS レコードとドメイン マッピングの詳細については、「[Traffic Manager を使用したドメインのマップ](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name)」を参照してください。

Azure エンドポイントについては、Web アプリにアクセスするためにユーザーが使用できるサブドメインを作成します。 このチュートリアルでは、**app.northwind.com** を使用できますが、この値はご自身のドメインに合わせてカスタマイズする必要があります。

また、Azure Stack Hub エンドポイントについても、A レコードを使用してサブドメインを作成する必要があります。 **azurestack.northwind.com** を使用できます。

### <a name="configure-a-custom-domain-in-azure"></a>Azure でカスタム ドメインを構成する

1. [Azure App Service に CNAME をマップ](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)して、**app.northwind.com** ホスト名を Azure Web アプリに追加します。

### <a name="configure-custom-domains-in-azure-stack-hub"></a>Azure Stack Hub でカスタム ドメインを構成する

1. [Azure App Service に A レコードをマップ](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record)して、ホスト名 **azurestack.northwind.com** を Azure Stack Hub Web アプリに追加します。 App Service アプリには、インターネット ルーティング可能な IP アドレスを使用します。

2. [Azure App Service に CNAME をマップ](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)して、ホスト名 **app.northwind.com** を Azure Stack Hub Web アプリに追加します。 CNAME のターゲットとして、前の手順 (1) で構成したホスト名を使用してください。

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>クラウド間スケーリング向けに SSL 証明書を構成する

Web アプリによって収集される機密データを移動中および SQL データベースで保管されている間、セキュリティで保護することが重要です。

すべての受信トラフィックについて SSL 証明書を使用するように、Azure の Web アプリと Azure Stack Hub の Web アプリを構成します。

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a>Azure と Azure Stack Hub に SSL を追加する

Azure に SSL を追加するには、次の手順に従います。

1. 作成したサブドメインに対し、取得した SSL 証明書が有効であることを確認します  (ワイルドカード証明書を使用してもかまいません)。

2. Azure で、[Azure Web Apps に既存のカスタム SSL 証明書をバインドする](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl)方法に関する記事の「**Web アプリの準備**」と **SSL 証明書のバインド**に関するセクションの指示に従います。 **[SSL の種類]** として **[SNI ベースの SSL]** を選択します。

3. すべてのトラフィックを HTTP ポートにリダイレクトします。 [Azure Web Apps への既存のカスタム SSL 証明書のバインド](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl)に関する記事のセクション「**HTTPS の適用**」の手順に従ってください。

Azure Stack Hub に SSL を追加するには、次の手順に従います。

1. Azure で使用した手順 1. から手順 3. を繰り返します。

## <a name="configure-and-deploy-the-web-app"></a>Web アプリを構成し、デプロイする

テレメトリを正しい Application Insights インスタンスに報告するようにアプリ コードを構成し、正しい接続文字列で Web アプリを構成します。 Application Insights の詳細については、「[Application Insights とは何か?](https://docs.microsoft.com/azure/application-insights/app-insights-overview)」を参照してください。

### <a name="add-application-insights"></a>Application Insights を追加する

1. Microsoft Visual Studio で Web アプリを開きます。

2. プロジェクトに [Application Insights を追加](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications)し、Web トラフィックが増減したときのアラートを生成するために Application Insights によって使用されるテレメトリが転送されるようにします。

### <a name="configure-dynamic-connection-strings"></a>動的接続文字列を構成する

Web アプリの各インスタンスでは、異なる方法を使用して SQL データベースに接続します。 Azure のアプリでは SQL Server VM のプライベート IP アドレスが使用され、Azure Stack Hub のアプリでは SQL Server VM のパブリック IP アドレスが使用されます。

> [!Note]  
> Azure Stack Hub 統合システムでは、パブリック IP アドレスをインターネット ルーティング可能にしないでください。 ASDK では、パブリック IP アドレスは ASDK の外部にルーティングできません。

App Service 環境変数を使用し、アプリの各インスタンスに異なる接続文字列を渡すことができます。

1. Visual Studio でアプリを開きます。

2. Startup.cs を開いて、次のコード ブロックを見つけます。

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. 前のコード ブロックを次のコードに置き換えます。このコードでは、*appsettings.json* ファイルで定義されている接続文字列が使用されます。

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a>App Service アプリ設定を構成する

1. Azure と Azure Stack Hub 用の接続文字列を作成します。 IP アドレス以外は、同じ文字列を使用してください。

2. Azure と Azure Stack Hub で、Web アプリの[アプリ設定として](https://docs.microsoft.com/azure/app-service/web-sites-configure)適切な接続文字列を追加します。そのとき、名前のプレフィックスとして `SQLCONNSTR\_` を使用します。

3. Web アプリ設定を**保存**し、アプリを再起動します。

## <a name="enable-automatic-scaling-in-global-azure"></a>グローバル Azure で自動スケーリングを有効にする

App Service 環境で Web アプリを作成するとき、1 つのインスタンスから始めます。 自動的にスケールアウトしてインスタンスを追加することで、アプリ用のコンピューティング リソースを増やすことができます。 同様に、自動的にスケールインして、アプリに必要なインスタンスの数を減らすことができます。

> [!Note]  
> スケールアウトとスケールインを構成するには、App Service プランが必要です。 プランをお持ちでない場合は、作成したうえで次の手順を開始してください。

### <a name="enable-automatic-scale-out"></a>自動スケールアウトを有効にする

1. Azure で、スケールアウトしたいサイトの App Service プランを見つけて、 **[スケールアウト (App Service プラン)]** を選択します。

    ![Azure App Service をスケールアウトする](media/solution-deployment-guide-hybrid/image16.png)

2. **[自動スケールの有効化]** を選択します。

    ![Azure App Service で自動スケーリングを有効にする](media/solution-deployment-guide-hybrid/image17.png)

3. **[自動スケール設定の名前]** に名前を入力します。 **既存**の自動スケール ルールで、 **[メトリックに基づいてスケーリングする]** を選択します。 **[インスタンスの制限]** で、 **[最小]** を 1、 **[最大]** を 10、 **[既定]** を 1 に設定します。

    ![Azure App Service で自動スケーリングを構成する](media/solution-deployment-guide-hybrid/image18.png)

4. **[+ ルールの追加]** を選択します。

5. **[メトリック ソース]** で **[現在のリソース]** を選択します。 このルールには、次の条件とアクションを使用します。

#### <a name="criteria"></a>条件

1. **[時間の集計]** で **[平均]** を選択します。

2. **[メトリック名]** で **[CPU の割合]** を選択します。

3. **[演算子]** で **[より大きい]** を選択します。

   - **[しきい値]** を **50** に設定します。
   - **[期間]** を **10** に設定します。

#### <a name="action"></a>アクション

1. **[操作]** で **[カウントを増やす量]** を選択します。

2. **[インスタンス数]** を **2** に設定します。

3. **[クール ダウン]** を **5** に設定します。

4. **[追加]** を選択します。

5. **[+ ルールの追加]** を選択します。

6. **[メトリック ソース]** で **[現在のリソース]** を選択します。

   > [!Note]  
   > 現在のリソースには、App Service プランの名前/GUID が表示され、 **[リソースの種類]** ドロップダウン リストと **[リソース]** ドロップダウン リストは利用できません。

### <a name="enable-automatic-scale-in"></a>自動スケールインを有効にする

トラフィックが減ると、Azure Web アプリでは、アクティブ インスタンスの数を自動的に減らし、コストを減らすことができます。 このアクションはスケールアウトより消極的であり、アプリ ユーザーへの影響を最小限に抑えます。

1. **[既定]** のスケールアウト条件に移動し、 **[+ ルールの追加]** を選択します。 このルールには、次の条件とアクションを使用します。

#### <a name="criteria"></a>条件

1. **[時間の集計]** で **[平均]** を選択します。

2. **[メトリック名]** で **[CPU の割合]** を選択します。

3. **[演算子]** で **[より小さい]** を選択します。

   - **[しきい値]** を **30** に設定します。
   - **[期間]** を **10** に設定します。

#### <a name="action"></a>アクション

1. **[操作]** で **[カウントを減らす量]** を選択します。

   - **[インスタンス数]** を **1** に設定します。
   - **[クール ダウン]** を **5** に設定します。

2. **[追加]** を選択します。

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Traffic Manager プロファイルを作成し、クラウド間スケーリングを構成する

Azure で Traffic Manager プロファイルを作成し、クラウド間スケーリングを有効にするようにエンドポイントを構成します。

### <a name="create-traffic-manager-profile"></a>Traffic Manager プロファイルを作成する

1. **[リソースの作成]** を選択します。
2. **[ネットワーク]** を選択します。
3. **[Traffic Manager プロファイル]** を選択し、次の設定を構成します。

   - **[名前]** に、プロファイルの名前を入力します。 この名前は、trafficmanager.net ゾーン内で一意であることが**必要**です。新しい DNS 名を作成するときに使用されます (例: northwindstore.trafficmanager.net)。
   - **[ルーティング方法]** で **[重み付け]** を選択します。
   - **[サブスクリプション]** で、このプロファイルを作成するサブスクリプションを選択します。
   - **[リソース グループ]** で、このプロファイルの新しいリソース グループを作成します。
   - **[リソース グループの場所]** で、リソース グループの場所を選択します。 これはリソース グループの場所を指定する設定であり、グローバルにデプロイされる Traffic Manager プロファイルには影響しません。

4. **［作成］** を選択します

    ![Traffic Manager プロファイルを作成する](media/solution-deployment-guide-hybrid/image19.png)

   Traffic Manager プロファイルのグローバル デプロイが完了すると、その作成先となったリソース グループのリソース一覧にそのプロファイルが表示されます。

### <a name="add-traffic-manager-endpoints"></a>Traffic Manager エンドポイントの追加

1. 作成した Traffic Manager プロファイルを検索します  プロファイルのリソース グループに移動した場合は、プロファイルを選択してください。

2. **[Traffic Manager プロファイル]** の **[設定]** で、 **[エンドポイント]** を選択します。

3. **[追加]** を選択します。

4. Azure Stack Hub について、 **[エンドポイントの追加]** で次の設定を使用します。

   - **[Type] (種類)** で、 **[外部エンドポイント]** を選択します。
   - エンドポイントの **[名前]** を入力します。
   - **完全修飾ドメイン名 (FQDN) または IP** として、Azure Stack Hub Web アプリの外部 URL を入力します。
   - **[重み]** は、既定値 (**1**) のままにします。 これにより、このエンドポイントが正常な状態である場合、すべてのトラフィックがそのエンドポイントに送信されるようになります。
   - **[無効として追加]** はオフのままにします。

5. **[OK]** を選択して、Azure Stack Hub エンドポイントを保存します。

次に、Azure エンドポイントを構成します。

1. **[Traffic Manager プロファイル]** で **[エンドポイント]** を選択します。
2. **[+追加]** を選択します。
3. Azure について、 **[エンドポイントの追加]** で次の設定を使用します。

   - **[Type] (種類)** で、 **[Azure エンドポイント]** を選択します。
   - エンドポイントの **[名前]** を入力します。
   - **[ターゲット リソースの種類]** で、 **[App Service]** を選択します。
   - **[ターゲット リソース]** で **[アプリ サービスの選択]** を選択し、同じサブスクリプションにある Web アプリの一覧を表示します。
   - **[リソース]** で、最初のエンドポイントとして追加する App Service を選択します。
   - **[重み]** に **2** を選択します。 この設定により、プライマリ エンドポイントが正常ではない場合や、トリガーされたらトラフィックをルーティングするルール/アラートがある場合、すべてのトラフィックがそのエンドポイントに送信されるようになります。
   - **[無効として追加]** はオフのままにします。

4. **[OK]** を選択して、Azure エンドポイントを保存します。

構成したエンドポイントはどちらも、 **[Traffic Manager プロファイル]** で **[エンドポイント]** を選択すると表示されます。 次の画面キャプチャの例には、2 つのエンドポイントが、それぞれの状態および構成情報と共に表示されています。

![Traffic Manager プロファイルのエンドポイント](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a>Application Insights の監視とアラートを設定する

Azure Application Insights を使用すると、アプリを監視し、構成した条件に応じてアラートを送信できます。 たとえば、アプリが利用できなくなった、障害が発生した、パフォーマンスの問題が生じたなどの例があります。

アラートの作成には、Application Insights のメトリックを使用します。 これらのアラートがトリガーされると、Web アプリ インスタンスが自動的に Azure Stack Hub から Azure に切り替わってスケールアウトし、その後、Azure Stack Hub に戻ってスケールインします。

### <a name="create-an-alert-from-metrics"></a>メトリックに基づくアラートを作成する

このチュートリアルのリソース グループに移動して Application Insights インスタンスを選択し、 **[Application Insights]** を開きます。

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

このビューを使用してスケールアウト アラートとスケールイン アラートを作成します。

### <a name="create-the-scale-out-alert"></a>スケールアウト アラートを作成する

1. **[構成]** で **[アラート (クラシック)]** を選択します。
2. **[メトリック アラートの追加 (クラシック)]** を選択します。
3. **[ルールの追加]** で、次の設定を構成します。

   - **[名前]** に「**Burst into Azure Cloud**」と入力します。
   - **[説明]** は省略できます。
   - **[ソース]** の **[アラート対象]** で **[メトリック]** を選択します。
   - **[条件]** で、自分のサブスクリプション、Traffic Manager プロファイルのリソース グループ、リソースに使用する Traffic Manager プロファイルの名前を選択します。

4. **[メトリック]** で **[要求率]** を選択します。
5. **[条件]** で **[より大きい]** を選択します。
6. **[しきい値]** に「**2**」を入力します。
7. **[期間]** で **[直近 5 分]** を選択します。
8. **[通知手段]** で次のように設定します。
   - **[所有者、共同作成者、閲覧者に電子メールを送信]** のチェック ボックスをオンにします。
   - **[追加する管理者の電子メール]** にメール アドレスを入力します。

9. メニュー バーで **[保存]** を選択します。

### <a name="create-the-scale-in-alert"></a>スケールイン アラートを作成する

1. **[構成]** で **[アラート (クラシック)]** を選択します。
2. **[メトリック アラートの追加 (クラシック)]** を選択します。
3. **[ルールの追加]** で、次の設定を構成します。

   - **[名前]** に「**Scale back into Azure Stack Hub**」と入力します。
   - **[説明]** は省略できます。
   - **[ソース]** の **[アラート対象]** で **[メトリック]** を選択します。
   - **[条件]** で、自分のサブスクリプション、Traffic Manager プロファイルのリソース グループ、リソースに使用する Traffic Manager プロファイルの名前を選択します。

4. **[メトリック]** で **[要求率]** を選択します。
5. **[条件]** で **[より小さい]** を選択します。
6. **[しきい値]** に「**2**」を入力します。
7. **[期間]** で **[直近 5 分]** を選択します。
8. **[通知手段]** で次のように設定します。
   - **[所有者、共同作成者、閲覧者に電子メールを送信]** のチェック ボックスをオンにします。
   - **[追加する管理者の電子メール]** にメール アドレスを入力します。

9. メニュー バーで **[保存]** を選択します。

次のスクリーンショットには、スケールアウトとスケールインのアラートが示されています。

   ![Application Insights のアラート (クラシック)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a>Azure と Azure Stack Hub の間でトラフィックをリダイレクトする

Azure と Azure Stack Hub の間で行われる Web アプリのトラフィックには、手動または自動の切り替えを構成できます。

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a>Azure と Azure Stack Hub の間で手動切り替えを構成する

Web サイトが構成済みのしきい値に達した場合、アラートが届きます。 トラフィックを手動で Azure にリダイレクトするには、次の手順を使用します。

1. Azure portal で、該当する Traffic Manager プロファイルを選択します。

    ![Azure portal の Traffic Manager エンドポイント](media/solution-deployment-guide-hybrid/image20.png)

2. **[エンドポイント]** を選択します。
3. **[Azure エンドポイント]** を選択します。
4. **[状態]** で **[有効]** を選択し、 **[保存]** を選択します。

    ![Azure portal で Azure エンドポイントを有効にする](media/solution-deployment-guide-hybrid/image23.png)

5. Traffic Manager プロファイルの **[エンドポイント]** で、 **[外部エンドポイント]** を選択します。
6. **[状態]** で **[無効]** を選択し、 **[保存]** を選択します。

    ![Azure portal で Azure Stack Hub エンドポイントを無効にする](media/solution-deployment-guide-hybrid/image24.png)

エンドポイントの構成後、アプリ トラフィックは、Azure Stack Hub Web アプリではなく、Azure スケールアウト Web アプリに送信されます。

 ![Azure web アプリのトラフィックで変更されたエンドポイント](media/solution-deployment-guide-hybrid/image25.png)

フローを再び Azure Stack Hub に戻すには、前の手順を使用して次の設定を行います。

- Azure Stack Hub エンドポイントを有効にします。
- Azure エンドポイントを無効にします。

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a>Azure と Azure Stack Hub の間で自動切り替えを構成する

Azure Functions によって実現される[サーバーレス](https://azure.microsoft.com/overview/serverless-computing/)環境で対象のアプリが実行されている場合は、Application Insights の監視を使用することもできます。

このシナリオでは、関数アプリを呼び出す Webhook を使用するように Application Insights を構成することができます。 このアプリでは、アラートに応じてエンドポイントの有効と無効を自動的に切り替えます。

次の手順を参考にして、自動トラフィック切り替えを構成してください。

1. Azure 関数アプリを作成します。
2. HTTP によってトリガーされる関数を作成します。
3. Resource Manager、Web Apps、Traffic Manager 用の Azure SDK をインポートします。
4. 次の処理を行うコードを作成します。

   - Azure サブスクリプションに対して認証を行う。
   - Traffic Manager のエンドポイントを切り替えるパラメーターを使用して、Azure または Azure Stack Hub にトラフィックを送信する。

5. 作成したコードを保存し、Application Insights のアラート ルール設定の **Webhook** セクションに、適切なパラメーターと共に関数アプリの URL を追加します。
6. Application Insights のアラートが発生すると、トラフィックが自動的にリダイレクトされます。

## <a name="next-steps"></a>次のステップ

- Azure のクラウド パターンの詳細については、「[Cloud Design Pattern (クラウド設計パターン)](https://docs.microsoft.com/azure/architecture/patterns)」を参照してください。
