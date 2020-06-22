---
title: Azure と Azure Stack Hub 向けのハイブリッド パターンとソリューションの例
description: Azure と Azure Stack Hub でのハイブリッド ソリューションの学習と構築のためのハイブリッド パターンとソリューションの例の概要。
author: BryanLa
ms.topic: overview
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ab0eb885e7b0fefaca8991522712652f979d8712
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910918"
---
# <a name="hybrid-patterns-and-solution-examples-for-azure-and-azure-stack"></a>Azure と Azure Stack 向けのハイブリッド パターンとソリューションの例

Microsoft では、Azure と Azure Stack の製品とソリューションを 1 つの一貫した Azure エコシステムとして提供しています。 Microsoft Azure Stack ファミリは Azure の拡張機能です。

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>ハイブリッド クラウドとハイブリッド アプリ

Azure Stack は、"*ハイブリッド クラウド*" を有効にすることにより、オンプレミス環境とエッジにクラウド コンピューティングの機敏性を提供します。 Azure Stack Hub、Azure Stack HCI、Azure Stack Edge により、Azure がクラウドから専用のデータセンター、支店、現場などにまで拡張されます。 このさまざまな機能セットを使用すると、次のことが可能になります。

- コードを再利用し、Azure とオンプレミス環境全体でクラウド ネイティブ アプリを一貫した方法で実行する。
- Azure サービスへのオプション接続を使用して、従来の仮想化されたワークロードを実行する。
- データをクラウドに転送するか、専用のデータセンターに保管してコンプライアンスを維持する。
- ハードウェア アクセラレータによる機械学習、コンテナー化、または仮想化されたワークロードをすべてインテリジェント エッジで実行する。

クラウドをまたぐアプリは "*ハイブリッド アプリ*" とも呼ばれます。 Azure でハイブリッド クラウド アプリを構築し、接続されている/いないを問わず、あらゆる場所にあるデータセンターにデプロイすることができます。

ハイブリッド アプリのシナリオは、開発に使用できるリソースによって大きく異なります。 また、地理、セキュリティ、インターネット アクセスなどの考慮事項も存在します。 ここで説明するパターンとソリューションは、すべての要件に対応しているとは限りませんが、ハイブリッド ソリューションを実装する際に探索して再利用するためのガイドラインと例を提供します。

## <a name="design-patterns"></a>設計パターン

設計パターンは、実際の顧客シナリオや経験から、一般化された反復可能な設計ガイダンスを選び抜いたものです。 パターンは抽象的であるため、さまざまな種類のシナリオや業界に適用できます。 各パターンにはコンテキストと問題が文書化されており、ソリューションの例の概要を示します。 このソリューションの例は、考えられるパターンの実装を示しています。

パターンの記事には、次の 2 種類があります。

- 単一パターン: 単一の汎用シナリオ向けの設計ガイダンスを提供します。
- 複数パターン: 複数のパターンの適用を使用する設計ガイダンスを提供します。 多くの場合、このパターンは、より複雑なシナリオや業界固有の問題を解決するために必要となります。

## <a name="solution-deployment-guides"></a>ソリューション デプロイ ガイド

ステップバイステップのデプロイ ガイドにより、ソリューションの例をデプロイできます。 このガイドでは、GitHub の[ソリューション サンプル リポジトリ](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)に格納されているコンパニオン コード サンプルを参照する場合もあります。

## <a name="next-steps"></a>次のステップ

- 製品とソリューションのポートフォリオ全体の詳細について、[Azure Stack ファミリの製品とソリューション](/azure-stack)を参照してください。
- 目次の「パターン」セクションと「ソリューション デプロイ ガイド」セクションを探索し、それぞれの詳細を参照してください。
- 「[ハイブリッド アプの設計の考慮事項](overview-app-design-considerations.md)」で、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素を確認します。
- [Azure Stack で開発環境を設定](/azure-stack/user/azure-stack-dev-start.md)し、Azure Stack で[最初のアプリをデプロイ](/azure-stack/user/azure-stack-dev-start-deploy-app.md)します。
