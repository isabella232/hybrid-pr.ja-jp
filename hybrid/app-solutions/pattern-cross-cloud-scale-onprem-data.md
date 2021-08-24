---
title: Azure Stack Hub のクラウド間スケーリング (オンプレミスのデータ) パターン
description: Azure と Azure Stack Hub でオンプレミスのデータを使用するスケーラブルなクラウド間アプリを構築する方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5c8e3adb621ae4322bf6d60792fc307dbb24ff90
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281246"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>クラウド間スケーリング (オンプレミスのデータ) パターン

Azure と Azure Stack Hub にまたがるハイブリッド アプリを構築する方法について説明します。 このパターンでは、単一のオンプレミス データ ソースをコンプライアンスに使用する方法についても説明します。

## <a name="context-and-problem"></a>コンテキストと問題

多くの組織では、大量の機密顧客データを収集して保存しています。 多くの場合、会社の規制や政府のポリシーにより、機密データをパブリック クラウドに格納することはできません。 その一方で、これらの組織は、パブリック クラウドのスケーラビリティを活用したいとも考えています。 パブリック クラウドは、トラフィックの季節的なピークに対処できるため、顧客は必要なときに必要なハードウェアの料金を支払うだけで済みます。

## <a name="solution"></a>解決策

このソリューションを使用すると、プライベート クラウドのコンプライアンス面でのメリットとパブリック クラウドのスケーラビリティとを融合させることができます。 Azure と Azure Stack Hub のハイブリッド クラウドは、開発者に一貫したエクスペリエンスを提供します。 この一貫性により、彼らのスキルをパブリック クラウドとオンプレミスの両方の環境に応用できます。

ソリューション デプロイ ガイドを使用すると、同じ Web アプリをパブリック クラウドとプライベート クラウドにデプロイできます。 また、プライベート クラウド上にホストされた、インターネットを介したルーティングが不可能なネットワークにアクセスできます。 Web アプリの負荷が監視されます。 トラフィックが大幅に増加すると、プログラムは DNS レコードを操作して、トラフィックをパブリック クラウドにリダイレクトします。 トラフィックが大量ではなくなると、DNS レコードは更新され、トラフィックは再びプライベート クラウドに送信されるようになります。

