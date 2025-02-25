---
title: 画像ラベル付けプロジェクトを設定する
titleSuffix: Azure Machine Learning
description: データのラベル付けツールを使用して画像にラベルを付けるプロジェクトを作成します。 支援付き機械学習ラベル付け、つまり人間参加型 (Human in the loop) のラベル付けを有効にして、このタスクを補助します。
author: sdgilley
ms.author: sgilley
ms.service: machine-learning
ms.subservice: mldata
ms.topic: how-to
ms.date: 10/21/2021
ms.custom: data4ml, ignite-fall-2021
ms.openlocfilehash: b1fc133b02c667ae46d543da3fe020d116d6eb54
ms.sourcegitcommit: 106f5c9fa5c6d3498dd1cfe63181a7ed4125ae6d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/02/2021
ms.locfileid: "131068895"
---
# <a name="create-an-image-labeling-project-and-export-labels"></a>画像ラベル付けプロジェクトを作成してラベルをエクスポートする

Azure Machine Learning で画像にラベルを付けるデータのラベル付けプロジェクトを作成して実行する方法について説明します。  このタスクを補助するために、支援付き機械学習データ ラベル付け、つまり人間参加型 (Human in the loop) のラベル付けを使用します。

分類、物体検出 (境界ボックス)、インスタンスのセグメント化 (ポリゴン) を行うためのラベルを設定します。

データのラベル付けツールを使用して、[テキスト ラベル付けプロジェクトを作成する](how-to-create-text-labeling-projects.md)こともできます。

## <a name="image-labeling-capabilities"></a>画像のラベル付け機能

Azure Machine Learning のデータのラベル付け機能を通じて、以下のように、データのラベル付けプロジェクトの作成、管理、監視を一元的に行うことができます。

- データ、ラベル、チーム メンバーを調整して、ラベル付けタスクを効率的に管理する。
- 進行状況を追跡し、未完了のラベル付けタスクのキューを維持する。
- プロジェクトを開始および停止し、ラベル付けの進行状況を制御する。
- ラベル付きのデータを確認し、Azure Machine Learning データセットとしてエクスポートする。

> [!Important]
> データ イメージは、Azure BLOB データストアで利用可能なファイルである必要があります。 (既存のデータストアがない場合は、プロジェクトの作成時にファイルをアップロードできます)。

画像データには、".jpg"、".jpeg"、".png"、".jpe"、".jfif"、".bmp"、".tif"、".tiff"、".dcm"、".dicom" のいずれかの種類のファイルを使用できます。 各ファイルはラベル付けされる項目です。

## <a name="prerequisites"></a>前提条件

[!INCLUDE [prerequisites](../../includes/machine-learning-data-labeling-prerequisites.md)]

## <a name="create-an-image-labeling-project"></a>イメージのラベル付けプロジェクトを作成する

[!INCLUDE [start](../../includes/machine-learning-data-labeling-start.md)]

1. プロジェクトを作成するには、 **[プロジェクトの追加]** を選択します。 プロジェクトに適切な名前を付けます。 プロジェクトが将来削除されても、プロジェクト名を再利用することはできません。

1. **[画像]** を選択して、画像ラベル付けプロジェクトを作成します。

    :::image type="content" source="media/how-to-create-labeling-projects/labeling-creation-wizard.png" alt-text="画像のラベル付けのためのラベル付けプロジェクトの作成":::

    * ラベルのセットから "*1 つのラベル*" だけを画像に適用する場合は、プロジェクトに対して **[イメージ分類の複数クラス]** を選択します。
    * ラベルのセットから "*1 つまたは複数*" のラベルを画像に適用する場合は、プロジェクトに対して **[Image Classification Multi-label]\(画像分類の複数ラベル\)** を選択します。 たとえば、犬の写真には "*犬*" と "*日中*" の両方のラベルが付けられる可能性があります。
    * 画像内の各オブジェクトにラベルと境界ボックスを割り当てる場合は、プロジェクトに対して **[オブジェクト ID (四角形領域)]** を選択します。
    * 画像内の各オブジェクトにラベルを割り当て、その周囲にポリゴンを描画する場合は、プロジェクトに対して **[インスタンスのセグメント化 (ポリゴン)]** を選択します。

1. 続行する準備ができたら、 **[次へ]** を選択します。

## <a name="add-workforce-optional"></a>従業員を追加する (省略可能)

[!INCLUDE [outsource](../../includes/machine-learning-data-labeling-outsource.md)]

## <a name="specify-the-data-to-label"></a>ラベル付けの対象データを指定する

