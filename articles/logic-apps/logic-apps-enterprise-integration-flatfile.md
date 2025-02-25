---
title: フラット ファイルのエンコードまたはデコード
description: Enterprise Integration Pack を使用して、Azure Logic Apps でエンタープライズ統合用のフラット ファイルをエンコードまたはデコードする。
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, azla
ms.topic: how-to
ms.date: 11/02/2021
ms.openlocfilehash: 5956f8fb7a1de3baaac0a69ef9e5b63d445416eb
ms.sourcegitcommit: 677e8acc9a2e8b842e4aef4472599f9264e989e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/11/2021
ms.locfileid: "132327122"
---
# <a name="encode-and-decode-flat-files-in-azure-logic-apps"></a>Azure Logic Appsでファイル内のフラット ファイルをエンコードおよびデコードする。

企業間 (B2B) シナリオでビジネス パートナーに XML コンテンツを送信する前に、そのコンテンツをあらかじめエンコードすることが必要になる場合があります。 ロジック アプリ ワークフローを構築すると、[組み込み](../connectors/built-in.md#integration-account-built-in-actions)の **フラット ファイル** アクションを使用してフラット ファイルをエンコードおよびデコードできます。

**フラット ファイル** トリガーはありませんが、別のトリガーまたはアクションを使用して、さまざまなソースからの XML コンテンツをワークフローに取得またはフィードして、エンコードやデコードを行うことができます。 たとえば、要求トリガー、別のアプリ、または [Azure Logic Apps でサポートされている他のコネクタ](../connectors/apis-list.md)を使用できます。 ワークフローの **フラット ファイル** アクションには、[**ロジックアプリ (従量課金)** と **ロジック アプリ (Standard)** のリソースの種類](single-tenant-overview-compare.md)を使用できます。

> [!NOTE]
> **ロジック アプリ (Standard)** の場合、**フラット ファイル** アクションは現在プレビュー段階です。 

この記事では、フラット ファイルのエンコードおよびデコードのアクションを既存のロジック アプリ ワークフローに追加する方法について説明します。 ロジック アプリを初めて使用する場合は、次のドキュメントを参照してください。

* [Azure Logic Apps とは](logic-apps-overview.md)
* [Azure Logic Apps と Enterprise Integration Pack を使用した B2B エンタープライズ統合ワークフロー](logic-apps-enterprise-integration-overview.md)

## <a name="prerequisites"></a>前提条件

* Azure アカウントとサブスクリプション。 サブスクリプションをまだお持ちでない場合には、[無料の Azure アカウントにサインアップ](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)してください。

* エンタープライズ統合および B2B ワークフローで使用する成果物 (取引先、契約、証明書など) を定義して保存する[統合アカウント リソース](logic-apps-enterprise-integration-create-integration-account.md)。 このリソースでは、次の要件が満たされている必要があります。

  * ロジック アプリ リソースと同じ Azure サブスクリプションに関連付けられている。

  * ロジック アプリ リソースと同じ場所または Azure リージョンに存在する。

  * [**ロジック アプリ (従量課金)** リソース タイプ](logic-apps-overview.md#resource-type-and-host-environment-differences)を使用する場合、統合アカウントには以下のアイテムが必要です。

    * XML コンテンツのエンコードまたはデコードに使用するフラット ファイル [スキーマ](logic-apps-enterprise-integration-schemas.md)。

    * [ご利用のロジック アプリ リソースへのリンク](logic-apps-enterprise-integration-create-integration-account.md#link-account)。

  * [**ロジック アプリ (Standard)** リソース タイプ](logic-apps-overview.md#resource-type-and-host-environment-differences)を使用している場合は、統合アカウントにスキーマを保存しません。 代わりに、Azure portal または Visual Studio Code を使用して[ロジック アプリ リソースにスキーマを直接追加](logic-apps-enterprise-integration-schemas.md)することができます。 その後、"*同じロジック アプリ リソース*" 内の複数のワークフローでこれらのスキーマを使用できます。

    取引先、契約、証明書などの他の成果物を保存すると共に、[AS2](logic-apps-enterprise-integration-as2.md)、[X12](logic-apps-enterprise-integration-x12.md)、および [EDIFACT](logic-apps-enterprise-integration-edifact.md) 操作を使用するには、引き続き統合アカウントが必要です。 ただし、ロジック アプリ リソースを統合アカウントにリンクする必要はないため、リンク機能は存在しません。 統合アカウントは、ロジック アプリ リソースと同じ Azure サブスクリプションを使用することや、ロジック アプリ リソースと同じ場所に存在することなど、他の要件も満たす必要があります。

    > [!NOTE]
    > 現時点では、[RosettaNet](logic-apps-enterprise-integration-rosettanet.md) の操作をサポートしているのは、**ロジック アプリ (従量課金)** のリソースの種類のみです。 **ロジック アプリ (Standard)** というリソースの種類には、[RosettaNet](logic-apps-enterprise-integration-rosettanet.md) 操作は含まれていません。

* **フラット ファイル** アクションを使用する、空または既存のロジック アプリ ワークフロー。

  空のワークフローがある場合は、任意のトリガーを使用してワークフローを開始します。 この例では、要求トリガーを使用します。

## <a name="limits"></a>制限

生成するフラット ファイル スキーマに含まれる XML グループに、*1 より大きい値* に設定された`max count`プロパティの数が多すぎるのが存在しないことを確認してください。 `max count`プロパティ値が1より大きいXMLグループを`max count`プロパティが1より大きい別のXMLグループ内にネストしないでください。

フラット ファイル スキーマで次のフラグメントを選択できるたび、スキーマを解析する Azure Logic Apps エンジンによって *シンボル* が生成され、そのフラグメントの *予測* が生成されます。 スキーマで許可されるコンストラクトの数が 100,000 を超えるなど、このような構造が多すぎると、スキーマの拡張が過度に大きくなり、リソースと時間が多すぎます。

## <a name="add-flat-file-encoding-action"></a>フラット ファイルのエンコード アクションを追加する

### <a name="consumption"></a>[従量課金プラン](#tab/consumption)

1. [Azure portal](https://portal.azure.com) で、ロジック アプリ ワークフローをデザイナーで開きます。

1. トリガーがない空のワークフローがある場合は、必要なトリガーを追加します。 使用する場合は、次の手順に進みます。

   この例では、要求トリガーを使用します。これは、 **[HTTP 要求の受信時]** という名前で、ロジック アプリ ワークフローの外部からの受信要求を処理します。 要求トリガーを追加するには、次の手順を実行します。

   1. デザイナーの検索ボックスで、 **[組み込み]** を選択します。 デザイナーの検索ボックスに、「`HTTP request`」と入力します。

   1. トリガーの一覧から、 **[HTTP 要求の受信時]** という名前の要求トリガーを選択します。

   > [!TIP]
   > JSON スキーマの指定は省略可能です。 受信要求からのサンプルのペイロードがある場合は、 **[サンプルのペイロードを使用してスキーマを生成する]** を選択し、サンプルのペイロードを入力して、 **[完了]** を選択します。 **[要求本文の JSON スキーマ]** ボックスにスキーマが表示されます。

1. **フラット ファイルのエンコード** アクションを追加するワークフローのステップで、次のオプションを選択します。

   * ワークフローの最後に **フラット ファイルのエンコード** アクションを追加するには、 **[新しいステップ]** を選択します。

   * 既存のステップの間に **フラット ファイルのエンコード** アクションを追加するには、それらのステップを接続する矢印にポインターを合わせて、プラス記号 ( **+** ) を表示させます。 プラス記号を選択し、 **[アクションの追加]** を選択します。

1. **[操作の選択]** 検索ボックスに「`flat file`」と入力します。 アクションの一覧で、 **[フラット ファイルのエンコード]** という名前のアクションを選択します。

   ![Azure portal で従量課金プランのデザイナーの検索ボックスに "flat file"と表示され、[フラットファイルのエンコード] アクションが選択されている状態のスクリーンショット。](./media/logic-apps-enterprise-integration-flatfile/flat-file-encoding-consumption.png)

1. **[コンテンツ]** ボックス内をクリックして、動的コンテンツ リストを表示します。 一覧から、 **[HTTP 要求の受信時]** セクションで、 **[本文]** プロパティを選択します。これには、トリガーからの要求本文の出力とエンコードするコンテンツが含まれます。

   ![従量課金プランのデザイナーで "コンテンツ" プロパティに動的なコンテンツ リストが表示され、エンコード用のコンテンツが選択された状態のスクリーンショット。](./media/logic-apps-enterprise-integration-flatfile/select-content-to-encode-consumption.png)

   > [!TIP]
   > 動的コンテンツ リストに **[本文]** プロパティが表示されない場合は、 **[HTTP 要求の受信時]** セクションのラベルの横にある **[表示数を増やす]** を選択します。
   > デコードするコンテンツを **[コンテンツ]** ボックスに直接入力することもできます。

1. **[スキーマ名]** の一覧から、エンコードに使用するリンク済みの統合アカウント内にあるスキーマを選択します。

   ![従量課金プランのデザイナーで [スキーマ名] の一覧が開かれ、エンコードに使用するスキーマが選択された状態を示すスクリーンショット。](./media/logic-apps-enterprise-integration-flatfile/select-encoding-schema-consumption.png)

   > [!NOTE]
   > 一覧にスキーマが表示されない場合は、ご使用の統合アカウントに、エンコードに使用するスキーマ ファイルが含まれていません。 使用するスキーマをご使用の統合アカウントにアップロードしてください。

1. ワークフローを保存します。 デザイナーのツール バーで、 **[保存]** を選択します。

1. ワークフローをテストするには、HTTPS エンドポイントに要求を送信します。これは、要求トリガーの "**HTTP POST の URL**" プロパティに表示されます。要求の本文には、エンコードする XML コンテンツを含めます。

以上で、フラット ファイルのエンコード アクションの設定が完了しました。 実際のアプリでは、エンコードしたデータを Salesforce などの基幹業務 (LOB) アプリに保存することが必要になる場合があります。 または、そのエンコードしたデータを取引先に送信することもできます。 エンコード アクションの出力を Salesforce または取引先に送信するには、[Azure Logic Apps で使用可能な他のコネクタ](../connectors/apis-list.md)を使用します。

### <a name="standard"></a>[Standard](#tab/standard)

1. [Azure portal](https://portal.azure.com) で、ロジック アプリ ワークフローをデザイナーで開きます。

1. トリガーがない空のワークフローがある場合は、必要なトリガーを追加します。 使用する場合は、次の手順に進みます。

   この例では、要求トリガーを使用します。これは、 **[HTTP 要求の受信時]** という名前で、ロジック アプリ ワークフローの外部からの受信要求を処理します。 要求トリガーを追加するには、次の手順を実行します。

   1. デザイナーで、 **[操作を選択してください]** を選択します。 **[操作を選択してください]** ペインが開きます。検索ボックスの下にある **[組み込み]** を選択します。

   1. 検索ボックスに「 `HTTP request`」と入力します。 トリガーの一覧から、 **[HTTP 要求の受信時]** という名前の要求トリガーを選択します。

     > [!TIP]
     > JSON スキーマの指定は省略可能です。 受信要求からのサンプルのペイロードがある場合は、 **[サンプルのペイロードを使用してスキーマを生成する]** を選択し、サンプルのペイロードを入力して、 **[完了]** を選択します。 **[要求本文の JSON スキーマ]** ボックスにスキーマが表示されます。

1. **フラット ファイルのエンコード** アクションを追加するワークフローのステップで、次のオプションを選択します。

   * ワークフローの最後に **フラット ファイルのエンコード** アクションを追加するには、プラス記号 ( **+** ) を選択し、 **[アクションの追加]** を選択します。

   * 既存のステップの間に **フラット ファイルのエンコード** アクションを追加するには、それらのステップの間に表示されるプラス記号 ( **+** ) を選択し、 **[Insert a new step]\(新しいステップを挿入\)** を選択します。

1. **[操作を選択してください]** ペインが開きます。検索ボックスの下にある **[組み込み]** を選択します。

1. 検索ボックスに「 `flat file`」と入力します。 アクションの一覧で、 **[フラット ファイルのエンコード]** という名前のアクションを選択します。

   ![Azure portal で Standard ワークフロー デザイナーの検索ボックスに "flat file"と表示され、[フラットファイルのエンコード] アクションが選択されている状態のスクリーンショット。](./media/logic-apps-enterprise-integration-flatfile/flat-file-encoding-standard.png)

1. **[コンテンツ]** ボックス内をクリックして、動的コンテンツ リストを表示します。 一覧から、 **[HTTP 要求の受信時]** セクションで、 **[本文]** プロパティを選択します。これには、トリガーからの要求本文の出力とエンコードするコンテンツが含まれます。

   ![Standard ワークフロー デザイナーで "コンテンツ" プロパティに動的なコンテンツ リストが表示され、エンコード用のコンテンツが選択された状態のスクリーンショット。](./media/logic-apps-enterprise-integration-flatfile/select-content-to-encode-standard.png)

   > [!TIP]
   > 動的コンテンツ リストに **[本文]** プロパティが表示されない場合は、 **[HTTP 要求の受信時]** セクションのラベルの横にある **[表示数を増やす]** を選択します。
   > エンコードするコンテンツを **[コンテンツ]** ボックスに直接入力することもできます。

1. **[名前]** ボックスの一覧から、エンコード用に以前にロジック アプリ リソースにアップロードしたスキーマを選択します。次に例を示します。

   ![Standard ワークフロー デザイナーで [名前] の一覧が開かれ、エンコードに使用するスキーマが選択された状態を示すスクリーンショット。](./media/logic-apps-enterprise-integration-flatfile/select-encoding-schema-standard.png)

   > [!NOTE]
   > 一覧にスキーマが表示されない場合は、Standard ロジック アプリのリソースに、エンコードに使用するスキーマ ファイルが含まれていません。 [使用するスキーマを Standard ロジック アプリのリソースにアップロード](logic-apps-enterprise-integration-schemas.md)する方法をご覧ください。

1. ワークフローを保存します。 デザイナーのツール バーで、 **[保存]** を選択します。

1. ワークフローをテストするには、HTTPS エンドポイントに要求を送信します。これは、要求トリガーの "**HTTP POST の URL**" プロパティに表示されます。要求の本文には、エンコードする XML コンテンツを含めます。

以上で、フラット ファイルのエンコード アクションの設定が完了しました。 実際のアプリでは、エンコードしたデータを Salesforce などの基幹業務 (LOB) アプリに保存することが必要になる場合があります。 または、そのエンコードしたデータを取引先に送信することもできます。 エンコード アクションの出力を Salesforce または取引先に送信するには、[Azure Logic Apps で使用可能な他のコネクタ](../connectors/apis-list.md)を使用します。

---

## <a name="add-flat-file-decoding-action"></a>フラット ファイルのデコード アクションを追加する

### <a name="consumption"></a>[従量課金プラン](#tab/consumption)

1. [Azure portal](https://portal.azure.com) で、ロジック アプリ ワークフローをデザイナーで開きます。

1. トリガーがない空のワークフローがある場合は、必要なトリガーを追加します。 使用する場合は、次の手順に進みます。

   この例では、要求トリガーを使用します。これは、 **[HTTP 要求の受信時]** という名前で、ロジック アプリ ワークフローの外部からの受信要求を処理します。 要求トリガーを追加するには、次の手順を実行します。

   1. デザイナーの検索ボックスで、 **[組み込み]** を選択します。 デザイナーの検索ボックスに、「`HTTP request`」と入力します。

   1. トリガーの一覧から、 **[HTTP 要求の受信時]** という名前の要求トリガーを選択します。

   > [!TIP]
   > JSON スキーマの指定は省略可能です。 受信要求からのサンプルのペイロードがある場合は、 **[サンプルのペイロードを使用してスキーマを生成する]** を選択し、サンプルのペイロードを入力して、 **[完了]** を選択します。 **[要求本文の JSON スキーマ]** ボックスにスキーマが表示されます。

1. **フラット ファイルのデコード** アクションを追加するワークフローのステップで、次のオプションを選択します。

   * ワークフローの最後に **フラット ファイルのデコード** アクションを追加するには、 **[新しいステップ]** を選択します。

   * 既存のステップの間に **フラット ファイルのデコード** アクションを追加するには、それらのステップを接続する矢印にポインターを合わせて、プラス記号 ( **+** ) を表示させます。 プラス記号を選択し、 **[アクションの追加]** を選択します。

1. **[操作の選択]** 検索ボックスに「`flat file`」と入力します。 アクションの一覧で、 **[フラット ファイルのデコード]** という名前のアクションを選択します。

   ![Azure portal で従量課金プランのデザイナーの検索ボックスに "flat file"と表示され、[フラットファイルのデコード] アクションが選択されている状態のスクリーンショット。](./media/logic-apps-enterprise-integration-flatfile/flat-file-decoding-consumption.png)

1. **[コンテンツ]** ボックス内をクリックして、動的コンテンツ リストを表示します。 一覧から、 **[HTTP 要求の受信時]** セクションで、 **[本文]** プロパティを選択します。これには、トリガーからの要求本文の出力とデコードするコンテンツが含まれます。

   ![従量課金プランのデザイナーで "コンテンツ" プロパティに動的なコンテンツ リストが表示され、デコード用のコンテンツが選択された状態のスクリーンショット。](./media/logic-apps-enterprise-integration-flatfile/select-content-to-decode-consumption.png)

   > [!TIP]
   > 動的コンテンツ リストに **[本文]** プロパティが表示されない場合は、 **[HTTP 要求の受信時]** セクションのラベルの横にある **[表示数を増やす]** を選択します。 デコードするコンテンツを **[コンテンツ]** ボックスに直接入力することもできます。

1. **[スキーマ名]** の一覧から、デコードに使用するリンク済みの統合アカウント内にあるスキーマを選択します。

   ![従量課金プランのデザイナーで [スキーマ名] の一覧が開かれ、デコードに使用するスキーマが選択された状態を示すスクリーンショット。](./media/logic-apps-enterprise-integration-flatfile/select-decoding-schema-consumption.png)

   > [!NOTE]
   > 一覧にスキーマが表示されない場合は、ご使用の統合アカウントに、デコードに使用するスキーマ ファイルが含まれていません。 使用するスキーマをご使用の統合アカウントにアップロードしてください。

1. ワークフローを保存します。 デザイナーのツール バーで、 **[保存]** を選択します。

1. ワークフローをテストするには、HTTPS エンドポイントに要求を送信します。これは、要求トリガーの "**HTTP POST の URL**" プロパティに表示されます。要求の本文には、デコードする XML コンテンツを含めます。

以上で、フラット ファイルのデコード アクションの設定が完了しました。 実際のアプリでは、デコードしたデータを Salesforce などの基幹業務 (LOB) アプリに保存することが必要になる場合があります。 または、そのデコードしたデータを取引先に送信することもできます。 デコード アクションの出力を Salesforce または取引先に送信するには、[Azure Logic Apps で使用可能な他のコネクタ](../connectors/apis-list.md)を使用します。

### <a name="standard"></a>[Standard](#tab/standard)

1. [Azure portal](https://portal.azure.com) で、ロジック アプリ ワークフローをデザイナーで開きます。

1. トリガーがない空のワークフローがある場合は、必要なトリガーを追加します。 使用する場合は、次の手順に進みます。

   この例では、要求トリガーを使用します。これは、 **[HTTP 要求の受信時]** という名前で、ロジック アプリ ワークフローの外部からの受信要求を処理します。 要求トリガーを追加するには、次の手順を実行します。

   1. デザイナーで、 **[操作を選択してください]** を選択します。 **[操作を選択してください]** ペインが開きます。検索ボックスの下にある **[組み込み]** を選択します。

   1. 検索ボックスに「 `HTTP request`」と入力します。 トリガーの一覧から、 **[HTTP 要求の受信時]** という名前の要求トリガーを選択します。

     > [!TIP]
     > JSON スキーマの指定は省略可能です。 受信要求からのサンプルのペイロードがある場合は、 **[サンプルのペイロードを使用してスキーマを生成する]** を選択し、サンプルのペイロードを入力して、 **[完了]** を選択します。 **[要求本文の JSON スキーマ]** ボックスにスキーマが表示されます。

1. **フラット ファイルのデコード** アクションを追加するワークフローのステップで、次のオプションを選択します。

   * ワークフローの最後に **フラット ファイルのデコード** アクションを追加するには、プラス記号 ( **+** ) を選択し、 **[アクションの追加]** を選択します。

   * 既存のステップの間に **フラット ファイルのデコード** アクションを追加するには、それらのステップの間に表示されるプラス記号 ( **+** ) を選択し、 **[Insert a new step]\(新しいステップを挿入\)** を選択します。

1. **[操作を選択してください]** ペインが開きます。検索ボックスの下にある **[組み込み]** を選択します。

1. 検索ボックスに「 `flat file`」と入力します。 アクションの一覧で、 **[フラット ファイルのデコード]** という名前のアクションを選択します。

   ![Azure portal で Standard ワークフロー デザイナーの検索ボックスに "flat file"と表示され、[フラットファイルのデコード] アクションが選択されている状態のスクリーンショット。](./media/logic-apps-enterprise-integration-flatfile/flat-file-decoding-standard.png)

1. **[コンテンツ]** ボックス内をクリックして、動的コンテンツ リストを表示します。 一覧から、 **[HTTP 要求の受信時]** セクションで、 **[本文]** プロパティを選択します。これには、トリガーからの要求本文の出力とデコードするコンテンツが含まれます。

   ![Standard ワークフロー デザイナーで "コンテンツ" プロパティに動的なコンテンツ リストが表示され、デコード用のコンテンツが選択された状態のスクリーンショット。](./media/logic-apps-enterprise-integration-flatfile/select-content-to-decode-standard.png)

   > [!TIP]
   > 動的コンテンツ リストに **[本文]** プロパティが表示されない場合は、 **[HTTP 要求の受信時]** セクションのラベルの横にある **[表示数を増やす]** を選択します。
   > デコードするコンテンツを **[コンテンツ]** ボックスに直接入力することもできます。

1. **[名前]** ボックスの一覧から、デコード用に以前にロジック アプリ リソースにアップロードしたスキーマを選択します。次に例を示します。

   ![Standard ワークフロー デザイナーで [名前] の一覧が開かれ、デコードに使用するスキーマが選択された状態を示すスクリーンショット。](./media/logic-apps-enterprise-integration-flatfile/select-decoding-schema-standard.png)

   > [!NOTE]
   > 一覧にスキーマが表示されない場合は、Standard ロジック アプリのリソースに、デコードに使用するスキーマ ファイルが含まれていません。 [使用するスキーマを Standard ロジック アプリのリソースにアップロード](logic-apps-enterprise-integration-schemas.md)する方法をご覧ください。

1. ワークフローを保存します。 デザイナーのツール バーで、 **[保存]** を選択します。

1. ワークフローをテストするには、HTTPS エンドポイントに要求を送信します。これは、要求トリガーの "**HTTP POST の URL**" プロパティに表示されます。要求の本文には、デコードする XML コンテンツを含めます。

以上で、フラット ファイルのデコード アクションの設定が完了しました。 実際のアプリでは、デコードしたデータを Salesforce などの基幹業務 (LOB) アプリに保存することが必要になる場合があります。 または、そのデコードしたデータを取引先に送信することもできます。 デコード アクションの出力を Salesforce または取引先に送信するには、[Azure Logic Apps で使用可能な他のコネクタ](../connectors/apis-list.md)を使用します。

---

## <a name="next-steps"></a>次のステップ

* [Enterprise Integration Pack](logic-apps-enterprise-integration-overview.md) について詳細を確認する
