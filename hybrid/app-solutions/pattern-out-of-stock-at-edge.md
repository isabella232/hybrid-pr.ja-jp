---
title: Azure と Azure Stack Edge を使用した在庫切れ検出
description: Azure と Azure Stack Edge サービスを使用して、在庫切れの検出を実装する方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 865f63bc4234e50ed169aa29cefdb1886750594c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911189"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>エッジ パターンでの在庫切れ検出

このパターンは、Azure Stack Edge または Azure IoT Edge デバイスとネットワーク カメラを使用して、棚に在庫切れの商品がないかどうかを確認する方法を示しています。

## <a name="context-and-problem"></a>コンテキストと問題

顧客が商品を探しているときにその商品が棚にしないという理由で、実際の小売店では売上が失われます。 しかし、商品は店の奥にあって、補充されていない場合があります。 店舗では、スタッフをより効率的に利用して、商品の補充が必要になったときに自動的に通知を受け取れるようにしたいと考えています。

## <a name="solution"></a>解決策

このソリューションの例では、各店舗の Azure Stack Edge などのエッジ デバイスを使用して、店内のカメラからのデータを効率的に処理します。 この最適化された設計により、店舗は関連するイベントや画像のみをクラウドに送信できます。 この設計によって、帯域幅とストレージ領域が節約され、顧客のプライバシーが確保されます。 各カメラからフレームが読み込まれると、ML モデルによって画像が処理され、在庫切れの領域があれば返されます。 画像および在庫切れの領域がローカル Web アプリに表示されます。 このデータを Time Series Insight 環境に送信して、Power BI で分析情報を表示することができます。

![エッジでの在庫切れのソリューション アーキテクチャ](media/pattern-out-of-stock-at-edge/solution-architecture.png)

このソリューションのしくみを次に示します。

1. 画像は、HTTP または RTSP 経由でネットワーク カメラからキャプチャされます。
2. 画像のサイズが変更され、推論ドライバーに送信されます。このドライバーでは、ML モデルと通信して、在庫切れの画像があるかどうかが判断されます。
3. ML モデルにより在庫切れの領域が返されます。
4. 推論ドライバーによって未加工の画像が BLOB にアップロードされ (指定されている場合)、モデルからの結果が Azure IoT Hub とデバイスの境界ボックス プロセッサに送信されます。
5. 境界ボックス プロセッサにより、境界ボックスが画像に追加され、画像パスがメモリ内データベースにキャッシュされます。
6. Web アプリによって画像が照会され、それらが受信された順に表示されます。
7. IoT Hub からのメッセージは Time Series Insights に集約されます。
8. Power BI には、Time Series Insights のデータと共に、長期間にわたる在庫切れ商品の対話型レポートが表示されます。


## <a name="components"></a>Components

このソリューションでは、次のコンポーネントを使用します。

| レイヤー | コンポーネント | 説明 |
|----------|-----------|-------------|
| オンプレミスのハードウェア | ネットワーク カメラ | ネットワーク カメラは、推論用の画像を提供するため、HTTP または RTSP フィードで必要です。 |
| Azure | Azure IoT Hub | [Azure IoT Hub](/azure/iot-hub/) では、デバイス プロビジョニングおよびエッジ デバイスのメッセージングが処理されます。 |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/) により、視覚化のために IoT Hub からのメッセージが格納されます。 |
|  | Power BI | [Microsoft Power BI](https://powerbi.microsoft.com/) では、ビジネスに重点を置いた在庫切れイベントのレポートが提供されます。 Power BI には、Azure Stream Analytics からの出力を表示するための、使いやすいダッシュボード インターフェイスが用意されています。 |
| Azure Stack Edge または<br>Azure IoT Edge デバイス | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) では、オンプレミスのコンテナーのランタイムが調整され、デバイスの管理と更新が処理されます。|
| | Azure Project Brainwave | Azure Stack Edge デバイスでは、ML 推論を高速化するため、[Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) で Field-Programmable Gate Arrays (FPGA) が使用されます。|

## <a name="issues-and-considerations"></a>問題と注意事項

このソリューションの実装方法を決めるときには、以下の点を考慮してください。

### <a name="scalability"></a>スケーラビリティ

ほとんどの機械学習モデルでは、指定されたハードウェアに応じて、1 秒あたり一定のフレーム数でのみ実行できます。 ML パイプラインでバックアップが行われないようにするには、ご使用のカメラから最適なサンプル レートを判断します。 ハードウェアの種類によって、処理するカメラの数とフレーム レートが異なります。

### <a name="availability"></a>可用性

エッジ デバイスで接続が失われた場合に何が発生する可能性があるかを考慮することが重要です。 Time Series Insights と Power BI ダッシュボードから失われる可能性があるデータについて考慮してください。 提供されているソリューションの例は、高可用性を実現するように設計されていません。

### <a name="manageability"></a>管理の容易性

このソリューションは多数のデバイスと場所にまたがることがあるため、扱いにくくなる可能性があります。 Azure の IoT サービスにより、新しい場所とデバイスを自動的にオンラインにし、最新の状態に保つことができます。 適切なデータ ガバナンス手順にも従う必要があります。

### <a name="security"></a>Security

このパターンでは、潜在的に敏感なデータが処理されます。 キーが定期的にローテーションされており、Azure Storage アカウントとローカル共有のアクセス許可が正しく設定されていることを確認します。

## <a name="next-steps"></a>次のステップ

この記事で紹介したトピックの関連情報:
- このパターンでは、[Azure IoT Edge](/azure/iot-edge/)、[Azure IoT Hub](/azure/iot-hub/)、[Azure Time Series Insights](/azure/time-series-insights/) など、複数の IoT 関連サービスが使用されます。
- Microsoft Project Brainwave の詳細については、[ブログの発表](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)を参照してください。また、[Azure Accelerated Machine Learning with Project Brainwave (Project Brainwave による Azure Machine Learning の高速化) のビデオ](https://www.youtube.com/watch?v=DJfMobMjCX0)をご覧ください。
- ベスト プラクティスの詳細とその他の疑問の回答を得るには、「[ハイブリッド アプリケーション設計に関する考慮事項](overview-app-design-considerations.md)」を参照してください。
- 製品とソリューションのポートフォリオ全体の詳細について、[Azure Stack ファミリの製品とソリューション](/azure-stack)を参照してください。

ソリューションの例をテストする準備ができたら、[分析ソリューションの階層化データのデプロイ ガイド](https://aka.ms/edgeinferencingdeploy)に進んでください。 デプロイ ガイドでは、コンポーネントをデプロイしてテストするための詳細な手順について説明します。
