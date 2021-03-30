---
title: Azure と Azure Stack Hub を使用した足取り検出パターン
description: Azure および Azure Stack Hub を使用して AI ベースの足取り検出ソリューションを実装し、小売店内のトラフィックを分析する方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 866557ec3af2337e9f034da84cf417675508563b
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895331"
---
# <a name="footfall-detection-pattern"></a>足取り検出パターン

このパターンでは、小売店で買い物客のトラフィックを分析するために、AI ベースの足取り検出ソリューションを実装するための概要を提供します。 このソリューションでは、Azure、Azure Stack Hub、Custom Vision AI Dev Kit を使用して、実際の行動から分析情報を生成します。

## <a name="context-and-problem"></a>コンテキストと問題

Contoso Stores では、店舗のレイアウトに関連して、顧客が現在の製品をどのように受け取っているかについて分析情報を得たいと考えています。 すべてのセクションにスタッフを配置することはできません。また、カメラ撮影した店舗内の映像すべてをアナリスト チームに確認してもらうのも非効率です。 さらに、分析用の映像をすべてのカメラからクラウドにストリーミングできるだけの十分な帯域幅は、どの店舗にもありません。

Contoso では、顧客の人口統計、ロイヤルティ、店舗のディスプレイと製品に対する反応を確認するための、邪魔にならずプライバシーに配慮した方法を見つけたいと考えています。

## <a name="solution"></a>ソリューション

この小売分析パターンでは、階層型アプローチを使用してエッジで推論を行います。 Custom Vision AI Dev Kit を使用することで、人の顔が含まれる画像のみが、分析のためにプライベート Azure Stack Hub に送信されます。Azure Stack Hub では、Azure Cognitive Services が実行されています。 集計された匿名データが Azure に送信され、すべての店舗にわたって集計されて Power BI で視覚化されます。 エッジとパブリック クラウドを組み合わせることにより、Contoso は最新の AI テクノロジを活用しながら、企業のポリシーに準拠し、顧客のプライバシーを尊重することができます。

[![足取り検出パターン ソリューション](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

このソリューションのしくみの概要を以下に示します。

1. Custom Vision AI Dev Kit は IoT Hub から構成を取得します。これにより IoT Edge ランタイムと ML モデルがインストールされます。
2. モデルが人を認識すると、写真が撮影され、Azure Stack Hub BLOB ストレージにアップロードされます。
3. Blob service によって Azure Stack Hub で Azure 関数がトリガーされます。
4. Azure 関数は、Face API を含むコンテナーを呼び出し、画像から人口統計データと感情データを取得します。
5. このデータは匿名化され、Azure Event Hubs クラスターに送信されます。
6. Event Hubs クラスターにより、データは Stream Analytics にプッシュされます。
7. Stream Analytics によってデータが集計され、Power BI にプッシュされます。

## <a name="components"></a>Components

このソリューションでは、次のコンポーネントを使用します。

| レイヤー | コンポーネント | 説明 |
|----------|-----------|-------------|
| 店舗内のハードウェア | [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) | 分析用に人の画像のみをキャプチャするローカル ML モデルを使用して、ストア内でフィルター処理を行います。 IoT Hub によって安全にプロビジョニングと更新が行われます。<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Azure Event Hubs は、Azure Stream Analytics と密接に統合された、匿名データを取り込むためのスケーラブルなプラットフォームを提供します。 |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Azure Stream Analytics ジョブは、匿名データを集計し、これを視覚化できるように 15 秒間隔でグループ化します。 |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI には、Azure Stream Analytics からの出力を表示するための、使いやすいダッシュボード インターフェイスが用意されています。 |
| Azure Stack Hub | [App Service](/azure-stack/operator/azure-stack-app-service-overview) | App Service リソースプロバイダー (RP) は、Web アプリ/API および関数のホスティングや管理の機能を含む、エッジ コンポーネントの基本を提供します。 |
| | [Azure Kubernetes Service (AKS) エンジン](https://github.com/Azure/aks-engine) クラスター | AKS RP では AKS エンジンが Azure Stack Hub にデプロイされているため、Face API コンテナーを実行するためのスケーラブルで回復力のあるエンジンが提供されます。 |
| | Azure Cognitive Services の [Face API コンテナー](/azure/cognitive-services/face/face-how-to-install-containers)| Face API コンテナーを備えた Azure Cognitive Services RP により、Contoso のプライベート ネットワーク上で人口統計、感情、および買い物客の一意検出が実現します。 |
| | Blob Storage | AI Dev Kit からキャプチャされた画像は、Azure Stack Hub の BLOB ストレージにアップロードされます。 |
| | Azure Functions | Azure Stack Hub で実行されている Azure Functions は、BLOB ストレージからの入力を受け取り、Face API とのやり取りを管理します。 次に、匿名データを、Azure にある Event Hubs クラスターに出力します。<br><br>|

## <a name="issues-and-considerations"></a>問題と注意事項

このソリューションの実装方法を決めるときには、以下の点を考慮してください。

### <a name="scalability"></a>スケーラビリティ

このソリューションを複数のカメラと場所にわたってスケーリングできるようにするには、増えた負荷をすべてのコンポーネントで処理できるようにする必要があります。 次のような措置を取る必要が生じる場合があります。

- Stream Analytics のストリーミング ユニット数を増やす。
- Face API デプロイをスケールアウトする。
- Event Hubs クラスターのスループットを増やす。
- 極端なケースでは、Azure Functions から仮想マシンへの移行が必要になる場合があります。

### <a name="availability"></a>可用性

このソリューションは階層化されているため、ネットワークや電源の障害に対処する方法を検討しておく必要があります。 ビジネス ニーズに応じて、画像をローカルにキャッシュし、接続が回復したときに Azure Stack Hub に転送するメカニズムを実装したい場合があります。 場所に余裕がある場合は、Face API コンテナーを備えた Data Box Edge をその場所に配置する方が適切である可能性もあります。

### <a name="manageability"></a>管理の容易性

このソリューションは多数のデバイスと場所にまたがることがあるため、扱いにくくなる可能性があります。 [Azure の IoT サービス](/azure/iot-fundamentals/)を使用することで、新しい場所とデバイスを自動的にオンラインにし、最新の状態に保つことができます。

### <a name="security"></a>セキュリティ

このソリューションでは顧客の画像をキャプチャするため、セキュリティが最も重要な考慮事項となります。 すべてのストレージ アカウントが適切なアクセス ポリシーで保護されていることを確認し、またキーを定期的にローテーションします。 ストレージ アカウントと Event Hubs に、企業および政府のプライバシー規制に準拠したアイテム保持ポリシーが備わっていることを確認します。 また、ユーザー アクセス レベルは必ず階層化してください。 階層化により、ユーザーは各自のロールに必要なデータにだけアクセスできるようになります。

## <a name="next-steps"></a>次のステップ

この記事で紹介したトピックの関連情報:

- 足取り検出パターンによって利用される、[階層化データ パターン](https://aka.ms/tiereddatadeploy)に関するページをご覧ください。
- Custom Vision の使用に関する詳細については、[Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) に関するページをご覧ください。 

ソリューションの例をテストする準備ができたら、[足取り検出のデプロイ ガイド](solution-deployment-guide-retail-footfall-detection.md)に進んでください。 デプロイ ガイドでは、コンポーネントをデプロイしてテストするための詳細な手順について説明します。
