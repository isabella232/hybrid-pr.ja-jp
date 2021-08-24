---
title: Azure と Azure Stack Hub を使用した高可用性 Kubernetes パターン
description: Kubernetes クラスター ソリューションで Azure と Azure Stack Hub を使用して高可用性を実現する方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: f8a733bcdab871695e552ec687d42e3ff4230490
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281314"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>高可用性 Kubernetes クラスター パターン

この記事では、Azure Stack Hub 上で Azure Kubernetes Service (AKS) エンジンを使用して、高可用性 Kubernetes ベースのインフラストラクチャを設計し、運用する方法について説明します。 このシナリオは、非常に制限の厳しい規制された環境において重要なワークロードを持つ組織で一般的です。 金融、防衛、政府機関などの領域に属する組織です。

## <a name="context-and-problem"></a>コンテキストと問題

多くの組織では、Kubernetes のような最先端のサービスとテクノロジを利用するクラウドネイティブ ソリューションを開発しています。 Azure によって世界中のほとんどのリージョンにデータセンターが提供されますが、ビジネス クリティカルなアプリケーションを特定の場所で実行する必要があるエッジ ユースケースやシナリオがある場合もあります。 考慮事項は次のとおりです。

- 場所の機密度
- アプリケーションとオンプレミス システムの間の待機時間
- 帯域幅の節約
- 接続
- 規制または法定要件

Azure を Azure Stack Hub と組み合わせると、これらの問題のほとんどに対処できます。 Azure Stack Hub 上で実行されている Kubernetes を正常に実装するためのさまざまなオプション、決定事項、および考慮事項について、以下で説明します。

## <a name="solution"></a>解決策

このパターンは、厳格な制約セットを処理する必要があることを前提としています。 アプリケーションはオンプレミスで実行する必要があり、すべての個人データがパブリック クラウド サービスに届かないようにする必要があります。 監視とその他の非 PII データを Azure に送信し、そこで処理することができます。 パブリック Container Registry などの外部サービスにはアクセスできますが、ファイアウォールまたはプロキシ サーバーを通じてフィルター処理することができます。

ここに示されているサンプル アプリケーション ([Azure Kubernetes Service ワークショップ](/learn/modules/aks-workshop/)) は、可能な限り Kubernetes ネイティブ ソリューションを使用するように設計されています。 この設計では、プラットフォームネイティブ サービスを使用する代わりに、ベンダー ロックインを回避します。 たとえば、アプリケーションでは、PaaS サービスや外部データベース サービスではなく、セルフホステッド MongoDB データベース バックエンドを使用します。

[![アプリケーション パターン ハイブリッド](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

上の図は Azure Stack Hub 上の Kubernetes で実行されているサンプル アプリケーションのアプリケーション アーキテクチャを示しています。 アプリは、次のようないくつかのコンポーネントで構成されています。

 1) Azure Stack Hub 上の AKS エンジン ベースの Kubernetes クラスター。
 2) [cert-manager](https://www.jetstack.io/cert-manager/)。これは、Kubernetes 内で証明書を管理するための一連のツールを提供し、Let's Encrypt に証明書を自動的に要求するために使用されます。
 3) フロントエンド (ratings-web)、api (ratings-api)、データベース (ratings-mongodb) 用のアプリケーション コンポーネントを含む Kubernetes 名前空間。
 4) Kubernetes クラスター内のエンドポイントに HTTP/HTTPS トラフィックをルーティングするイングレス コントローラー。

サンプル アプリケーションは、アプリケーション アーキテクチャを示すために使用されます。 すべてのコンポーネントは例です。 アーキテクチャには、アプリケーションのデプロイが 1 つだけ含まれています。 高可用性 (HA) を実現するために、2 つの異なる Azure Stack Hub インスタンス上でデプロイを少なくとも 2 回実行します。これらは、同じ場所または 2 つ (またはそれ以上) の異なるサイト内で実行できます。

