---
title: Azure と Azure Stack Hub のハイブリッド リレー パターン
description: Azure と Azure Stack Hub のハイブリッド リレー パターンを使用して、ファイアウォールで保護されているエッジ リソースに接続します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911147"
---
# <a name="hybrid-relay-pattern"></a>ハイブリッド リレー パターン

ハイブリッドリレーパターンと Azure Relay を使用して、ファイアウォールで保護されているエッジリソースまたはデバイスに接続する方法について説明します。

## <a name="context-and-problem"></a>コンテキストと問題

多くの場合、エッジ デバイスは、企業のファイアウォールまたは NAT デバイスの背後にあります。 これらはセキュリティで保護されていますが、パブリック クラウド、または他の企業ネットワーク上のエッジ デバイスと通信できない可能性があります。 このため、パブリック クラウド内のユーザーに特定のポートおよび機能を安全な方法で公開することが必要になる場合があります。

## <a name="solution"></a>解決策

ハイブリッドリレーパターンでは、Azure Relay を使用して、直接通信できない2つのエンドポイント間に Websocket トンネルを確立します。 オンプレミスではないが、オンプレミスのエンドポイントに接続する必要があるデバイスは、パブリック クラウド内のエンドポイントに接続されます。 このエンドポイントは、セキュリティ保護されたチャネルを介して、事前定義されたルートでトラフィックをリダイレクトします。 オンプレミスの環境内にあるエンドポイントはトラフィックを受信し、それを正しい宛先にルーティングします。

![ハイブリッド リレー パターンのソリューション アーキテクチャ](media/pattern-hybrid-relay/solution-architecture.png)

ハイブリッド リレー パターンがどのように機能するかを次に示します。

1. デバイスが、事前定義されたポートで、Azure 内の仮想マシン (VM) に接続されます。
2. トラフィックは、Azure の Azure Relay に転送されます。
3. Azure Relay への長時間接続が確立されている Azure Stack Hub 上の VM は、トラフィックを受信し、転送先に転送します。
4. オンプレミスのサービスまたはエンドポイントがこの要求を処理します。

## <a name="components"></a>Components

このソリューションでは、次のコンポーネントを使用します。

| レイヤー | コンポーネント | 説明 |
|----------|-----------|-------------|
| Azure | Azure VM | Azure VM は、オンプレミス リソースに対して、パブリックにアクセス可能なエンドポイントを提供します。 |
| | Azure Relay | [Azure Relay](/azure/azure-relay/)には、Azure vm と AZURE STACK Hub vm 間のトンネルと接続を維持するためのインフラストラクチャが用意されています。|
| Azure Stack Hub | Compute | Azure Stack Hub VM は、サーバー側のハイブリッド リレー トンネルを提供します。 |
| | ストレージ | Azure Stack Hub にデプロイされた AKS エンジン クラスターによって、Face API コンテナーを実行するためのスケーラブルで回復性があるエンジンが提供されます。|

## <a name="issues-and-considerations"></a>問題と注意事項

このソリューションの実装方法を決めるときには、以下の点を考慮してください。

### <a name="scalability"></a>スケーラビリティ

このパターンでは、クライアントとサーバーで 1:1 のポート マッピングのみを使用できます。 たとえば、ポート 80 が Azure エンドポイント上のあるサービス用にトンネリングされている場合、このポートを別のサービスに使用することはできません。 ポート マッピングを適切に計画する必要があります。 トラフィックを処理するには、Azure Relay と Vm を適切にスケーリングする必要があります。

### <a name="availability"></a>可用性

これらのトンネルと接続は冗長ではありません。 高可用性を確保するために、エラー チェック コードを実装することをお勧めします。 もう1つの方法は、ロードバランサーの背後に Azure Relay 接続された Vm のプールを作成することです。

### <a name="manageability"></a>管理の容易性

このソリューションは多数のデバイスと場所にまたがることがあるため、扱いにくくなる可能性があります。 Azure の IoT サービスにより、新しい場所とデバイスを自動的にオンラインにし、最新の状態に保つことができます。

### <a name="security"></a>Security

図のように、このパターンでは、エッジから内部デバイス上のポートに自由にアクセスできます。 内部デバイス上のサービス、またはハイブリッド リレー エンドポイントの前面に、認証メカニズムを追加することを検討してください。

## <a name="next-steps"></a>次のステップ

この記事で紹介したトピックの関連情報:

- このパターンでは、Azure Relay を使用します。 詳細については、 [Azure Relay のドキュメント](/azure/azure-relay/)を参照してください。
- ベスト プラクティスの詳細を確認し、その他の疑問の回答を得るには、「[ハイブリッド アプリケーション設計に関する考慮事項](overview-app-design-considerations.md)」を参照してください。
- 製品とソリューションのポートフォリオ全体の詳細について、[Azure Stack ファミリの製品とソリューション](/azure-stack)を参照してください。

ソリューションの例をテストする準備ができたら、[ハイブリッド リレーのソリューション デプロイ ガイド](https://aka.ms/hybridrelaydeployment)に進んでください。 デプロイ ガイドでは、コンポーネントをデプロイしてテストするための詳細な手順について説明します。