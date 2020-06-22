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
# <a name="train-machine-learning-model-at-the-edge-pattern"></a>エッジ パターンで機械学習モデルをトレーニングする

オンプレミスにのみ存在するデータから移植可能な機械学習 (ML) モデルを生成します。

## <a name="context-and-problem"></a>コンテキストと問題

データ サイエンティストが理解するツールを利用し、オンプレミスまたはレガシ データから分析情報を解き明かすことを望む組織はたくさん存在します。 [Azure Machine Learning](/azure/machine-learning/) では、ML とディープ ラーニング モデルをトレーニング、チューニング、デプロイするためのクラウドネイティブ ツールが提供されます。  

しかしながら、一部のデータは大きすぎて、あるいは、規制上の理由から、クラウドに送信できません。 このパターンを使用すると、データ サイエンティストは Azure Machine Learning を利用し、オンプレミス データとコンピューティングでモデルをトレーニングできます。

## <a name="solution"></a>解決策

エッジ パターンのトレーニングでは、Azure Stack Hub で実行されている仮想マシン (VM) が使用されます。 VM は Azure ML でコンピューティング先として登録され、オンプレミスでのみ利用できるデータにアクセスすることが許可されます。 この場合、データは Azure Stack Hub の BLOB ストレージに保存されます。

モデルがトレーニングされると、Azure ML に登録され、コンテナー化され、Azure Container Registry にデプロイのために追加されます。 このパターンを反復するためには、Azure Stack Hub トレーニング VM に公共のインターネット経由でアクセスできる必要があります。

[![エッジ アーキテクチャで ML モデルをトレーニングする](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)

パターンのしくみは次のとおりです。

1. Azure Stack Hub VM がデプロイされ、Azure ML にコンピューティング先として登録されます。
2. Azure ML で、コンピューティング先として Azure Stack Hub VM を使用する実験が作成されます。
3. トレーニングされたモデルは登録され、コンテナー化されます。
4. これで、オンプレミスまたはクラウドにある場所にモデルをデプロイできます。

## <a name="components"></a>Components

このソリューションでは、次のコンポーネントを使用します。

| レイヤー | コンポーネント | 説明 |
|----------|-----------|-------------|
| Azure | Azure Machine Learning | [Azure Machine Learning](/azure/machine-learning/) により ML モデルのトレーニングが調整されます。 |
| | Azure Container Registry | Azure ML によりモデルがパッケージ化され、コンテナーが作られ、デプロイのために [Azure Container Registry](/azure/container-registry/) に格納されます。|
| Azure Stack Hub | App Service | [Azure Stack Hub と App Service](/azure-stack/operator/azure-stack-app-service-overview) は末端のコンポーネントの基礎となります。 |
| | Compute | Ubuntu と Docker を実行する Azure Stack Hub VM は ML モデルのトレーニングに使用されます。 |
| | ストレージ | 非公開データは Azure Stack Hub BLOB ストレージでホストできます。 |

## <a name="issues-and-considerations"></a>問題と注意事項

このソリューションの実装方法を決めるときには、以下の点を考慮してください。

### <a name="scalability"></a>スケーラビリティ

このソリューションをスケーリングするには、トレーニングのために Azure Stack Hub で適切なサイズの VM を作成する必要があります。

### <a name="availability"></a>可用性

トレーニング スクリプトと Azure Stack Hub VM でトレーニングに使用されるオンプレミス データにアクセスできることを確認します。

### <a name="manageability"></a>管理の容易性

モデル デプロイ中の混乱を避けるため、モデルと実験が正しく登録され、バージョン管理され、タグ付けされていることを確認します。

### <a name="security"></a>Security

このパターンを使用することで、Azure ML からオンプレミスの機密性が高い可能性のあるデータにアクセスできるようになります。 Azure Stack Hub VM に SSH 接続するためのアカウントに強固なパスワードが与えられていることとトレーニング スクリプトではデータが保存されず、クラウドにアップロードされないことを確認します。

## <a name="next-steps"></a>次のステップ

この記事で紹介したトピックの関連情報:

- ML と関連トピックの概要については、[Azure Machine Learning のドキュメント](/azure/machine-learning)を参照してください。
- コンテナー デプロイ向けのイメージを作成、格納、管理する方法については、「[Azure Container Registry](/azure/container-registry/)」を参照してください。
- リソース プロバイダーとデプロイ方法の詳細については、「[Azure Stack 上の App Service の概要](/azure-stack/operator/azure-stack-app-service-overview)」を参照してください。
- ベスト プラクティスの詳細を確認し、その他の疑問の回答を取得するには、[ハイブリッド アプリケーションの設計上の考慮事項](overview-app-design-considerations.md)に関する記事を参照してください。
- 製品とソリューションのポートフォリオ全体の詳細について、[Azure Stack ファミリの製品とソリューション](/azure-stack)を参照してください。

ソリューションの例をテストする準備ができたら、[エッジ デプロイの ML モデル トレーニング ガイド](https://aka.ms/edgetrainingdeploy)に進んでください。 デプロイ ガイドでは、コンポーネントをデプロイしてテストするための詳細な手順について説明します。