![インフラストラクチャ アーキテクチャ](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Azure Container Registry、Azure Monitor などのサービスは、Azure 内またはオンプレミスの Azure Stack Hub の外部でホストされます。 このハイブリッド設計では、単一の Azure Stack Hub インスタンスの停止からソリューションを保護します。

## <a name="components"></a>コンポーネント

全体的なアーキテクチャは、次のコンポーネントで構成されます。

**Azure Stack Hub** は Azure の拡張機能であり、データセンター内で Azure サービスを提供することによりオンプレミス環境でワークロードを実行できます。 詳細については、「[Azure Stack Hub の概要](/azure-stack/operator/azure-stack-overview)」をご覧ください。

**Azure Kubernetes Service エンジン (AKS エンジン)** は、現在 Azure で利用可能なマネージド Kubernetes サービス オファリング (Azure Kubernetes Service (AKS)) の背後にあるエンジンです。 Azure Stack Hub の場合、AKS エンジンにより、Azure Stack Hub の IaaS 機能を使用して、完全に機能するセルフマネージド Kubernetes クラスターをデプロイ、スケール、アップグレードすることができます。 詳細については、[AKS エンジンの概要](https://github.com/Azure/aks-engine)に関するページをご覧ください。

Azure 上の AKS エンジンと Azure Stack Hub 上の AKS エンジンの違いについては、「[既知の問題と制限](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations)」を参照してください。

**Azure Virtual Network (VNet)** は、Kubernetes クラスター インフラストラクチャをホストする仮想マシン (VM) の各 Azure Stack Hub 上でネットワーク インフラストラクチャを提供するために使用されます。

**Azure Load Balancer** は、Kubernetes API エンドポイントと Nginx イングレス コントローラーに使用されます。 ロード バランサーは、特定のサービスを提供しているノードと VM に外部 (インターネットなど) トラフィックをルーティングします。

**Azure Container Registry (ACR)** は、クラスターにデプロイされたプライベート Docker イメージと Helm チャートを格納するために使用されます。 AKS エンジンは、Azure AD ID を使用してコンテナー レジストリに対して認証できます。 Kubernetes には ACR は必要ありません。 Docker Hub などの他のコンテナー レジストリを使用できます。

**Azure Repos** は、コードの管理に使用できるバージョン管理ツールのセットです。 GitHub またはその他の git ベースのリポジトリを使用することもできます。 詳細については、[Azure Repos の概要](/azure/devops/repos/get-started/what-is-repos)に関する記事をご覧ください。

**Azure Pipelines** は Azure DevOps Services の一部であり、自動化されたビルド、テスト、およびデプロイを実行します。 Jenkins などのサードパーティ CI/CD ソリューションも使用できます。 詳細については、[Azure Pipeline の概要](/azure/devops/pipelines/get-started/what-is-azure-pipelines)に関する記事をご覧ください。

**Azure Monitor** では、ソリューション内の Azure サービスのプラットフォーム メトリックやアプリケーション テレメトリなど、メトリックとログが収集されて格納されます。 このデータは、アプリケーションを監視したり、アラートやダッシュボードを設定したり、障害の根本原因分析を実行したりするために使用します。 Azure Monitor は、コントローラー、ノード、およびコンテナーからのメトリックや、コンテナー ログおよびマスター ノード ログを収集するために Kubernetes と統合します。 詳細については、「[Azure Monitor の概要](/azure/azure-monitor/overview)」をご覧ください。

**Azure Traffic Manager** は、異なる Azure リージョンまたは Azure Stack Hub デプロイにまたがってトラフィックをサービスに最適に配分できるようにする DNS ベースのトラフィック ロード バランサーです。 Traffic Manager により、高可用性と応答性も提供されます。 アプリケーション エンドポイントは、外部からアクセスできる必要があります。 他のオンプレミス ソリューションも利用できます。

**Kubernetes イングレス コントローラー** により、Kubernetes クラスター内のサービスへの HTTP(S) ルートが公開されます。 この目的には、Nginx または任意の適切なイングレス コントローラーを使用できます。

**Helm** は、Kubernetes デプロイ用のパッケージ マネージャーであり、デプロイ、サービス、シークレットなどのさまざまな Kubernetes オブジェクトを 1 つの "チャート" にバンドルする手段を提供します。 チャート オブジェクトの発行、デプロイ、制御バージョン管理、および更新を行うことができます。 Azure Container Registry は、パッケージ化された Helm チャートを格納するためのリポジトリとして使用できます。

## <a name="design-considerations"></a>設計上の考慮事項

このパターンは、この記事の次以降のセクションで詳しく説明されているいくつかの高レベルの考慮事項に従います。

- アプリケーションでは、ベンダー ロックインを避けるために、Kubernetes ネイティブ ソリューションを使用します。
- アプリケーションではマイクロサービス アーキテクチャが使用されます。
- Azure Stack Hub では受信は必要ありませんが、送信インターネット接続が許可されます。

これらの推奨プラクティスは、実際のワークロードとシナリオにも適用されます。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

スケーラビリティは、一貫性があり信頼性とパフォーマンスの高いアプリケーションへのアクセスをユーザーに提供するために重要です。

サンプル シナリオでは、複数レイヤーのアプリケーション スタックのスケーラビリティについて説明します。 さまざまなレイヤーの概要を次に示します。

| アーキテクチャ レベル | 影響 | 方法はありますか。 |
| --- | --- | ---
| Application | Application | ポッド、レプリカ、コンテナー インスタンスの数に基づく水平スケーリング* |
| クラスター | Kubernetes クラスター | ノード数 (1 から 50)、VM-SKU サイズ、およびノード プール (Azure Stack Hub 上の AKS エンジンは、現在 1 つのノード プールのみをサポート)。AKS エンジンの scale コマンドを使用 (手動) |
| インフラストラクチャ | Azure Stack Hub | Azure Stack Hub デプロイ内のノード数、容量、およびスケール ユニット数 |

\* Kubernetes の ポッドの水平オートスケーラー (HPA) を使用。コンテナー インスタンス (CPU やメモリ) のサイズ変更による、自動化されたメトリックベースのスケーリングまたは垂直スケーリング。

**Azure Stack Hub (インフラストラクチャ レベル)**

Azure Stack Hub はデータセンター内の物理ハードウェア上で実行されるため、Azure Stack Hub インフラストラクチャはこの実装の基盤です。 ハブ ハードウェアを選択するときは、CPU、メモリ密度、ストレージ構成、およびサーバー数を選択する必要があります。 Azure Stack Hub のスケーラビリティの詳細については、次のリソースをご確認ください。

- [Azure Stack Hub のためのキャパシティ プランニングの概要](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [Azure Stack Hub のスケール ユニット ノードを追加する](/azure-stack/operator/azure-stack-add-scale-node)

**Kubernetes クラスター (クラスター レベル)**

Kubernetes クラスター自体は、コンピューティング、ストレージ、ネットワーク リソースなどの Azure (Stack) IaaS コンポーネントで構成され、それらの上に構築されています。 Kubernetes ソリューションには、Azure (および Azure Stack Hub) 内に VM としてデプロイされるマスターおよびワーカー ノードが含まれます。

- [コントロール プレーン](/azure/aks/concepts-clusters-workloads#control-plane) (マスター) により、主要な Kubernetes サービスと、アプリケーション ワークロードのオーケストレーションが提供されます。
- [ワーカー ノード](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (ワーカー) により、アプリケーション ワークロードが実行されます。

初期デプロイの VM サイズを選択する際には、いくつかの考慮事項があります。  

- **コスト** - ワーカー ノードを計画するときは、発生する VM ごとのコスト全体を念頭に置いてください。 たとえば、アプリケーションのワークロードに限られたリソースが必要な場合は、サイズの小さい VM をデプロイすることを計画する必要があります。 Azure のような Azure Stack Hub は通常、消費量に基づいて課金されるため、Kubernetes ロール用に VM を適切にサイズ設定することは、消費コストを最適化するために重要です。 

- **スケーラビリティ** - マスターおよびワーカー ノードの数をスケールインおよびアウトしたり、追加のノード プールを追加したりすることによって (現在、Azure Stack Hub 上では利用できません)、クラスターのスケーラビリティが実現されます。 クラスターのスケーリングは、Container Insights (Azure Monitor + Log Analytics) を使用して収集されたパフォーマンス データに基づいて行うことができます。 

    アプリケーションに必要なリソースが増減する場合は、現在のノードを水平方向 (1 から 50 ノード) にスケールアウト (またはイン) できます。 50 を超えるノードが必要な場合は、別のサブスクリプション内で追加のクラスターを作成できます。 クラスターを再デプロイしないと、実際の VM を別の VM サイズに垂直方向にスケールアップすることはできません。

    スケーリングは、最初に Kubernetes クラスターをデプロイするために使用された AKS エンジン ヘルパー VM を使用して手動で行います。 詳細については、「[Kubernetes クラスターのスケーリング](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)」を参照してください

- **クォータ** - Azure Stack Hub 上で AKS のデプロイを計画するときに構成した [クォータ](/azure-stack/operator/azure-stack-quota-types)について検討します。 各[サブスクリプション](/azure-stack/operator/service-plan-offer-subscription-overview)に適切なプランとクォータが構成されていることを確認します。 サブスクリプションは、スケールアウトの際にクラスターに必要なコンピューティング、ストレージ、およびその他のサービスの量に対応できる必要があります。

- **アプリケーションのワークロード** - Azure Kubernetes Service ドキュメントの Kubernetes コア概念に関する記事で [クラスターとワークロードの概念](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools)に関するページをご覧ください。 この記事は、アプリケーションのコンピューティングとメモリのニーズに基づいて適切な VM サイズを調べるのに役立ちます。  

**アプリケーション (アプリケーション レベル)**

アプリケーション レイヤーでは、Kubernetes の[ポッドの水平オートスケーラー (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) を使用します。 HPA を使用して、さまざまなメトリック (CPU 使用率など) に基づいて、デプロイ内のレプリカ (ポッドとコンテナー インスタンス) の数を増減できます。

別の方法として、コンテナー インスタンスを垂直方向にスケーリングすることもできます。 これは、特定のデプロイで要求され、使用できる CPU とメモリの量を変更することによって実現できます。 詳細については、kubernetes.io で「[コンテナーのリソース管理](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)」を参照してください。

## <a name="networking-and-connectivity-considerations"></a>ネットワークと接続性に関する考慮事項

ネットワークと接続は、Azure Stack Hub 上の Kubernetes について前述した 3 つのレイヤーにも影響します。 次の表は、レイヤーとそれらに含まれるサービスを示しています。

| レイヤー | 影響 | 何ですか? |
| --- | --- | ---
| Application | Application | アプリケーションにアクセスする方法。 インターネットに公開されるかどうか。 |
| クラスター | Kubernetes クラスター | Kubernetes API、AKS エンジン VM、コンテナー イメージのプル (エグレス)、監視データとテレメトリの送信 (エグレス) |
| インフラストラクチャ | Azure Stack Hub | ポータルや Azure Resource Manager エンドポイントなどの Azure Stack Hub 管理エンドポイントのアクセシビリティ。 |

**アプリケーション**。

アプリケーション レイヤーの場合、最も重要な考慮事項は、アプリケーションが公開され、インターネットからアクセス可能であるかどうかです。 Kubernetes の観点から見ると、インターネット アクセシビリティとは、Kubernetes サービスまたはイングレス コントローラーを使用してデプロイまたはポッドを公開することを意味します。

> [!NOTE]
> Azure Stack Hub 上のフロントエンド パブリック IP の数は 5 つに制限されているため、イングレス コントローラーを使用して Kubernetes サービスを公開することをお勧めします。 この設計では、(LoadBalancer タイプの) Kubernetes サービスの数も 5 つに制限されます。これは、多くのデプロイに対して小さすぎます。 詳細については、[AKS エンジンのドキュメント](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips)をご覧ください。

ロード バランサーまたはイングレス コントローラーを介してパブリック IP を使用してアプリケーションを公開することは、アプリケーションがインターネット経由でアクセスできるようになったことを意味するものではありません。 Azure Stack Hub は、ローカル イントラネット上でのみ見えるパブリック IP アドレスを持つことができます。すべてのパブリック IP アドレスが実際にインターネットに接続されているわけではありません。

前のブロックでは、アプリケーションへのイングレス トラフィックを考慮しています。 Kubernetes のデプロイを成功させるために考慮する必要があるもう 1 つのトピックは、送信およびエグレス トラフィックです。 エグレス トラフィックを必要とするいくつかのユース ケースを次に示します。

- DockerHub または Azure Container Registry に格納されているコンテナー イメージのプル
- Helm チャートの取得
- Application Insights データ (またはその他の監視データ) の出力

一部のエンタープライズ環境では "_透過的_" または "_非透過的_" プロキシ サーバーの使用が必要になる場合があります。 これらのサーバーには、クラスターのさまざまなコンポーネントに対する特定の構成が必要です。 AKS エンジンのドキュメントには、ネットワーク プロキシに対応する方法についてのさまざまな詳細が記載されています。 詳細については、「[AKS エンジンとプロキシ サーバー](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)」を参照してください

最後に、クラスター間のトラフィックは、Azure Stack Hub インスタンス間を流れる必要があります。 サンプルのデプロイは、個々の Azure Stack Hub インスタンス上で実行されている個々の Kubernetes クラスターで構成されます。 2 つのデータベース間のレプリケーション トラフィックなど、これらの間のトラフィックは "外部トラフィック" です。 外部トラフィックは、2 つの Azure Stack Hub インスタンス上で Kubernetes を接続するために、サイト間 VPN または Azure Stack Hub のパブリック IP アドレスを通じてルーティングする必要があります。

![クラスター間または内トラフィック](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**クラスター**  

Kubernetes クラスターには、必ずしもインターネット経由でアクセスできる必要はありません。 関連する部分は、たとえば `kubectl` を使用したクラスターの操作に使用される Kubernetes API です。 Kubernetes API エンドポイントは、クラスターを操作するすべてのユーザーからアクセスできるようにするか、アプリケーションとサービスをその上にデプロイする必要があります。 このトピックの詳細については、下の「[デプロイ (CI/CD) に関する考慮事項](#deployment-cicd-considerations)」で DevOps の観点から見た詳細をご覧ください。

クラスター レベルでは、エグレス トラフィックに関する考慮事項もいくつかあります。

- ノードの更新 (Ubuntu の場合)
- (Azure LogAnalytics に送信された) データの監視
- 送信トラフィックを必要とする他のエージェント (各デプロイ担当者の環境に固有)

AKS エンジンを使用して Kubernetes クラスターをデプロイする前に、最終的なネットワーク設計を計画してください。 専用の仮想ネットワークを作成するのではなく、既存のネットワークにクラスターをデプロイする方が効率的な場合があります。 たとえば、Azure Stack Hub 環境で既に構成されている既存のサイト間 VPN 接続を利用できます。

**インフラストラクチャ**  

インフラストラクチャとは、Azure Stack Hub 管理エンドポイントへのアクセスを指します。 エンドポイントには、テナントと管理ポータル、および Azure Resource Manager 管理者とテナントのエンドポイントが含まれます。 これらのエンドポイントは、Azure Stack Hub とそのコア サービスを操作するために必要です。

## <a name="data-and-storage-considerations"></a>データとストレージに関する考慮事項

2 つの Azure Stack Hub インスタンスにまたがって、2 つの個別の Kubernetes クラスターにアプリケーションの 2 つのインスタンスがデプロイされます。 この設計の場合、データをレプリケートしてそれらの間で同期する方法を検討する必要があります。

Azure では、クラウド内の複数のリージョンとゾーンにわたってストレージをレプリケートする機能が組み込まれています。 現在 Azure Stack Hub には、2つの異なる Azure Stack Hub インスタンスにストレージをレプリケートするネイティブな方法はありません。2 つの独立したクラウドが形成され、それらをセットとして管理するための包括的な方法はありません。 Azure Stack Hub で実行されているアプリケーションの回復性を計画すると、アプリケーションの設計とデプロイにおいて、このような独立性を考慮せざるを得なくなります。

ほとんどの場合、AKS にデプロイされた回復力のある高可用性アプリケーションにはストレージのレプリケーションは必要ありません。 ただし、アプリケーションの設計では、Azure Stack Hub インスタンスごとに個別のストレージを考慮する必要があります。 この設計が Azure Stack Hub へのソリューションのデプロイに対する懸念事項または障害である場合、ストレージの添付を提供する Microsoft パートナーのソリューションがあります。 ストレージの添付により、複数の Azure Stack Hub と Azure 間にストレージ レプリケーション ソリューションが提供されます。 詳細については、「[パートナー ソリューション](#partner-solutions)」を参照してください。

このアーキテクチャでは、以下のレイヤーを検討しました。

**Configuration**

構成には、Azure Stack Hub、AKS エンジン、および Kubernetes クラスター自体の構成が含まれます。 構成は、可能な限り自動化し、Azure DevOps や GitHub などの Git ベースのバージョン コントロール システムにコードとしてのインフラストラクチャとして格納する必要があります。 これらの設定を複数のデプロイ間で簡単に同期することはできません。 そのため、構成を外部から保存および適用し、DevOps パイプラインを使用することをお勧めします。

**アプリケーション**。

アプリケーションは、Git ベースのリポジトリに格納する必要があります。 新しいデプロイがある場合、アプリケーションに変更を加えた場合、またはディザスター リカバリーを行う場合は、Azure Pipelines を使用して簡単にデプロイできます。

**データ**

データは、ほとんどのアプリケーション設計で最も重要な考慮事項です。 アプリケーション データは、アプリケーションの異なるインスタンス間で同期されている必要があります。 また、障害が発生した場合に備えて、データのバックアップとディザスター リカバリーの戦略も必要です。

この設計の実現は、テクノロジの選択に大きく依存します。 Azure Stack Hub 上で可用性の高い方法でデータベースを実装するためのソリューション例を次に示します。

- [Azure と Azure Stack Hub に SQL Server 2016 可用性グループをデプロイする](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [高可用性の MongoDB ソリューションを Azure と Azure Stack Hub にデプロイする](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

複数の場所でデータを操作する場合の考慮事項は、可用性と回復性が高いソリューションではさらに複雑な考慮事項です。 次の点を考慮してください。

- Azure Stack Hub 間の待機時間とネットワーク接続。
- サービスとアクセス許可のための ID の可用性。 各 Azure Stack Hub インスタンスは、外部ディレクトリと統合されます。 デプロイ時に、Azure Active Directory (Azure AD) または Active Directory フェデレーション サービス (ADFS) のどちらを使用するかを選択します。 そのため、複数の独立した Azure Stack Hub インスタンスと対話できる 1 つの ID を使用する可能性があります。

## <a name="business-continuity-and-disaster-recovery"></a>事業継続とディザスター リカバリー

事業継続とディザスター リカバリー (BCDR) は、Azure Stack Hub と Azure の両方で重要なトピックです。 主な違いは、Azure Stack Hub ではオペレーターが BCDR プロセス全体を管理する必要があることです。 Azure では、BCDR の一部が Microsoft によって自動的に管理されます。

BCDR は、前のセクション「[データとストレージに関する考慮事項](#data-and-storage-considerations)」で説明したのと同じ領域に影響を与えます。

- インフラストラクチャと構成
- アプリケーションの可用性
- アプリケーション データ

前のセクションで説明したように、これらの領域は Azure Stack Hub オペレーターの担当であり、組織によって異なる場合があります。 使用可能なツールとプロセスに従って BCDR を計画します。

**インフラストラクチャと構成**

このセクションでは、Azure Stack Hub の物理的および論理的なインフラストラクチャと構成について説明します。 管理およびテナント スペースでの操作について説明します。

Azure Stack Hub オペレーター (または管理者) は、Azure Stack Hub インスタンスのメンテナンスを担当します。 ネットワーク、ストレージ、ID などのコンポーネントと、この記事の範囲外のその他のトピックが含まれます。 Azure Stack Hub の操作の詳細については、次のリソースをご覧ください。

- [Infrastructure Backup サービスを使用した Azure Stack Hub のデータの回復](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [管理者ポータルで Azure Stack Hub のバックアップを有効にする](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [致命的なデータ損失からの復旧](/azure-stack/operator/azure-stack-backup-recover-data)
- [インフラストラクチャ バックアップ サービスのベスト プラクティス](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack Hub は、Kubernetes アプリケーションがデプロイされるプラットフォームでありファブリックです。 Kubernetes アプリケーションのアプリケーション所有者は Azure Stack Hub のユーザーになり、ソリューションに必要なアプリケーション インフラストラクチャをデプロイするためのアクセス権が付与されます。 この場合のアプリケーション インフラストラクチャは、AKS エンジンを使用してデプロイされた Kubernetes クラスターと、その周辺のサービスを意味します。 これらのコンポーネントは Azure Stack Hub にデプロイされ、Azure Stack Hub オファーによって制約されます。 Kubernetes アプリケーション所有者が受け入れるオファーに、ソリューション全体をデプロイするのに十分な容量 (Azure Stack Hub クォータで表される) があることを確認します。 前のセクションで推奨されているように、アプリケーションのデプロイは、コードとしてのインフラストラクチャと、Azure DevOps Azure Pipelines のようなデプロイ パイプラインを使用して自動化する必要があります。

Azure Stack Hub の詳細については、「[Azure Stack Hub サービス、プラン、オファー、サブスクリプションの概要](/azure-stack/operator/service-plan-offer-subscription-overview)」を参照してください

AKS エンジンの構成は、その出力を含めて安全に保存し、格納することが重要です。 これらのファイルには、Kubernetes クラスターへのアクセスに使用される機密情報が含まれているため、管理者以外に公開されないように保護する必要があります。

**アプリケーションの可用性**

デプロイされたインスタンスのバックアップにアプリケーションが依存しないようにしてください。 標準的な方法としては、コードとしてのインフラストラクチャ パターンに従ってアプリケーションを完全に再デプロイします。 たとえば、Azure DevOps Azure Pipelines を使用して再デプロイします。 BCDR プロシージャでは、同じまたは別の Kubernetes クラスターにアプリケーションを再デプロイする必要があります。

**アプリケーション データ**

アプリケーション データは、データの損失を最小限に抑えるための重要な部分です。 前のセクションでは、アプリケーションの 2 つ (またはそれ以上) のインスタンス間でデータをレプリケートおよび同期する手法について説明しました。 データの格納に使用されるデータベース インフラストラクチャ (MySQL、MongoDB、MSSQL など) によっては、さまざまなデータベース可用性とバックアップの手法から選択できます。

整合性を確保するには、次のいずれかの方法を使用することをお勧めします。
- 特定のデータベースのネイティブ バックアップ ソリューション。
- アプリケーションで使用されるデータベースの種類のバックアップと回復を正式にサポートするバックアップ ソリューション。

> [!IMPORTANT]
> アプリケーション データが存在するのと同じ Azure Stack Hub インスタンスにバックアップ データを格納しないでください。 Azure Stack Hub インスタンスが完全に停止すると、バックアップも危険にさらされます。

## <a name="availability-considerations"></a>可用性に関する考慮事項

AKS エンジンを介してデプロイされた Azure Stack Hub 上の Kubernetes は、マネージド サービスではありません。 これは、Azure サービスとしてのインフラストラクチャ (IaaS) を使用した Kubernetes クラスターの自動化されたデプロイと構成です。 そのため、基になるインフラストラクチャと同じ可用性を提供します。

Azure Stack Hub インフラストラクチャは、既に障害に対する回復性があり、複数の[障害および更新ドメイン](/azure-stack/user/azure-stack-vm-considerations#high-availability)にコンポーネントを分散する可用性セットのような機能を提供します。 しかし、基盤となっているテクノロジ (フェールオーバー クラスタリング) では、ハードウェアが故障した場合にその影響を受ける物理サーバー上の VM に多少のダウンタイムが発生します。

運用環境の Kubernetes クラスターに加えて、ワークロードを 2 つ (またはそれ以上) のクラスターにデプロイすることをお勧めします。 これらのクラスターは別の場所またはデータセンターにホストする必要があり、Azure Traffic Manager などのテクノロジを使用して、クラスターの応答時間に基づくか地理に基づいてユーザーをルーティングします。

![Traffic Manager を使用したトラフィック フローの制御](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

1 つだけの Kubernetes クラスターを持つお客様は、通常、特定のアプリケーションのサービス IP または DNS 名に接続します。 複数クラスターのデプロイでは、各 Kubernetes クラスター上のサービスとイングレスを指す、Traffic Manager DNS 名に接続するようにしましょう。

![Traffic Manager を使用したオンプレミス クラスターへのルーティング](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> このパターンは、[Azure 内の (マネージド) AKS クラスター向けのベスト プラクティス](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment)でもあります。

AKS エンジンを介してデプロイされる Kubernetes クラスター自体は、少なくとも 3 つのマスター ノードと 2 つのワーカー ノードで構成されている必要があります。

## <a name="identity-and-security-considerations"></a>ID とセキュリティに関する考慮事項

ID とセキュリティは重要なトピックです。 ソリューションが独立した Azure Stack Hub インスタンスにまたがっている場合は特にそうです。 Kubernetes と Azure (Azure Stack Hub を含む) のどちらにも、ロールベースのアクセス制御 (RBAC) について固有のメカニズムがあります。

- Azure RBAC により、Azure (および Azure Stack Hub) でのリソースへのアクセス (新しい Azure リソースを作成する機能を含む) が制御されます。 アクセス許可は、ユーザー、グループ、またはサービス プリンシパルに割り当てることができます。 (サービス プリンシパルは、アプリケーションによって使用されるセキュリティ ID です。)
- Kubernetes RBAC は、Kubernetes API へのアクセス許可を制御します。 たとえば、ポッドの作成やポッドの一覧表示は、RBAC によってユーザーに許可 (または拒否) できるアクションです。 ユーザーに Kubernetes アクセス許可を割り当てるには、ロールとロール バインディングを作成します。

**Azure Stack Hub ID と RBAC**

Azure Stack Hub では、ID プロバイダーに 2 つの選択肢が用意されています。 使用するプロバイダーは、環境と、接続されている、または切断されている環境のどちらで実行するかによって決まります。

- Azure AD - 接続された環境でのみ使用できます。
- 従来の Active Directory フォレストへの ADFS - 接続されている、または切断されている環境のどちらでも使用できます。

ID プロバイダーにより、ユーザーとグループが管理されます。これには、リソースにアクセスするための認証と認可が含まれます。 サブスクリプション、リソース グループ、VM やロード バランサーなどの個々のリソースのような Azure Stack Hub リソースへのアクセス権を付与することができます。 一貫性のあるアクセス モデルを使用するには、すべての Azure Stack Hub に同じグループ (直接または入れ子) を使用することを検討する必要があります。 次に構成例を示します。

![azure stack hub を使用した入れ子になった aad グループ](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

この例には、特定の目的を持つ専用のグループ (AAD または ADFS を使用) が含まれています。 たとえば、特定の Azure Stack Hub インスタンス (ここでは、"Seattle K8s Cluster Contributor") に Kubernetes クラスター インフラストラクチャを含むリソース グループに対する共同作成者のアクセス許可を付与します。 これらのグループは、各 Azure Stack Hub の "サブグループ" を含む全体グループに入れ子になっています。

サンプル ユーザーには、Kubernetes インフラストラクチャ リソースのセット全体を含む両方のリソース グループに対する "共同作成者" アクセス許可が付与されました。 インスタンスは同じ ID プロバイダーを共有するので、ユーザーは両方の Azure Stack Hub インスタンス上のリソースにアクセスできます。

> [!IMPORTANT]
> これらのアクセス許可は、Azure Stack Hub と、その上にデプロイされた一部のリソースにのみ影響します。 このレベルのアクセス権を持つユーザーは、多くの問題を引き起こすことがありますが、Kubernetes デプロイに追加でアクセスすることなく Kubernetes IaaS VM や Kubernetes API にアクセスすることはできません。

**Kubernetes ID と RBAC**

既定では、Kubernetes クラスターは、基盤となる Azure Stack Hub と同じ ID プロバイダーを使用しません。 Kubernetes クラスター、マスター、およびワーカー ノードをホストしている VM は、クラスターのデプロイ時に指定された SSH キーを使用します。 この SSH キーは、SSH を使用してこれらのノードに接続するために必要です。

Kubernetes API (たとえば、`kubectl` を使用してアクセスされる) は、既定の "cluster admin" サービス アカウントを含むサービス アカウントによっても保護されます。 このサービス アカウントの資格情報は、最初は Kubernetes マスター ノード上の `.kube/config` ファイルに格納されています。

**シークレットの管理とアプリケーションの資格情報**

接続文字列やデータベース資格情報などのシークレットを格納するには、次のようないくつかの選択肢があります。

- Azure Key Vault
- Kubernetes シークレット
- HashiCorp Vault のようなサードパーティ ソリューション (Kubernetes 上で実行)

構成ファイル、アプリケーション コード、またはスクリプト内にシークレットや資格情報をプレーンテキストで保存しないでください。 また、バージョン コントロール システムには格納しないでください。 代わりに、必要に応じてデプロイの自動化によってシークレットを取得する必要があります。

## <a name="patch-and-update"></a>パッチと更新プログラム

Azure Kubernetes Service の **パッチと更新プログラム (PNU)** プロセスは、部分的に自動化されています。 Kubernetes バージョンのアップグレードは手動でトリガーされますが、セキュリティ更新プログラムは自動的に適用されます。 これらの更新プログラムには、OS のセキュリティ修正プログラムやカーネルの更新プログラムが含まれている場合があります。 AKS は、更新プロセスを完了するためにこれらの Linux ノードを自動的に再起動しません。 

Azure Stack Hub 上で AKS エンジンを使用してデプロイされた Kubernetes クラスターの場合、PNU プロセスは管理されません。これは、クラスター オペレーターが担当します。 

AKS エンジンは、2 つの最も重要なタスクに役立ちます。

- [新しい Kubernetes とベース OS イメージ バージョンへのアップグレード](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [ベース OS イメージのみのアップグレード](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

新しいベース OS イメージには、最新の OS セキュリティ修正プログラムとカーネル更新プログラムが含まれています。 

[無人アップグレード](https://wiki.debian.org/UnattendedUpgrades) メカニズムでは、新しいベース OS イメージ バージョンを Azure Stack Hub Marketplace で利用できるようになる前にリリースされたセキュリティ更新プログラムが自動的にインストールされます。 無人アップグレードは既定で有効になっており、セキュリティ更新プログラムは自動的にインストールされますが、Kubernetes クラスター ノードは再起動されません。 ノードの再起動は、オープンソースの [**K** Ubernetes **RE** boot **D** aemon (kured))](/azure/aks/node-updates-kured) を使用して自動化できます。 Kured によって、再起動を必要とする Linux ノードが監視され、ポッドの実行とノードの再起動プロセスの再スケジュールが自動的に処理されます。

## <a name="deployment-cicd-considerations"></a>デプロイ (CI/CD) に関する考慮事項

Azure と Azure Stack Hub では同じ Azure Resource Manager REST API が公開されます。 これらの API は、他の Azure クラウド (Azure、Azure China 21Vianet、Azure Government) と同様に対処されます。 クラウド間の API バージョンに違いがある可能性があり、Azure Stack Hub ではサービスのサブセットのみが提供されます。 また、管理エンドポイントの URI は、クラウドごと、および Azure Stack Hub のインスタンスごとに異なります。

ここで説明した微妙な違いを除けば、Azure Resource Manager REST API によって、Azure と Azure Stack Hub の両方と対話するための一貫した方法が提供されます。 ここでは、他の Azure クラウドで使用したものと同じツール セットを使用できます。 Azure DevOps、Jenkins などのツール、または PowerShell を使用して、サービスを Azure Stack Hub にデプロイし、オーケストレーションを行うことができます。

**考慮事項**

Azure Stack Hub のデプロイに関する大きな違いの 1 つは、インターネット アクセシビリティの問題です。 インターネット アクセシビリティによって、CI/CD ジョブに対して、Microsoft ホステッドとセルフホステッド のどちらのビルド エージェントを選択するかが決まります。

セルフホステッド エージェントは、Azure Stack Hub の上で (IaaS VM として)、または Azure Stack Hub にアクセスできるネットワーク サブネット内で実行できます。 相違点の詳細については、「[Azure Pipelines エージェント](/azure/devops/pipelines/agents/agents)」をご覧ください。

次の図は、セルフホステッドまたは Microsoft ホステッドのどちらのビルド エージェントが必要かを判断するのに役立ちます。

![セルフホステッド ビルド エージェント。はいまたはいいえ](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Azure Stack Hub 管理エンドポイントは、インターネット経由でアクセスできますか。
  - はい:Microsoft ホステッド エージェントと共に Azure Pipelines を使用して、Azure Stack Hub に接続できます。
  - いいえ:Azure Stack Hub の管理エンドポイントに接続できるセルフホステッド エージェントが必要です。
- Kubernetes クラスターにはインターネット経由でアクセスできますか。
  - はい:Microsoft ホステッド エージェントと共に Azure Pipelines を使用して、Kubernetes API エンドポイントと直接対話できます。
  - いいえ:Kubernetes クラスター API エンドポイントに接続できるセルフホステッド エージェントが必要です。

インターネット経由で Azure Stack Hub 管理エンドポイントと Kubernetes API にアクセスできるシナリオでは、デプロイに Microsoft ホステッド エージェントを使用できます。 このデプロイを実行すると、アプリケーション アーキテクチャは次のようになります。

[![パブリック アーキテクチャの概要](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Azure Resource Manager エンドポイント、Kubernetes API、またはその両方がインターネット経由で直接アクセスできない場合は、セルフホステッド ビルド エージェントを利用してパイプラインのステップを実行できます。 この設計では、必要な接続が少なく、Azure Resource Manager エンドポイントと Kubernetes API へのオンプレミス ネットワーク接続のみを使用してデプロイできます。

[![オンプレミス アーキテクチャの概要](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **切断されたシナリオについて** Azure Stack Hub、Kubernetes、またはその両方がインターネットに接続された管理エンドポイントを持たないシナリオでも、デプロイに Azure DevOps を使用できます。 セルフホステッド エージェント プール (オンプレミスまたは Azure Stack Hub 自体で実行されている DevOps エージェント)、またはオンプレミスの完全にセルフホステッドの Azure DevOps Server を使用できます。 セルフホステッド エージェントは、送信 HTTPS (TCP/443) インターネット接続のみを必要とします。

このパターンでは、各 Azure Stack Hub インスタンス上で Kubernetes クラスター (AKS エンジンを使用してデプロイおよび調整) を使用できます。 これには、フロントエンド、中間層、バックエンド サービス (MongoDB など)、および nginx ベースのイングレス コントローラーで構成されるアプリケーションが含まれます。 K8s クラスターにホストされているデータベースを使用する代わりに、"外部データ ストア" を利用できます。 データベース オプションには、MySQL、SQL Server、または Azure Stack Hub の外部か IaaS 内にホストされている任意の種類のデータベースが含まれます。 このような構成は、ここでは扱いません。

## <a name="partner-solutions"></a>パートナー ソリューション

Azure Stack Hub の機能を拡張できる Microsoft パートナー ソリューションがあります。 これらのソリューションは、Kubernetes クラスター上で実行されているアプリケーションのデプロイに役立つことがわかっています。  

## <a name="storage-and-data-solutions"></a>ストレージとデータのソリューション

「[データとストレージに関する考慮事項](#data-and-storage-considerations)」で説明したように、現在 Azure Stack Hub には、複数のインスタンスにストレージをレプリケートするためのネイティブ ソリューションがありません。 Azure とは異なり、複数のリージョンにわたってストレージをレプリケートする機能は存在しません。 Azure Stack Hub では、各インスタンスは独自の個別クラウドです。 ただし、Azure Stack Hub と Azure 間でのストレージ レプリケーションを可能にする Microsoft パートナーのソリューションを利用できます。 

**SCALITY**

[Scality](https://www.scality.com/) では、2009 年以降、デジタル ビジネスを強化した Web スケール ストレージを提供しています。 Microsoft のソフトウェアで定義されたストレージである Scality RING は、商用 x86 サーバーを、ペタバイト規模のあらゆる種類のデータ (ファイルとオブジェクト) 用の無制限のストレージ プールに変換します。

**CLOUDIAN**

[Cloudian](https://www.cloudian.com/) では、大規模なデータ セットを 1 つの簡単に管理できる環境に統合する無制限の拡張性を備えたストレージを使用して、エンタープライズ ストレージが簡素化されます。

## <a name="next-steps"></a>次のステップ

この記事で紹介した概念の関連情報:

- Azure Stack Hub 内での[クラウド間スケーリング](pattern-cross-cloud-scale.md)および[地理的に分散されたアプリ パターン](pattern-geo-distributed.md)。
- [Azure Kubernetes Service (AKS) 上のマイクロサービス アーキテクチャ](/azure/architecture/reference-architectures/microservices/aks)。

ソリューションの例をテストする準備ができたら、[高可用性 Kubernetes クラスター デプロイ ガイド](/azure/architecture/hybrid/deployments/solution-deployment-guide-highly-available-kubernetes)に進んでください。 デプロイ ガイドでは、コンポーネントをデプロイしてテストするための詳細な手順について説明します。