対象のデータが含まれるデータセットを既に作成済みの場合は、 **[既存のデータセットを選択します]** ボックスの一覧から選択します。 または、 **[Create a dataset]\(データセットの作成\)** を選択して、既存の Azure データストアを選択するか、ローカル ファイルをアップロードします。

> [!NOTE]
> 1 つのプロジェクトに 50 万個を超えるファイルを含めることはできません。  データセットにこれを超えるファイルがある場合は、最初の 50 万個だけが読み込まれます。  

### <a name="create-a-dataset-from-an-azure-datastore"></a>Azure データストアからデータセットを作成する

多くの場合は、ローカル ファイルをアップロードするだけでかまいません。 ただし、[Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/) には、大量のデータをより高速かつ堅牢に転送する方法が用意されています。 既定のファイル移動方法としては、Storage Explorer をお勧めします。

Azure Blob Storage に既に格納済みのデータからデータセットを作成するには:

1. **[Create a dataset]\(データセットの作成\)**  >  **[From datastore]\(データストアから\)** の順に選択します。
1. データセットに **名前** を割り当てます。
1. **[データセットの種類]** がファイルに設定されている場合、イメージのデータセットの種類として使用できるのはファイルのみになります。
1. データストアを選択します。
1. データがお使いの Blob Storage 内のサブフォルダーにある場合は、 **[参照]** を選択してパスを選択します。
    * 選択したパスのサブフォルダー内にあるすべてのファイルを含めるには、パスの末尾に "/**" を追加します。
    * 現在のコンテナーとそのサブフォルダー内にあるすべてのデータを含めるには、"* */* .*" を追加します。
1. (省略可能) データセットの説明を入力します。
1. **[次へ]** を選択します。
1. 詳細を確認します。 **[戻る]** を選択して設定を変更するか、 **[作成]** を選択してデータセットを作成します。

### <a name="create-a-dataset-from-uploaded-data"></a>アップロードしたデータからデータセットを作成する

データを直接アップロードするには:

1. **[Create a dataset]\(データセットの作成\)**  >  **[From local files]\(ローカル ファイルから\)** の順に選択します。
1. データセットに **名前** を割り当てます。
1. **[データセットの種類]** がファイルに設定されている場合、イメージのデータセットの種類として使用できるのはファイルのみになります。
1. (省略可能) データセットの説明を入力します。
1. **[次へ]** を選択します。
1. (省略可能) データストアを選択または作成します。 または、既定のまま、Machine Learning ワークスペースの既定の BLOB ストア ("workspaceblobstore") にアップロードします。
1. **[参照]** を選択し、アップロードするローカル ファイルまたはフォルダーを選択します。
1. **[次へ]** を選択します。
1. 詳細を確認します。 **[戻る]** を選択して設定を変更するか、 **[作成]** を選択してデータセットを作成します。

## <a name="configure-incremental-refresh"></a><a name="incremental-refresh"> </a> 増分更新を構成する

[!INCLUDE [refresh](../../includes/machine-learning-data-labeling-refresh.md)]

## <a name="specify-label-classes"></a>ラベル クラスを指定する

[!INCLUDE [classes](../../includes/machine-learning-data-labeling-classes.md)]

## <a name="describe-the-image-labeling-task"></a>画像のラベル付けタスクを説明する

[!INCLUDE [describe](../../includes/machine-learning-data-labeling-describe.md)]

境界ボックスについては、次のような重要な質問があります。

* このタスクでは境界ボックスはどのように定義されていますか。 完全にオブジェクトの内側に配置すべきですか、それとも外側に配置するべきですか。 できるだけ近くなるようにトリミングする必要がありますか、それともある程度の余白は許容されますか。
* 境界ボックスを定義する際にラベラーがどの程度の注意と一貫性を適用することを期待しますか。
* 画像内に一部しか表示されていないオブジェクトにラベル付けするにはどうすればよいですか。 
* 他のオブジェクトによって部分的に覆い隠されているオブジェクトにラベル付けするにはどうすればよいですか。

>[!NOTE]
> ラベラーは最初の 9 つのラベルを 1 から 9 の数字キーを使用して選択できることを必ず明記してください。

## <a name="use-ml-assisted-data-labeling"></a>支援付き機械学習データ ラベル付けを使用する

**[ML によるラベル付け]** ページでは、自動機械学習モデルをトリガーして、ラベル付けタスクを高速化することができます。 医療画像 (".dcm") は、支援付き補助ラベル付けには含まれません。

ラベル付けプロジェクトの初めに、潜在的な偏りを減らすために、項目がランダムな順序にシャッフルされます。 ただし、データセットに偏りが存在すれば、それがトレーニング済みのモデルに反映されます。 たとえば、項目の 80% が単一のクラスに属している場合、モデルのトレーニングに使用されるデータの約 80% は、そのクラスに属することになります。

