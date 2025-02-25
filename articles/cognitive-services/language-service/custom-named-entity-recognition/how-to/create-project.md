---
title: カスタム NER で Azure リソースを使用する
titleSuffix: Azure Cognitive Services
description: カスタム NER で Azure リソースを使用する手順について説明します。
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-service
ms.topic: conceptual
ms.date: 11/02/2021
ms.author: aahi
ms.custom: language-service-custom-ner, references_regions, ignite-fall-2021
ms.openlocfilehash: 97b4a527fe049f4a36839ed38b1498307ea30d36
ms.sourcegitcommit: 0415f4d064530e0d7799fe295f1d8dc003f17202
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/17/2021
ms.locfileid: "132705077"
---
# <a name="how-to-create-custom-ner-projects"></a>カスタム NER プロジェクトを作成する方法

カスタム NER の使用を開始する前に、いくつかのものが必要です。

* Azure 言語リソース 
* `.txt` ファイルをアップロードする Azure ストレージ アカウント。このファイルは、テキストを分類する AI モデルをトレーニングするために使用します

この記事を使用して、カスタム NER を使用するための要件を準備する方法について学習してください。

## <a name="prerequisites"></a>前提条件

Azure サブスクリプション。[無料で作成できます](https://azure.microsoft.com/free/cognitive-services)。
データに使用する[プロジェクト スキーマ](design-schema.md)を把握しておく必要があります。

この記事を使用して、カスタム テキスト分類を使用するための要件を準備する方法について学習してください。

## <a name="azure-resources"></a>Azure リソース

カスタム NER の使用を開始する前に、Azure 言語リソースが必要になります。 Azure portal でこれを作成する場合、[クイックスタート](../quickstart.md)の手順をお勧めします。 Azure portal でリソースを作成すると、必要なすべてのアクセス許可が事前に構成された状態で、Azure ストレージ アカウントを同時に作成できます。 また、記事を読み進めると、既存のリソースを使用する方法について学習でき、カスタム NER と連携するようにリソースを構成できます。

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

[!INCLUDE [create a new resource from the Azure portal](../../custom-classification/includes/resource-creation-azure-portal.md)]

# <a name="language-studio"></a>[Language Studio](#tab/studio)

### <a name="create-a-new-resource-from-language-studio"></a>Language Studio から新しいリソースを作成する

初めてログインすると、[Language Studio](https://aka.ms/languageStudio) でウィンドウが表示され、そのウィンドウで言語リソースを選ぶことや、新しい言語リソースを作成することができます。 また、右上隅にある設定アイコンをクリックし、 **[リソース]** を選択し、 **[新しいリソースを作成する]** をクリックしてリソースを作成することもできます。

> [!IMPORTANT]
> * カスタム NER を使用するには、Standard (**S**) 価格レベルで **米国西部 2** または **西ヨーロッパ** の言語リソースが必要になります。
> * リソースを作成するときは、必ず **[マネージド ID]** を選択してください。 

:::image type="content" source="../../media/create-new-resource-small.png" alt-text="Language Studio のリソース作成画面のスクリーンショット。" lightbox="../../media/create-new-resource.png":::

カスタム NER を使用するには、[Azure ストレージ アカウントを作成](../../../../storage/common/storage-account-create.md)する必要があります (このアカウントをまだ持っていない場合)。 

次に、ストレージ アカウントに[適切なロール](#roles-for-your-storage-account)を割り当てて、言語リソースに接続する必要があります。 

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

### <a name="create-a-new-resource-with-the-azure-powershell"></a>Azure PowerShell を使用して新しいリソースを作成する

新しいリソースとストレージ アカウントは、GitHub でホストされる次の CLI [テンプレート](https://github.com/Azure-Samples/cognitive-services-sample-data-files)と[パラメーター](https://github.com/Azure-Samples/cognitive-services-sample-data-files) ファイルを使用して作成できます。

パラメーター ファイルで次の値を編集します。

| パラメーター名 | 値の説明 |
|--|--|
|`name`| 言語リソースの名前|
|`location`| リソースがホストされているリージョン。 カスタム NER は、**米国西部 2** と **西ヨーロッパ** でのみ使用できます。|
|`sku`| リソースの価格レベル。 この機能は、**S** レベルでのみ動作します|
|`storageResourceName`| ストレージ アカウントの名前|
|`storageLocation`| ストレージ アカウントがホストされているリージョン。|
|`storageSkuType`| [ストレージ アカウント](/rest/api/storagerp/srp_sku_types)の SKU。|
|`storageResourceGroupName`| ストレージ アカウントのリソース グループ|
<!-- |builtInRoleType| このロールを **"Contributor"** に設定します| -->

次の PowerShell コマンドを使用して、編集したファイルと共に Azure Resource Manager (ARM) テンプレートをデプロイします。

```powershell
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateFile <path-to-arm-template> `
  -TemplateParameterFile <path-to-parameters-file>
```

[テンプレートのデプロイ](../../../../azure-resource-manager/templates/deploy-powershell.md#parameter-files)と[パラメーター ファイル](../../../../azure-resource-manager/templates/parameter-files.md?tabs=json)について詳しくは、ARM テンプレートのドキュメントを参照してください。

--- 

## <a name="using-a-pre-existing-azure-resource"></a>既存の Azure リソースの使用

既存の言語リソースを使用してカスタム NER の使用を開始できますが、このリソースが以下の要件を満たしている必要があります。

|要件  |説明  |
|---------|---------|
|リージョン     | サポートされている 2 つのリージョン (**米国西部 2** または **西ヨーロッパ**) のいずれかで既存のリソースがプロビジョニングされていることを確認します。 これが行われていない場合、これらのリージョンで新しいリソースを作成する必要があります。        |
|Pricing tier     | 既存のリソースが Standard (**S**) 価格レベルであることを確認します。 この価格レベルのみサポートされています。 リソースがこの価格レベルではない場合、新しいリソースを作成する必要があります。        |
|マネージド ID     | リソースのマネージド ID 設定が有効になっていることを確認します。 これが行われていない場合、次のセクションを参照してください。 |

カスタム NER を使用するには、[Azure ストレージ アカウントを作成](../../../../storage/common/storage-account-create.md)する必要があります (このアカウントをまだ持っていない場合)。 

次に、ストレージ アカウントに[適切なロール](#roles-for-your-storage-account)を割り当てて、言語リソースに接続する必要があります。 

## <a name="roles-for-your-azure-language-resource"></a>Azure 言語リソースのロール

**所有者** または **共同作成者** のロールが Azure 言語リソースに割り当てられている必要があります。

## <a name="enable-identity-management-for-your-resource"></a>リソースの ID 管理を有効にする

言語リソースには ID 管理が必要です。これは、Azure portal または Language Studio のいずれかを使用して有効にすることができます。 [Language Studio](https://aka.ms/languageStudio) を使用して有効にするには:
1. 画面の右上隅の設定アイコンをクリックします
2. **[リソース]** を選択します
3. Azure リソースの **[マネージド ID]** を選択します。

## <a name="roles-for-your-storage-account"></a>ストレージ アカウントのロール

Azure BLOB ストレージ アカウントには、次のロールが必要です。

* リソースには、ストレージ アカウントの **所有者** または **共同作成者** のロールが必要です。
* リソースには、ストレージ アカウントの **ストレージ BLOB データ所有者** または **ストレージ BLOB データ共同作成者** のロールが必要です。
* リソースには、ストレージ アカウントの **閲覧者** ロールが必要です。

ストレージ アカウントに適切なロールを設定するには、次のようにします。

1. [Azure portal](https://ms.portal.azure.com/) でストレージ アカウントのページに移動します。
2. 左側のナビゲーション メニューで **[アクセス制御 (IAM)]** を選択します。
3. **[追加]** を選択して **[ロールの割り当ての追加]** を行い、 **[所有者]** または **[共同作成者]** ロールを選択します。 **[選択]** フィールドでユーザー名を検索できます。

[!INCLUDE [Storage connection note](../../custom-classification/includes/storage-account-note.md)]

## <a name="prepare-training-data"></a>トレーニング データの準備

* カスタム NER プロジェクトを作成するための前提条件として、トレーニング データをストレージ アカウントの BLOB コンテナーにアップロードする必要があります。 トレーニング ファイルの作成とアップロードは、Azure から直接行うことも、Azure Storage Explorer ツールを使用して行うこともできます。 Azure Storage Explorer ツールを使用すると、より多くのデータを短い時間でアップロードできます。

  * [Azure からのファイルの作成とアップロード](../../../../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container)
  * [Azure Storage Explorer を使用したファイルの作成とアップロード](../../../../vs-azure-tools-storage-explorer-blobs.md)

* カスタム NER に使用できるのは、`.txt` ファイルのみです 。 データが別の形式の場合は、[Cognitive Services 言語ユーティリティ ツール](https://aka.ms/CognitiveServicesLanguageUtilities)を使用して、ファイルを `.txt` フォーマットに解析できます。

* タグ付きデータをアップロードすること、または Language Studio でデータにタグを付けることができます。 タグ付きデータは、[タグ ファイル形式](../concepts/data-formats.md)に従っている必要があります。 

>[!TIP]
> データの選択と準備について詳しくは、[データの準備とスキーマの定義](../how-to/design-schema.md)に関するページを参照してください。

## <a name="create-a-custom-named-entity-recognition-project"></a>カスタム固有表現認識プロジェクトを作成する

>[!NOTE]
> データセットに異なる言語のファイルが含まれる場合、または実行時に異なる言語が想定される場合は、多言語オプションを有効にすることができます。

[!INCLUDE [Create custom NER project](../includes/create-project.md)]

## <a name="next-steps"></a>次のステップ

プロジェクトが作成された後、[データへのタグ付け](tag-data.md)を開始できます。これにより、エンティティ抽出モデルにテキストを解釈する方法が伝えられ、トレーニングと評価に使用されます。