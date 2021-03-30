---
title: Azure と Azure Stack Hub で AI ベースの足取り検出ソリューションをデプロイする
description: Azure と Azure Stack Hub を使用して小売店の訪問者トラフィックを分析するための AI ベースの足取り検出ソリューションをデプロイする方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: caedbd4758b9ae8c93cf9bb625ed9aac68bfa196
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895367"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a>Azure と Azure Stack Hub を使用して AI ベースの足取り検出ソリューションをデプロイする

この記事では、Azure、Azure Stack Hub、Custom Vision AI Dev Kit を使用して、実際のアクションから分析情報を生成する AI ベースのソリューションをデプロイする方法について説明します。

このソリューションでは、次の方法を学習します。

> [!div class="checklist"]
> - エッジに Cloud Native Application Bundles (CNAB) をデプロイする。 
> - クラウドの境界をまたぐアプリをデプロイします。
> - エッジでの推論に Custom Vision AI Dev Kit を使用する。

> [!Tip]  
> ![ハイブリッドの柱の図](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub は Azure の拡張機能です。 Azure Stack Hub により、オンプレミス環境にクラウド コンピューティングの機敏性とイノベーションがもたらされ、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドが可能になります。  
> 
> [ハイブリッド アプリの設計の考慮事項](overview-app-design-considerations.md)に関する記事では、ハイブリッド アプリを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。 これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。

## <a name="prerequisites"></a>前提条件

このデプロイ ガイドの使用を開始する前に、必ず次のことを行ってください。

- 「[足取り検出パターン](pattern-retail-footfall-detection.md)」トピックを確認します。
- 次のものがある、Azure Stack Development Kit (ASDK) または Azure Stack Hub 統合システム インスタンスへのユーザー アクセス権を取得します。
  - [Azure App Service on Azure Stack Hub リソース プロバイダー](/azure-stack/operator/azure-stack-app-service-overview)がインストールされている。 インストールには、Azure Stack Hub インスタンスへのオペレーター アクセス、または管理者と協力する必要があります。
  - App Service とストレージ クォータを提供するオファーに対するサブスクリプション。 オファーを作成するには、オペレーター アクセスが必要です。
- Azure サブスクリプションへのアクセスを取得します。
  - Azure サブスクリプションがない場合は、始める前に[無料試用版アカウント](https://azure.microsoft.com/free/)にサインアップしてください。
- ディレクトリに 2 つのサービス プリンシパルを作成する
  - 1 つは、Azure サブスクリプションのスコープでアクセスする、Azure リソースで使用するように設定します。
  - 1 つは、Azure Stack Hub サブスクリプションのスコープでアクセスする、Azure Stack Hub リソースで使用するように設定します。
  - サービス プリンシパルの作成とアクセスの承認の詳細については、[アプリ ID を使用してリソースにアクセスする](/azure-stack/operator/azure-stack-create-service-principals)に関するページを参照してください。 Azure CLI を使用する場合は、「[Azure CLI で Azure サービス プリンシパルを作成する](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true)」を参照してください。
- Azure Cognitive Services を Azure または Azure Stack Hub にデプロイします。
  - まず、[Cognitive Services の詳細について説明します](https://azure.microsoft.com/services/cognitive-services/)。
  - 次に、「[Azure Cognitive Services を Azure Stack にデプロイする](/azure-stack/user/azure-stack-solution-template-cognitive-services)」にアクセスして、Azure Stack Hub 上に Cognitive Services をデプロイします。 プレビューにアクセスするには、まず、サインアップする必要があります。
- 未構成の Azure Custom Vision AI Dev Kit をクローンまたはダウンロードします。 詳細については、[Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/) を参照してください。
- Power BI アカウントにサインアップする。
- Azure Cognitive Services Face API サブスクリプション キーとエンドポイント URL。 どちらも「[Cognitive Services を試す](https://azure.microsoft.com/try/cognitive-services/?api=face-api)」の無料試用版を使用して取得できます。 または、[Cognitive Services アカウントの作成](/azure/cognitive-services/cognitive-services-apis-create-account)の手順に従います。
- 次の開発リソースをインストールする。
  - [Azure CLI 2.0](/azure-stack/user/azure-stack-version-profiles-azurecli2)
  - [Docker CE](https://hub.docker.com/search/?type=edition&offering=community)
  - [Porter](https://porter.sh/)。 提供されている CNAB バンドル マニフェストを使用してクラウド アプリをデプロイするには、Porter を使用します。
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [Visual Studio Code 用の Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [Visual Studio Code 用の Python 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [Python](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a>ハイブリッド クラウド アプリをデプロイする

まず、Porter CLI を使用して資格情報セットを生成してから、クラウド アプリをデプロイします。  

1. [https://github.com/azure-samples/azure-intelligent-edge-patterns](https://github.com/azure-samples/azure-intelligent-edge-patterns ) からソリューション サンプル コードを複製するか、ダウンロードします。 

1. Porter によって、アプリのデプロイを自動化する一連の資格情報が生成されます。 資格情報の生成コマンドを実行する前に、次のものを使用できるようにしておく必要があります。

    - Azure リソースにアクセスするためのサービス プリンシパル (サービス プリンシパル ID、キー、テナント DNS など)。
    - ご利用の Azure サブスクリプションのサブスクリプション ID。
    - Azure Stack Hub リソースにアクセスするためのサービス プリンシパル (サービス プリンシパル ID、キー、テナント DNS など)。
    - ご利用の Azure Stack Hub サブスクリプションのサブスクリプション ID。
    - ご利用の Azure Cognitive Services Face API キーとリソース エンドポイント URL。

1. Porter 資格情報生成プロセスを実行し、画面の指示に従います。

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. Porter では、実行するパラメーターのセットも必要です。 パラメーター テキスト ファイルを作成し、次の名前と値のペアを入力します。 必要な値についてサポートが必要な場合は、Azure Stack Hub 管理者に問い合わせてください。

   > [!NOTE] 
   > `resource suffix` 値は、デプロイのリソースが Azure 全体で一意の名前になることを保証するために使用されます。 これは、8 文字以内の文字と数字の一意の文字列である必要があります。

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   テキスト ファイルを保存し、パスをメモしておきます。

1. これで、Porter を使用してハイブリッド クラウド アプリをデプロイする準備ができました。 インストール コマンドを実行し、Azure と Azure Stack Hub にリソースがデプロイされる様子を確認します。

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. デプロイが完了したら、次の値をメモしておきます。
    - カメラの接続文字列。
    - 画像ストレージ アカウントの接続文字列。
    - リソース グループ名。

## <a name="prepare-the-custom-vision-ai-devkit"></a>Custom Vision AI DevKit を準備する

次に、[Vision AI DevKit のクイックスタート](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/)に示されているように、Custom Vision AI Dev Kit を設定します。 また、前の手順で指定した接続文字列を使用して、カメラを設定し、テストします。

## <a name="deploy-the-camera-app"></a>カメラ アプリをデプロイする

Porter CLI を使用して資格情報セットを生成してから、カメラ アプリをデプロイします。

1. Porter によって、アプリのデプロイを自動化する一連の資格情報が生成されます。 資格情報の生成コマンドを実行する前に、次のものを使用できるようにしておく必要があります。

    - Azure リソースにアクセスするためのサービス プリンシパル (サービス プリンシパル ID、キー、テナント DNS など)。
    - ご利用の Azure サブスクリプションのサブスクリプション ID。
    - クラウド アプリをデプロイしたときに指定した画像ストレージ アカウントの接続文字列。

1. Porter 資格情報生成プロセスを実行し、画面の指示に従います。

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. Porter では、実行するパラメーターのセットも必要です。 パラメーター テキスト ファイルを作成し、次のテキストを入力します。 わからない必要な値がいくつかある場合は、Azure Stack Hub 管理者に問い合わせてください。

    > [!NOTE]
    > `deployment suffix` 値は、デプロイのリソースが Azure 全体で一意の名前になることを保証するために使用されます。 これは、8 文字以内の文字と数字の一意の文字列である必要があります。

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    テキスト ファイルを保存し、パスをメモしておきます。

4. これで、Porter を使用してカメラ アプリをデプロイする準備ができました。 インストール コマンドを実行し、IoT Edge デプロイが作成される様子を確認します。

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. `https://<camera-ip>:3000/` でカメラ フィードを表示して、カメラのデプロイが完了したことを確認します。ここで、`<camara-ip>` はカメラの IP アドレスです。 この手順には最大で 10 分かかる場合があります。

## <a name="configure-azure-stream-analytics"></a>Azure Stream Analytics の構成

これでデータがカメラから Azure Stream Analytics に流れるようになりました。Power BI と通信するには、これを手動で承認する必要があります。

1. Azure portal から、 **[すべてのリソース]** を開き、*process-footfall\[yoursuffix\]* ジョブを開きます。

2. [Stream Analytics ジョブ] ウィンドウの **[ジョブ トポロジ]** セクションで、 **[出力]** オプションを選択します。

3. **traffic-output** 出力シンクを選択します。

4. **[承認の更新]** を選択して、自分の Power BI アカウントにサインインします。
  
    ![Power BI で承認プロンプトを更新する](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. 出力の設定を保存します。

6. **[概要]** ウィンドウにアクセスし、 **[開始]** を選択して Power BI へのデータの送信を開始します。

7. ジョブ出力の開始時刻に **[現在]** を選択し、 **[開始]** を選択します。 通知バーでジョブの状態を確認できます。

## <a name="create-a-power-bi-dashboard"></a>Power BI ダッシュボードを作成する

1. ジョブが成功したら [Power BI](https://powerbi.com/) に移動し、職場または学校アカウントを使用してサインインします。 Stream Analytics ジョブ クエリによる結果の出力が進行中の場合、作成した *footfall-dataset* データセットは **[データセット]** タブにあります。

2. Power BI ワークスペースで **[+ 作成]** を選択し、*Footfall Analysis* という名前の新しいダッシュボードを作成します。

3. ウィンドウの上部にある **[タイルの追加]** を選択します。 次に、 **[カスタム ストリーミング データ]** と **[次に]** を選択します。 **[データセット]** の下で **footfall-dataset** を選択します。 **[視覚化タイプ]** ドロップダウンで **[カード]** を選択し、**年齢** を **[フィールド]** に追加します。 **[次へ]** を選択してタイルに名前を入力し、 **[適用]** を選択してタイルを作成します。

4. 必要に応じて、フィールドとカードを追加できます。

## <a name="test-your-solution"></a>ソリューションをテストする

さまざまな人がカメラの前を歩いたときに、Power BI で作成したカードのデータがどのように変化するかを観察します。 推論は、記録されてから表示されるまで最大 20 秒かかることがあります。

## <a name="remove-your-solution"></a>ソリューションを削除する

ソリューションを削除する場合は、Porter を使用して、デプロイ用に作成したのと同じパラメーター ファイルを使用して次のコマンドを実行します。

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a>次のステップ

- [ハイブリッド アプリの設計上の考慮事項](overview-app-design-considerations.md)の詳細を確認する
- [GitHub でこのサンプルのコード](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis)を確認し、改善点を提案する