支援付きラベル付けを有効にするには、 *[ML Assist によるラベル付けを有効にする]* を選んで、GPU を指定します。 GPU クラスターがお使いのワークスペースにない場合は、自動的に作成されて、ワークスペースに追加されます。   クラスターは最小限の 0 ノードで作成されるので、使用されていないときはコストがかかりません。


支援付き機械学習ラベル付けは、2 つのフェーズで構成されます。

* クラスタリング
* 事前ラベル付け

ML によるラベル付けを開始するために必要なラベル付けされたデータの正確な数は固定数ではありません。  これは、ラベル付けプロジェクトごとに大きく異なる場合があります。 プロジェクトによっては、300 個の項目に手動でラベル付けした後に、事前ラベル付けまたはクラスターのタスクが可能になる場合があります。 ML によるラベル付けでは、"*転移学習*" と呼ばれる手法が使用されます。この手法では、事前トレーニング済みモデルを使用してトレーニング プロセスを開始します。 対象となるデータセットのクラスが、事前トレーニング済みモデル内のクラスと似ている場合、手動でラベル付けされた項目が数百個あれば、事前ラベルを利用できる可能性があります。 対象となるデータセットが、モデルの事前トレーニングに使用されたデータと大きく異なる場合には、さらに時間がかかる可能性があります。

最終的なラベルは依然としてラベラーからの入力に依存するため、この技術は "*人間参加 (Human in the loop) 型*" のラベル付けと呼ばれることがあります。

> [!NOTE]
> ML によるデータのラベル付けでは、[仮想ネットワーク](how-to-network-security-overview.md)の内側でセキュリティ保護された既定のストレージ アカウントはサポートされません。 ML によるデータのラベル付けには、既定以外のストレージ アカウントを使用する必要があります。 既定以外のストレージ アカウントは、仮想ネットワークの背後でセキュリティ保護できます。

### <a name="clustering"></a>クラスタリング

一定数のラベルが送信されると、分類の機械学習モデルは、類似する項目をグループ化し始めます。  手動でのタグ付けにかかる時間を短縮するために、これらの類似する画像は同じ画面上でラベラーに表示されます。 ラベラーに表示される画像が 4 枚、6 枚、9 枚のいずれかのグリッドであるとき、クラスタリングは特に効果的です。

手動でラベル付けされたデータで機械学習モデルのトレーニングが完了すると、モデルは切り詰められて、その最後の全結合層に到達します。 ラベル付けされていない画像は、その後、切り詰められたモデルを介し、一般に "埋め込み" や "特徴付け" として知られるプロセスに渡されます。 このようにそれぞれの画像が、このモデル レイヤーによって定義された高次元空間に埋め込まれます。 この空間内でニアレストネイバーとなった画像が、クラスタリング タスクに使用されます。 

クラスタリング フェーズは、オブジェクト検出モデルまたはテキスト分類には表示されません。

### <a name="prelabeling"></a>事前ラベル付け

十分な数のラベルが送信されると、分類モデルを使用してタグが予測されます。 または、オブジェクト検出モデルを使用して境界ボックスが予測されます。 ラベラーには、各項目に既に存在する予測済みのラベルを含むページが表示されるようになります。 オブジェクト検出では、予測されたボックスも表示されます。 その後、このタスクでは、これらの予測をレビューし、間違ってラベル付けされた画像を修正してから、ページを送信します。  

手動でラベル付けされたデータで機械学習モデルのトレーニングが完了すると、手動でラベル付けされた項目のテスト セットでモデルが評価され、さまざまな信頼度のしきい値でその精度が判定されます。 事前ラベルを表示するだけの精度がモデルにあるかどうかの基準となる信頼度のしきい値は、この評価プロセスを使用して割り出されます。 その後、このモデルは、ラベル付けされていないデータに対して評価されます。 予測の信頼度がこのしきい値を超える項目が、事前ラベル付けに使用されます。

## <a name="initialize-the-image-labeling-project"></a>画像のラベル付けプロジェクトを初期化する

[!INCLUDE [initialize](../../includes/machine-learning-data-labeling-initialize.md)]

## <a name="run-and-monitor-the-project"></a>プロジェクトを実行して監視する

[!INCLUDE [run](../../includes/machine-learning-data-labeling-run.md)]

### <a name="dashboard"></a>ダッシュボード

**[ダッシュボード]** タブに、ラベル付けタスクの進行状況が表示されます。

:::image type="content" source="./media/how-to-create-labeling-projects/labeling-dashboard.png" alt-text="データのラベル付けダッシュボード":::