[![オンプレミスのデータを使用したクラウド間スケーリング パターン](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Components

このソリューションでは、次のコンポーネントを使用します。

| レイヤー | コンポーネント | 説明 |
|----------|-----------|-------------|
| Azure | Azure App Service | [Azure App Service](/azure/app-service/) を使用すると、Web アプリ、RESTful API アプリ、および Azure Functions を構築してホストすることができます。 すべて任意のプログラミング言語で行うことができ、インフラストラクチャを管理する必要はありません。 |
| | Azure Virtual Network| [Azure Virtual Network (VNet)](/azure/virtual-network/virtual-networks-overview) は、Azure 内のプライベート ネットワークの基本的な構成ブロックです。 VNet により、仮想マシン (VM) などのさまざまな種類の Azure リソースは、他の Azure リソース、インターネット、およびオンプレミスのネットワークと安全に通信することができます。 また、このソリューションでは、追加のネットワーク コンポーネントの使用方法も示します。<br>- アプリとゲートウェイ サブネット。<br>- ローカルのオンプレミス ネットワーク ゲートウェイ。<br>- サイト間 VPN ゲートウェイ接続として機能する仮想ネットワーク ゲートウェイ。<br>- パブリック IP アドレス。<br>- ポイント対サイト VPN 接続。<br>- DNS ドメインをホストし、名前解決を提供するための Azure DNS。 |
| | Azure Traffic Manager | [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) は、DNS ベースのトラフィック ロード バランサーです。 これを使用すると、さまざまなデータセンターのサービス エンドポイントへのユーザー トラフィックの分散を制御できます。 |
| | Azure Application Insights | [Application Insights](/azure/azure-monitor/app/app-insights-overview) は、複数のプラットフォームでアプリを構築および管理する Web 開発者向けの拡張可能なアプリケーション パフォーマンス管理サービスです。|
| | Azure Functions | [Azure Functions](/azure/azure-functions/) を使用すると、最初に VM を作成したり、Web アプリを公開したりしなくても、サーバーレス環境でコードを実行できます。 |
| | Azure の自動スケール | [自動スケーリング](/azure/azure-monitor/platform/autoscale-overview)は、Cloud Services、VM、および Web アプリの組み込み機能です。 この機能を使用すると、アプリは、需要の変化に応じて最高のパフォーマンスを発揮できます。 アプリは、トラフィックの急増に合わせて調整を行い、メトリクスが変化すると通知し、必要に応じてスケーリングします。 |
| Azure Stack Hub | IaaS コンピューティング | Azure Stack Hub では、Azure で有効なものと同一のアプリ モデル、セルフサービス ポータル、および API を使用できます。 Azure Stack Hub IaaS では、一貫性のあるハイブリッド クラウド デプロイに対する幅広いオープン ソース テクノロジを使用できます。 たとえば、このソリューション例では、SQL Server に Windows Server VM を使用しています。|
| | Azure App Service | Azure Web アプリと同様、このソリューションでは、[Azure Stack Hub 上の Azure App Service](/azure-stack/operator/azure-stack-app-service-overview) を使用して Web アプリをホストします。 |
| | ネットワーク | Azure Stack Hub Virtual Network は、Azure Virtual Network とまったく同様に機能します。 カスタム ホスト名など、使用する多くのネットワーク コンポーネントも同じです。
| Azure DevOps Services | サインアップ | ビルド、テスト、およびデプロイのための継続的な統合を迅速に設定します。 詳細については、「[Sign up, sign in to Azure DevOps](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops)」 (Azure DevOps へのサインアップ、サインイン) を参照してください。 |
| | Azure Pipelines | 継続的インテグレーション/継続的デリバリーに [Azure Pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops)を使用します。 Azure Pipelines を使用すると、ホストされるビルド、リリース エージェント、およびリリース定義を管理できます。 |
| | コード リポジトリ | 複数のコード リポジトリを利用して、開発パイプラインを効率化します。 GitHub、Bitbucket、Dropbox、OneDrive、Azure Repos 内の既存のコード リポジトリを使用します。 |

## <a name="issues-and-considerations"></a>問題と注意事項

このソリューションの実装方法を決めるときには、以下の点を考慮してください。

### <a name="scalability"></a>スケーラビリティ

Azure と Azure Stack Hub は、今日のグローバルに分散したビジネスのニーズに対応するのに非常に適しています。

#### <a name="hybrid-cloud-without-the-hassle"></a>簡単に利用可能なハイブリッド クラウド

Microsoft は、オンプレミスの資産と Azure Stack Hub および Azure を統合した、他に類のない単一の統合ソリューションを提供します。 この統合により、複数のポイント ソリューションやさまざまなクラウド プロバイダーを管理する煩雑さを軽減します。 クラウド間スケーリングにより、数回のクリックだけで Azure の機能を利用できます。 クラウド バーストを使用して Azure Stack Hub を Azure に接続するだけで、必要に応じて自分のデータとアプリを Azure で使用できるようになります。

- セカンダリ DR サイトを構築して保守する必要はありません。
- テープによるバックアップを排除することで時間とコストを節約し、最大 99 年分のバックアップデータを Azure に保存できます。
- 実行中の Hyper-V、物理サーバー (プレビュー)、VMware (プレビュー) のワークロードを簡単に Azure に移行し、クラウドの経済性と弾力性を活用できます。
- 運用ワークロードに影響を与えることなく、Azure 内のオンプレミス資産の複製されたコピーに対してコンピューティング負荷の高いレポート生成や分析を実行できます。
- オンプレミスのワークロードをクラウドにバーストして、Azure で実行することができます。必要に応じてより大規模なコンピューティング テンプレートを使用することもできます。 ハイブリッドは、必要なときに必要な能力を提供します。
- Azure では、わずか数回のクリックで多層開発環境を作成できます。また、ライブ実稼働データを開発/テスト環境にレプリケートし、ほぼリアルタイムで同期することもできます。

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Azure Stack Hub によるクラウド間スケーリングの経済性

クラウド バーストの主な利点は、経済的な節約です。 リソースに対する需要がある場合、追加リソースについてのみ料金が発生します。 必要のない余分な容量にコストをかけることや、需要のピークと変動を予測することが不要になります。

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>クラウドに対する高需要の負荷を軽減

クラウド間スケーリングを使用して、処理の負荷を分散させることができます。 基本アプリをパブリック クラウドに移行し、ビジネスクリティカルなアプリのためにローカル リソースを解放することで、負荷を分散します。 アプリをプライベート クラウドで動作させ、需要への対処が必要なときにのみパブリック クラウドにバーストできます。

### <a name="availability"></a>可用性

グローバルなデプロイには特有の課題があります。たとえば、接続の変動が大きいこと、政府の規制が地域によって異なることなどがあります。 開発者は、1 つのアプリだけを開発し、要件が異なるさまざまな理由でデプロイすることができます。 アプリを Azure パブリック クラウドにデプロイし、追加のインスタンスまたはコンポーネントをローカルにデプロイします。 Azure を使用すると、すべてのインスタンス間のトラフィックを管理できます。

### <a name="manageability"></a>管理の容易性

#### <a name="a-single-consistent-development-approach"></a>単一の一貫した開発アプローチ

Azure と Azure Stack Hub を使用すると、組織全体で一貫した一連の開発ツールを使用できます。 この一貫性により、継続的インテグレーションおよび継続的開発 (CI/CD) のプラクティスをより簡単に実装できます。 Azure または Azure Stack Hub にデプロイされた多くのアプリとサービスは互換性があり、どちらの場所でもシームレスに実行できます。

ハイブリッド CI/CD パイプラインは、次の場合に役立ちます。

- コード リポジトリへのコード コミットに基づいて、新しいビルドを開始します。
- ユーザー受け入れテストのため、新しくビルドされたコードを Azure に自動的にデプロイします。
- コードがテストを通過すると、自動的に Azure Stack Hub にデプロイします。

### <a name="a-single-consistent-identity-management-solution"></a>単一の一貫した ID 管理ソリューション

Azure Stack Hub は、Azure Active Directory (Azure AD) および Active Directory フェデレーション サービス (AD FS) の両方と連携します。 Azure Stack Hub では、接続されたシナリオで Azure AD と連携します。 接続されていない環境では、ADFS を、切断されたソリューションとして使用できます。 サービス プリンシパルを使用してアプリへのアクセスが許可され、Azure Resource Manager を介してリソースをデプロイまたは構成することができます。

### <a name="security"></a>セキュリティ

#### <a name="ensure-compliance-and-data-sovereignty"></a>コンプライアンスの確保とデータの主権

Azure Stack Hub により、パブリック クラウドを使用する場合と同様、複数の国で同じサービスを実行できます。 各国のデータセンターに同じアプリをデプロイすることで、データの主権要件を満たすことができます。 この機能により、個人データを各国の境界内で保持することができます。

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack Hub - セキュリティ体制

セキュリティ体制には、堅牢で継続的なサービス プロセスが不可欠です。 そのため、Microsoft では、インフラストラクチャ全体にシームレスに修正プログラムや更新プログラムを適用するオーケストレーション エンジンに投資しました。

Azure Stack Hub OEM パートナーとのパートナーシップにより、Microsoft は同じセキュリティ体制を OEM 固有のコンポーネント (ハードウェア ライフサイクル ホストや、その上で実行されるソフトウェアなど) に拡張しています。 このパートナーシップにより、Azure Stack Hub に、インフラストラクチャ全体で一貫した堅牢なセキュリティ体制を確立できます。 このため、お客様は、アプリのワークロードを構築し、セキュリティで保護することができます。

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>PowerShell、CLI、Azure portal を介したサービス プリンシパルの使用

スクリプトまたはアプリに対してリソースへのアクセスを許可するには、アプリの ID を設定し、アプリ自身の資格情報でアプリを認証できます。 この ID はサービス プリンシパルと呼ばれており、これを使用して次のことが可能になります。

- アプリ ID に対して、お客様自身のアクセス許可とは異なり、かつアプリのニーズだけに制限されたアクセス許可を割り当てることができます。
- 無人スクリプトを実行するときに、証明書を使用して認証できます。

サービス プリンシパルを作成する方法、および資格証明に証明書を使用する方法の詳細については、「[アプリ ID を使用してリソースにアクセスする](/azure-stack/operator/azure-stack-create-service-principals)」を参照してください。

## <a name="when-to-use-this-pattern"></a>このパターンを使用する状況

- 組織は、DevOps アプローチを既に使用しているか、または近い将来にそれを予定しています。
- Azure Stack Hub の実装およびパブリック クラウド全体で、CI/CD プラクティスを実装したいと考えています。
- クラウド環境とオンプレミス環境にまたがって CI/CD パイプラインを統合したいと考えています。
- クラウド サービスまたはオンプレミスのサービスを使用してシームレスにアプリを開発できる必要があります。
- クラウドおよびオンプレミスのアプリで一貫した開発者スキルを活用したいと考えています。
- Azure を使用していますが、オンプレミスの Azure Stack Hub クラウドで作業している開発者もいます。
- オンプレミスのアプリ エクスペリエンスでは、季節的な変動、周期的な変動、または予測できない変動により需要が急増します。
- オンプレミスのコンポーネントがあり、クラウドを使用してそれらをシームレスにスケーリングしたいと考えています。
- クラウドのスケーラビリティが必要ですが、アプリは可能な限りオンプレミスで実行したいと考えています。

## <a name="next-steps"></a>次のステップ

この記事で紹介したトピックの関連情報:

- このパターンの使用方法の概要については、「[Dynamically scale apps between datacenters and public cloud](https://www.youtube.com/watch?v=2lw8zOpJTn0)」(データセンターとパブリック クラウド間でアプリを動的にスケーリングする) を参照してください。
- ベスト プラクティスの詳細とその他の疑問の回答を確認するには、「[ハイブリッド アプリケーション設計に関する考慮事項](overview-app-design-considerations.md)」を参照してください。
- このパターンでは、Azure Stack Hub を含む Azure Stack 製品ファミリを使用します。 製品とソリューションのポートフォリオ全体の詳細について、[Azure Stack ファミリの製品とソリューション](/azure-stack)を参照してください。

ソリューションの例をテストする準備ができたら、[クラウド間スケーリング (オンプレミスのデータ) のソリューション デプロイ ガイド](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling-onprem-data)に進んでください。 デプロイ ガイドでは、コンポーネントをデプロイしてテストするための詳細な手順について説明します。