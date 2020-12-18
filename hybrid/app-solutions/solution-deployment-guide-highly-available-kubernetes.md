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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>高可用性 Kubernetes クラスターを Azure Stack Hub でデプロイする

この記事では、複数の物理的な場所にある複数の Azure Stack Hub インスタンスにデプロイされた高可用性 Kubernetes クラスター環境を構築する方法について説明します。

このソリューション デプロイ ガイドでは、以下を実行する方法について説明します。

> [!div class="checklist"]
> - AKS Engine をダウンロードして準備する
> - AKS Engine ヘルパー VM に接続する
> - Kubernetes クラスターのデプロイ
> - Kubernetes クラスターに接続する
> - Azure Pipelines を Kubernetes クラスターに接続する
> - 監視の構成
> - アプリケーションをデプロイする
> - アプリケーションの自動スケーリングを実行する
> - Traffic Manager を構成する
> - Kubernetes のアップグレード
> - Kubernetes をスケーリングする

> [!Tip]  
> ![ハイブリッドの柱](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub は Azure の拡張機能です。 Azure Stack Hub により、オンプレミス環境にクラウド コンピューティングの機敏性とイノベーションがもたらされ、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドが可能になります。  
> 
> [ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。 これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。

## <a name="prerequisites"></a>前提条件

このデプロイ ガイドの使用を開始する前に、必ず次のことを行ってください。

- 「[高可用性 Kubernetes クラスター パターン](pattern-highly-available-kubernetes.md)」の記事を確認する。
- [コンパニオン GitHub リポジトリ](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)の内容を確認する。この記事で参照されている追加アセットが含まれています。
- 少なくとも ["共同作成者" アクセス許可](/azure-stack/user/azure-stack-manage-permissions)を使用して [Azure Stack Hub ユーザー ポータル](/azure-stack/user/azure-stack-use-portal)にアクセスできるアカウントを持っている。

## <a name="download-and-prepare-aks-engine"></a>AKS Engine をダウンロードして準備する

AKS Engine は、Azure Stack Hub Azure Resource Manager エンドポイントに接続できるすべての Windows または Linux ホストから使用できるバイナリです。 このガイドでは、Azure Stack Hub で新しい Linux (または Windows) VM をデプロイする方法について説明します。 それは、後で AKS Engine によって Kubernetes クラスターがデプロイされるときに使用されます。

> [!NOTE]
> 既存の Windows または Linux VM を使用して、AKS Engine を使用する Azure Stack Hub で Kubernetes クラスターをデプロイすることもできます。

AKS Engine のステップバイステップ プロセスと要件については、次のドキュメントを参照してください。

* [Azure Stack Hub で AKS Engine を Linux にインストールする](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (または [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows) を使用する)

AKS Engine は、(アンマネージド) Kubernetes クラスターを (Azure と Azure Stack Hub で) デプロイして運用するためのヘルパー ツールです。

Azure Stack Hub の AKS Engine の詳細と相違点については、次を参照してください。

* [Azure Stack Hub の AKS Engine とは](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [Azure Stack Hub の AKS Engine](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (GitHub)

サンプル環境では、Terraform を使用して、AKS Engine VM のデプロイを自動化します。 [コンパニオン GitHub リポジトリで詳細とコード](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md)を確認できます。

この手順の結果は、AKS Engine ヘルパー VM と関連リソースを含む Azure Stack Hub の新しいリソース グループになります。

![Azure Stack Hub の AKS Engine VM リソース](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> 接続されていないエアギャップされた環境に AKS Engine をデプロイする必要がある場合は、[接続されていない Azure Stack Hub インスタンス](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances)に関する記事で詳細を確認してください。

次の手順では、新しくデプロイされた AKS Engine VM を使用して Kubernetes クラスターをデプロイします。

## <a name="connect-to-the-aks-engine-helper-vm"></a>AKS Engine ヘルパー VM に接続する

最初に、先ほど作成した AKS Engine ヘルパー VM に接続する必要があります。

VM にはパブリック IP アドレスが必要であり、SSH (ポート 22/TCP) を介してアクセスできる必要があります。

![AKS Engine VM の概要ページ](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> Windows 10 の MobaXterm、puTTY、PowerShell などの任意のツールで、SSH を使用して Linux VM に接続できます。

```console
ssh <username>@<ipaddress>
```

接続した後、コマンド `aks-engine` を実行します。 AKS Engine と Kubernetes のバージョンの詳細については、[サポートされている AKS Engine のバージョン](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions)に関する記事を参照してください。

![aks-engine コマンド ラインの例](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Kubernetes クラスターのデプロイ

AKS Engine ヘルパー VM 自体には、まだ Azure Stack Hub で Kubernetes クラスターが作成されていません。 クラスターの作成は、AKS Engine ヘルパー VM で実行される最初のアクションです。

ステップバイステップ プロセスについては、次のドキュメントを参照してください。

* [AKS エンジンを使用して Azure Stack Hub に Kubernetes クラスターをデプロイする](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

`aks-engine deploy` コマンドと前の手順で行った準備の最終的な結果は、最初の Azure Stack Hub インスタンスのテナント空間にデプロイされた完全に機能する Kubernetes クラスターです。 クラスター自体は、VM、ロード バランサー、VNet、ディスクなどの Azure IaaS コンポーネントで構成されます。

![Azure Stack Hub ポータルでのクラスターの IaaS コンポーネント](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Azure ロード バランサー (K8s API エンドポイント)
2) ワーカー ノード (エージェント プール)
3) マスター ノード

これでクラスターが稼働状態になったので、次の手順でそれに接続します。

## <a name="connect-to-the-kubernetes-cluster"></a>Kubernetes クラスターに接続する

先ほど作成した Kubernetes クラスターに、SSH (デプロイの一部として指定された SSH キーを使用) または `kubectl` (推奨) を介して接続できるようになりました。 Windows、Linux、および macOS 用の Kubernetes コマンドライン ツール `kubectl` は、[こちら](https://kubernetes.io/docs/tasks/tools/install-kubectl/)から入手できます。 それは既にクラスターのマスター ノードに事前インストールされて構成されています。

```console
ssh azureuser@<k8s-master-lb-ip>
```

![マスター ノードで kubectl を実行する](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

マスター ノードを管理タスク用のジャンプボックスとして使用することは推奨されません。 `kubectl` 構成は、マスター ノードと AKS Engine VM の `.kube/config` に格納されます。 Kubernetes クラスターに接続される管理マシンに構成をコピーし、そこで `kubectl` コマンドを使用できます。 `.kube/config` ファイルは、後で Azure Pipelines 内にサービスを構成するためにも使用されます。

> [!IMPORTANT]
> このファイルには Kubernetes クラスターの資格情報が含まれているので、常にセキュリティで保護してください。 ファイルにアクセスできる攻撃者は、管理者としてアクセスするための十分な情報を持っています。 最初の `.kube/config` ファイルを使用して実行されるすべてのアクションは、クラスター管理者アカウントを使用して実行されます。

ここで、クラスターの状態を確認するさまざまな `kubectl` のコマンドを試してみることができます。

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
> Kubernetes には、きめ細かいロールの定義とロールのバインドを作成できる独自の _ *ロールベースのアクセス制御 (RBAC)* * モデルがあります。 これは、cluster-admin アクセス許可を渡す代わりに、クラスターへのアクセスを制御するための望ましい方法です。

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Azure Pipelines を Kubernetes クラスターに接続する

Azure Pipelines を新しくデプロイされた Kubernetes クラスターに接続するには、前の手順で説明した kube config (`.kube/config`) ファイルが必要です。

* Kubernetes クラスターのマスター ノードのいずれかに接続します。
* `.kube/config` ファイルの内容をコピーします。
* [Azure DevOps] > [プロジェクトの設定] > [サービス接続] の順に移動して、新しい "Kubernetes" サービス接続を作成します (KubeConfig を認証方法として使用します)。

> [!IMPORTANT]
> Azure Pipelines (またはそのビルド エージェント) が Kubernetes API にアクセスできる必要があります。 Azure Pipelines から Azure Stack Hub Kubernetes クラスターへのインターネット接続がある場合は、自己ホスト型 Azure Pipelines ビルド エージェントをデプロイする必要があります。

Azure Pipelines 用の自己ホスト型エージェントをデプロイする場合は、Azure Stack Hub または必要なすべての管理エンドポイントへのネットワーク接続を備えたコンピューターへのデプロイのいずれかを実行できます。 詳細については、次を参照してください。

* [Windows](/azure/devops/pipelines/agents/v2-windows) または [Linux](/azure/devops/pipelines/agents/v2-linux) 上の [Azure Pipelines エージェント](/azure/devops/pipelines/agents/agents)

パターンの「[デプロイ (CI/CD) に関する考慮事項](pattern-highly-available-kubernetes.md#deployment-cicd-considerations)」セクションに、Microsoft によってホストされるエージェントと自己ホスト型エージェントのどちらを使用するかを理解するのに役立つ意思決定フローが含まれています。

[![自己ホスト型エージェントの意思決定フロー](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

このサンプル ソリューションのトポロジには、各 Azure Stack Hub インスタンスに自己ホスト型ビルド エージェントが含まれています。 エージェントは、Azure Stack Hub 管理エンドポイントと Kubernetes クラスターの API エンドポイントにアクセスできます。

[![送信トラフィックのみ](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

この設計では、アプリケーション ソリューションからの送信接続のみを対象とする一般的な規制要件に対応しています。

## <a name="configure-monitoring"></a>監視の構成

コンテナー用の [Azure Monitor](/azure/azure-monitor/) を使用して、ソリューション内のコンテナーを監視できます。 これは、Azure Stack Hub の AKS Engine によってデプロイされた Kubernetes クラスターに対する Azure Monitor を指します。

クラスターで Azure Monitor を有効にするには、2 つの方法があります。 どちらの方法でも、Azure 上に Log Analytics ワークスペースを設定する必要があります。

* [方法 1](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) では、Helm Chart が使用されます
* [方法 2](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) は、AKS Engine クラスター仕様の一部です

サンプル トポロジでは、プロセスを自動化し、更新プログラムを簡単にインストールできる "方法 1" を使用します。

次の手順では、コンピューター上に Azure LogAnalytics ワークスペース (ID とキー)、`Helm` (バージョン 3)、および `kubectl` が必要です。

Helm は Kubernetes パッケージ マネージャーであり、macOS、Windows、および Linux で実行されるバイナリとして入手できます。 こちらからダウンロードできます: [helm.sh](https://helm.sh/docs/intro/quickstart/)。Helm は、`kubectl` コマンドに使用される Kubernetes 構成ファイルに依存しています。

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

このコマンドにより、Kubernetes クラスターに Azure Monitor エージェントがインストールされます。

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

Kubernetes クラスターの Operations Management Suite (OMS) エージェントにより、(送信 HTTPS を使用して) Azure Log Analytics ワークスペースに監視データが送信されます。 これで、Azure Monitor を使用して、Azure Stack Hub で Kubernetes クラスターに関するより深い分析情報を得ることができます。 この設計は、アプリケーションのクラスターと共に自動的にデプロイできる分析のパワーを示するための強力な方法です。

[![Azure Monitor でのAzure Stack Hub クラスター](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Azure Monitor クラスターの詳細](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Azure Monitor に Azure Stack Hub データが表示されない場合は、「[AzureMonitor-Containers ソリューションを Azure Loganalytics ワークスペースに追加する方法](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md)」の手順に従っていることを確認してください。

## <a name="deploy-the-application"></a>アプリケーションの配置

サンプル アプリケーションをインストールする前に、Kubernetes クラスターに nginx ベースのイングレス コントローラーを構成する別の手順があります。 イングレス コントローラーは、ホスト、パス、またはプロトコルに基づいてクラスター内のトラフィックをルーティングするためのレイヤー 7 のロード バランサーとして使用されます。 nginx-ingress は、Helm Chart として使用できます。 詳細な手順については、[Helm Chart GitHub リポジトリ](https://github.com/helm/charts/tree/master/stable/nginx-ingress)を参照してください。

さらに、サンプル アプリケーションは、前の手順の [Azure Monitoring エージェント](#configure-monitoring)のように Helm Chart としてパッケージ化されます。 そのため、Kubernetes クラスターへのアプリケーションのデプロイを簡単に実行できます。 [コンパニオン GitHub リポジトリで Helm Chart](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm) を確認できます。

サンプル アプリケーションは、2 つの Azure Stack Hub インスタンスの Kubernetes クラスターにデプロイされる 3 層アプリケーションです。 アプリケーションでは、MongoDB データベースが使用されます。 複数のインスタンスにレプリケートされたデータを取得する方法の詳細については、パターンの「[データとストレージに関する考慮事項](pattern-highly-available-kubernetes.md#data-and-storage-considerations)」を参照してください。

アプリケーション用の Helm Chart をデプロイした後、単一のポッド、デプロイ、およびステートフル セット (データベース用) として表されたアプリケーションの 3 つの層のすべてを表示できます。

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

サービス側では、nginx ベースのイングレス コントローラーとそのパブリック IP アドレスを確認できます。

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

"外部 IP" アドレスが "アプリケーション エンドポイント" です。 それはユーザーがアプリケーションを開くために接続する方法であり、次の手順の「[Traffic Manager を構成する](#configure-traffic-manager)」でエンドポイントとしても使用されます。

## <a name="autoscale-the-application"></a>アプリケーションの自動スケーリングを行う
必要に応じて [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) を構成して、CPU 使用率などの特定のメトリックに基づくスケールアップまたはスケールダウンを実行できます。 次のコマンドを使用すると、ratings-web デプロイによって制御されるポッドの 1 から 10 個のレプリカを保持する Horizontal Pod Autoscaler が作成されます。 HPA によって、すべてのポッドの平均 CPU 使用率を 80% に保つために、レプリカの数が (デプロイを介して) 増減されます。

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
以下を実行することで、オートスケーラーの現在の状態を確認できます。

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Traffic Manager を構成する

アプリケーションの 2 つ (以上) のデプロイ間でトラフィックを分散するために、[Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) を使用します。 Azure Traffic Manager は、Azure 内の DNS ベースのトラフィック ロード バランサーです。

> [!NOTE]
> Traffic Manager では、DNS を使用して、トラフィック ルーティング方法とエンドポイントの正常性に基づいて最適なサービス エンドポイントにクライアント要求が送信されます。

Azure Traffic Manager を使用する代わりに、オンプレミスでホストされている他のグローバルな負荷分散ソリューションを使用することもできます。 サンプル シナリオでは、Azure Traffic Manager を使用して、アプリケーションの 2 つのインスタンス間にトラフィックを分散します。 同じ場所または異なる場所にある Azure Stack Hub インスタンスで実行できます。

![オンプレミスのトラフィック マネージャー](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

Azure で、アプリケーションの 2 つの異なるインスタンスをポイントするように Traffic Manager を構成します。

[![TM エンドポイント プロファイル](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

ご覧のように、2 つのエンドポイントによって、[前のセクション](#deploy-the-application)でデプロイされたアプリケーションの 2 つのインスタンスがポイントされています。

この時点で:
- Kubernetes インフラストラクチャが作成されています (イングレス コントローラーが含まれています)。
- クラスターは 2 つの Azure Stack Hub インスタンスにデプロイされています。
- 監視が構成されています。
- Azure Traffic Manager によって、2 つの Azure Stack Hub インスタンスにトラフィックが負荷分散されます。
- インフラストラクチャの上部に、3 層のサンプル アプリケーションが、Helm Charts を使用して自動化された方法でデプロイされています。 

これで、ソリューションが起動し、ユーザーがアクセスできるようになります。

検討する価値があるデプロイ後の運用上の考慮事項がいくつかあります。それらについて、次の 2 つのセクションで説明します。

## <a name="upgrade-kubernetes"></a>Kubernetes のアップグレード

Kubernetes クラスターをアップグレードするときは、次のトピックを考慮してください。

- Kubernetes クラスターのアップグレードは、AKS Engine を使用して実行できる複雑な運用段階の操作です。 詳細については、「[Azure Stack Hub で Kubernetes クラスターをアップグレードする](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade)」を参照してください。
- AKS Engine を使用して、クラスターの Kubernetes とベース OS イメージを新しいバージョンにアップグレードできます。 詳細については、「[新しい Kubernetes バージョンにアップグレードする手順](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)」を参照してください。 
- 基礎になるノードだけをベース OS イメージの新しいバージョンにアップグレードすることもできます。 詳細については、「[OS イメージのみをアップグレードする手順](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)」を参照してください。

新しいベース OS イメージには、セキュリティとカーネルの更新プログラムが含まれています。 新しい Kubernetes バージョンと OS イメージの可用性を監視するのはクラスター オペレーターの責任です。 オペレーターは、AKS Engine を使用するこれらのアップグレードを計画して実行する必要があります。 ベース OS イメージは、Azure Stack Hub オペレーターが Azure Stack Hub Marketplace からダウンロードする必要があります。

## <a name="scale-kubernetes"></a>Kubernetes をスケーリングする

スケーリングは、AKS Engine を使用して調整できる別の運用段階の操作です。

スケーリング コマンドでは、出力ディレクトリ内のクラスター構成ファイル (apimodel.json) が、新しい Azure Resource Manager デプロイの入力として再利用されます。 AKS Engine により、特定のエージェント プールに対してスケーリング操作が実行されます。 スケーリング操作が完了すると、AKS Engine により、同じ apimodel. json ファイル内のクラスター定義が更新されます。 クラスター定義には、更新された最新のクラスター構成を反映するために、新しいノード数が反映されます。

- [Azure Stack Hub で Kubernetes クラスターをスケールする](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>次のステップ

- [ハイブリッド アプリの設計上の考慮事項](overview-app-design-considerations.md)の詳細を確認する
- [GitHub でこのサンプルのコード](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)を確認し、改善点を提案する