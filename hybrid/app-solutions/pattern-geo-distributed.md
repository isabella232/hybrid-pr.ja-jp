---
title: Azure Stack Hub の地理的に分散されたアプリ パターン
description: Azure と Azure Stack Hub を使用した、インテリジェント エッジの地理的に分散されたアプリ パターンについて説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 3c839d9bf3b6c3e1ff50cc695fd5f1a1127793d2
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281229"
---
# <a name="geo-distributed-app-pattern"></a>地理的に分散されたアプリ パターン

複数のリージョンにアプリ エンドポイントを提供し、場所とコンプライアンスのニーズに基づいてユーザー トラフィックをルーティングする方法について説明します。

## <a name="context-and-problem"></a>コンテキストと問題

広い地理的範囲にまたがる組織は、必要なレベルのセキュリティ、コンプライアンス、およびパフォーマンスを、境界を越えてユーザー、場所、およびデバイスごとに確保しながら、データへのアクセスを安全かつ正確に分散させて実現するために努力します。

## <a name="solution"></a>解決策

Azure Stack Hub の地理的トラフィック ルーティング パターン (地理的に分散されたアプリ) では、さまざまなメトリックに基づいてトラフィックを特定のエンドポイントに送信できます。 地理的ベースのルーティングとエンドポイント構成を使用して Traffic Manager を作成すると、リージョンの要件、企業および国際的な規制、およびデータ ニーズに基づいてトラフィックをエンドポイントにルーティングできます。

![地理的に分散されたパターン](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>コンポーネント

### <a name="outside-the-cloud"></a>クラウドの外部

#### <a name="traffic-manager"></a>Traffic Manager

この図では、Traffic Manager はパブリック クラウドの外部にありますが、ローカル データセンターとパブリック クラウドの両方でトラフィックを調整できる必要があります。 バランサーは地理的な場所にトラフィックをルーティングします。

#### <a name="domain-name-system-dns"></a>ドメイン ネーム システム (DNS)

ドメイン ネーム システム (DNS) は、Web サイトまたはサービスの名前をその IP アドレスに変換する (または解決する) 役割を担います。

### <a name="public-cloud"></a>パブリック クラウド

#### <a name="cloud-endpoint"></a>クラウド エンドポイント

パブリック IP アドレスは、受信トラフィックをトラフィック マネージャー経由でパブリック クラウド アプリのリソース エンドポイントにルーティングするために使用されます。  

### <a name="local-clouds"></a>ローカル クラウド

#### <a name="local-endpoint"></a>ローカル エンドポイント

パブリック IP アドレスは、受信トラフィックをトラフィック マネージャー経由でパブリック クラウド アプリのリソース エンドポイントにルーティングするために使用されます。

## <a name="issues-and-considerations"></a>問題と注意事項

このパターンの実装方法を決めるときには、以下の点に注意してください。

### <a name="scalability"></a>スケーラビリティ

このパターンでは、トラフィックの増加に合わせてスケールするのではなく、地理的なトラフィック ルーティングを処理します。 ただし、このパターンを他の Azure およびオンプレミスのソリューションと組み合わせることができます。 たとえば、このパターンはクラウド間スケーリング パターンと一緒に使用できます。

### <a name="availability"></a>可用性

オンプレミス ハードウェア構成およびソフトウェア デプロイを通じて高可用性をもたらすように、ローカルでデプロイされたアプリが構成されていることを確認します。

### <a name="manageability"></a>管理の容易性

パターンは、複数の環境にわたるシームレスな管理と使い慣れたインターフェイスを実現します。

## <a name="when-to-use-this-pattern"></a>このパターンを使用する状況

- 組織には、リージョンのセキュリティおよび分散に関するカスタム ポリシーが必要な支店が世界中にあります。
- 組織の支店それぞれは、従業員、ビジネス、および施設のデータをプルしますが、これには地域の規制やタイム ゾーンに従ったレポート アクティビティが必要になります。
- 高スケール要件を満たすには、極端に負荷の大きい要件に対応できるように、単一のリージョン内、および複数のリージョンにわたって、複数のアプリ デプロイを使用してアプリを水平方向に拡張します。
- 1 つのリージョンで障害が発生した場合でも、アプリは高可用性とクライアント要求への応答性を実現する必要があります。

## <a name="next-steps"></a>次のステップ

この記事で紹介したトピックの関連情報:

- この DNS ベースのトラフィック ロード バランサーの詳細なしくみについては、「[Traffic Manager について](/azure/traffic-manager/traffic-manager-overview)」を参照してください。
- ベスト プラクティスの詳細とその他の疑問の回答を得るには、「[ハイブリッド アプリケーション設計に関する考慮事項](overview-app-design-considerations.md)」を参照してください。
- 製品とソリューションのポートフォリオ全体の詳細について、[Azure Stack ファミリの製品とソリューション](/azure-stack)を参照してください。

ソリューションの例をテストする準備ができたら、[地理的に分散されたアプリのソリューション デプロイ ガイド](/azure/architecture/hybrid/deployments/solution-deployment-guide-geo-distributed)に進んでください。 デプロイ ガイドでは、コンポーネントをデプロイしてテストするための詳細な手順について説明します。 地理的に分散されたアプリ パターンを使用し、さまざまなメトリックに基づいて特定のエンドポイントにトラフィックを送信する方法を学習します。 地理的ベースのルーティングとエンドポイント構成で Traffic Manager プロファイルを作成すると、リージョンの要件、企業および国際的な規制、およびデータ ニーズに基づいて、情報がエンドポイントにルーティングされます。