進行状況グラフには、ラベル付けされた項目、スキップされた項目、レビューが必要な項目、またはまだそれが行われていない項目の数が表示されます。  グラフの上にカーソルを合わせると、各セクションの項目数が表示されます。

中央のセクションには、これから割り当てられるタスクのキューが表示されます。 ML によるラベル付けが無効になっている場合、このセクションには割り当てられる手動タスクの数が表示されます。 ML によるラベル付けがオンになっている場合は、次の内容が表示されます。

* キュー内にクラスター化された項目を含むタスク
* キュー内に事前にラベル付けされた項目を含むタスク

また、ML によるラベル付けが有効になっている場合は、次のトレーニングが実行されるタイミングが、小さな進行状況バーに表示されます。  [実験] セクションには、機械学習の各実行に対してリンクが用意されています。

* トレーニング - ラベルを予測できるようにモデルをトレーニングする
* 検証 - このモデルの予測を項目の事前ラベル付けのために使用するかどうかを判断する 
* 推論 - 新しい項目について予測を実行する
* 特徴量化 - 項目をクラスター化する (画像の分類プロジェクトの場合のみ)

右側にあるのは、完了している当該タスクのラベルの分布です。  プロジェクトの種類によっては、1 つの項目に複数のラベルを含めることができ、その場合、ラベルの合計数が、項目の合計数よりも大きくなる可能性があるので注意してください。

### <a name="data-tab"></a>[データ] タブ

**[データ]** タブでは、データセットを表示して、ラベル付けされたデータを確認できます。 ラベルを表示するには、ラベル付けされたデータをスクロールします。 間違ってラベル付けされたデータが表示された場合は、それを選択して **[拒否]** を選択することで、ラベルを削除し、データをラベルなしのキューに戻すことができます。

### <a name="details-tab"></a>[詳細] タブ

プロジェクトの詳細を表示します。  このタブでは、次のことができます。

* プロジェクトの詳細と入力データセットを表示する
* 増分更新を有効にする
* プロジェクトでのラベル付きの出力を格納するために使用するストレージ コンテナーの詳細を表示する
* プロジェクトにラベルを追加する
* ラベルに付ける指示を編集する
* ML によるラベル付けの詳細を編集する (有効、無効を含む)

### <a name="access-for-labelers"></a>ラベラー用のアクセス権

[!INCLUDE [access](../../includes/machine-learning-data-labeling-access.md)]

## <a name="add-new-label-class-to-a-project"></a>プロジェクトに新しいラベル クラスを追加する

[!INCLUDE [add-label](../../includes/machine-learning-data-labeling-add-label.md)]

## <a name="export-the-labels"></a>ラベルをエクスポートする

ラベル付けプロジェクトの **[プロジェクトの詳細]** ページにある **[エクスポート]** ボタンを使用します。 いつでも、Machine Learning の実験用にラベル データをエクスポートできます。 

* 画像ラベルは以下としてエクスポートできます。
    * [COCO 形式](http://cocodataset.org/#format-data)。COCO ファイルは、Azure Machine Learning ワークスペースの既定の BLOB ストアにある *Labeling/export/coco* 内のフォルダーに作成されます。 
    * [ラベル付き Azure Machine Learning データセット](how-to-use-labeled-dataset.md)。 

エクスポートした Azure Machine Learning データセットには、Machine Learning の **[データセット]** セクションでアクセスします。 また、データセットの詳細ページには、Python からラベルにアクセスするためのサンプル コードも表示されます。

![エクスポートされたデータセット](./media/how-to-create-labeling-projects/exported-dataset.png)

ラベル付きデータを Azure Machine Learning データセットにエクスポートした後、AutoML を使用して、ラベル付きデータでトレーニングされた Computer Vision モデルを作成できます。 詳細については、「[Python を使用して Computer Vision モデルをトレーニングする AutoML を設定する (プレビュー)](how-to-auto-train-image-models.md)」を参照してください。

## <a name="troubleshooting"></a>トラブルシューティング

[!INCLUDE [troubleshooting](../../includes/machine-learning-data-labeling-troubleshooting.md)]

### <a name="object-detection-troubleshooting"></a>物体検出のトラブルシューティング

|問題  |解決策  |
|---------|---------|
|オブジェクト検出のラベル付け中に Esc キーを押すと、左上隅にゼロ サイズのラベルが作成されます。 この状態でラベルを送信することはできません。     |   ラベルの横にある交差マークをクリックして、ラベルを削除してください。  |

## <a name="next-steps"></a>次のステップ

* [チュートリアル:初めての画像分類のラベル付けプロジェクトの作成に関するチュートリアル](tutorial-labeling.md)を参照してください。
* [画像にタグ付けする方法](how-to-label-data.md)
