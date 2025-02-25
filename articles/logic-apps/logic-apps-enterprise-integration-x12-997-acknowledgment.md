---
title: X12 997 受信確認とエラー コード
description: Azure Logic Apps での X12 メッセージの 997 機能確認とエラー コードについて学習します。
services: logic-apps
ms.suite: integration
author: praveensri
ms.author: psrivas
ms.reviewer: estfan, divswa, azla
ms.topic: reference
ms.date: 07/15/2021
ms.openlocfilehash: ec727997663286b732fe7bf1d00a50da2f706a0a
ms.sourcegitcommit: 7d63ce88bfe8188b1ae70c3d006a29068d066287
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/22/2021
ms.locfileid: "114481698"
---
# <a name="997-functional-acknowledgments-and-error-codes-for-x12-messages-in-azure-logic-apps"></a>Azure Logic Apps での X12 メッセージの 997 機能確認とエラー コード

Azure Logic Apps では、**X12** 操作を使用するときに、電子データ交換 (EDI) 通信用の X12 メッセージを処理するワークフローを作成できます。 EDI メッセージングでは、受信確認によって EDI インターチェンジの処理の状態が示されます。 [**X12 デコード** アクション](logic-apps-enterprise-integration-x12-decode.md)では、インターチェンジを受信するときに、有効になっている受信確認の種類と、指定されたレベルの検証に基づいて、1 つまたは複数の種類の受信確認を送信側に返すことができます。

たとえば、受信側は、処理中に発生する各エラーと共に ''*997 機能確認 (ACK)* '' を送信することで、受信した X12 でエンコードされたメッセージの機能グループ ヘッダー (GS) と機能グループ トレーラー (GE) を検証した状態を報告します。 **X12 デコード** アクションでは常に 4010 準拠の 997 ACK が生成されますが、[**X12 エンコード** アクション](logic-apps-enterprise-integration-x12-encode.md)と **X12 デコード** アクションの両方で 5010 準拠の 997 ACK を検証できます。

受信側は、機能グループ ヘッダー (GS) および機能グループ トレーラー (GE) エンベロープ内で 997 ACK を送信します。 しかし、この GS および GE エンベロープは、他のトランザクション セットと何ら変わりはありません。

このトピックでは、インターチェンジの 997 ACK セグメントとそれらのセグメントで使用されるエラー コードを含む、X12 997 ACK に関する簡単な概要を示します。 その他の関連情報については、次のドキュメントをご確認ください。

* [X12 TA1 技術受信確認とエラー コード](logic-apps-enterprise-integration-x12-ta1-acknowledgment.md)
* [B2B エンタープライズ統合用の X12 メッセージを交換する](logic-apps-enterprise-integration-x12.md)
* [B2B エンタープライズ統合用の EDIFACT メッセージを交換する](logic-apps-enterprise-integration-edifact.md)
* [Azure Logic Apps とは](logic-apps-overview.md)
* [Azure Logic Apps と Enterprise Integration Pack を使用した B2B エンタープライズ統合ソリューション](logic-apps-enterprise-integration-overview.md)

<a name="997-ack-segments"></a>

## <a name="997-ack-segments"></a>997 ACK セグメント

次の表では、インターチェンジ内の 997 ACK セグメントについて説明し、次の定義を使用します。

* M = 必須
* O = 省略可能

| 位置 | セグメント ID | 名前 | 必須の指定 <br>(指定要件) | 最大使用数 | ループの繰り返し |
|----------|------------|------|--------------------------------------|-------------|-------------|
| 010 | ST | トランザクション セット ヘッダー (受信確認の場合) | M | 1 | - |
| 020 | AK1 | 機能グループ応答ヘッダー | M | 1 | - |
| 030 | AK2 | トランザクション セット応答ヘッダー | O | 1 | 999999 <br>(ループ ID = AK2) |
| 040 | AK3 | データ セグメントの説明 | O | 1 | 999999 <br>(ループ ID = AK2 または AK3) |
| 050 | AK4 | データ要素の説明| O | 9 9 | - |
| 060 | AK5 | トランザクション セット応答トレーラ | M | 1 | - |
| 070 | AK9 | 機能グループ応答トレーラ | M | 1 | - |
| 080 | SE | トランザクション セット トレーラー (受信確認の場合) | M | 1 | - |
|||||||

以下のセクションでは、各 AK セグメントについて詳しく説明します。 AK2 から AK5 のループでは、セグメントによって、トランザクション セットでのエラーについての情報が提供されます。

### <a name="ak1"></a>AK1

必須の AK1 セグメントでは、次のデータ要素を使用して確認する機能グループを識別します。

| 要素 | 説明 |
|---------|-------------|
| AK101 | 必須。確認する機能グループの機能グループ ID (GS01) を識別します。 |
| AK102 | 必須。確認する機能グループのグループ制御番号 (GS06 および GE02) を識別します。 |
| AK103 | 省略可能。元のトランザクションの GS08 で送信された EDI 実装のバージョンを識別します。 AK103 では、受信 5010 準拠の 997 ACK がサポートされます。 |
|||

### <a name="ak2"></a>AK2

省略可能な AK2 セグメントには、受信した機能グループ内のトランザクション セットの受信確認が含まれます。 複数の AK2 セグメントが存在する場合は、一連のループとして送信されます。 各 AK2 ループでは、受信順序を使用してトランザクション セットを識別します。 トランザクション セットにエラーがある場合、AK2 ループには AK3、AK4、および AK5 セグメントが含まれます。 詳細については、このトピックの後述のセグメントの説明をご確認ください。

AK2 セグメントでは、次のデータ要素を使用してトランザクション セットを識別します。

| 要素 | 説明 |
|---------|-------------|
| AK201 | 必須。確認するトランザクション セットのトランザクション セット ID (ST01) を識別します。 |
| AK202 | 必須。確認するトランザクション セットのトランザクション セット制御番号 (ST02 および SE02) を識別します。 |
| AK203 | 省略可能。元のトランザクションの ST03 で送信された EDI 実装のバージョンを識別します。 AK203 では受信 5010 準拠の 997 がサポートされます。 |
|||

#### <a name="generate-ak2-segments"></a>AK2 セグメントを生成する

AK2 セグメントを ''*すべて*'' の受理および拒否されたトランザクション セットに対して生成するか、拒否されたトランザクション セットに対して ''*のみ*'' 生成するかを指定できます。 それ以外の場合は、Azure Logic Apps により、拒否されたトランザクション セットに対して ''*のみ*'' が AK2 ループが生成されます。 契約が応答対象のインターチェンジに対して解決されない場合、997 の生成の設定は既定でフォールバック契約の設定になり、受理されたトランザクション セットの AK2 セグメントは生成されません。

AK501 == A の場合に受理されたトランザクション セットの AK2 セグメントを Azure Logic Apps で生成するには、これらの手順に従います。

1. Azure portal で、統合アカウントを開いてから、X12 取引先との間の X12 契約の成果物を開きます。

1. **[受信設定]** ペインを開き、 **[FA が必要]** が選択されているのを確認します。 その後、 **[AK2 / IK2 ループを含める]** を選択できます。

### <a name="ak3"></a>AK3

AK3 セグメント (省略可能) は、データ セグメント内のエラーおよびそのデータ セグメントの場所を報告します。 AK3 セグメントは、エラーを含むトランザクション セットのセグメントごとに作成されます。 複数の AK3 セグメントが存在する場合は、ループごとに 1 つのセグメントを含む一連のループとして送信されます。 AK3 セグメントでは、エラーを含む各セグメントの場所を指定し、次のデータ要素を使用して、その場所で見つかった構文エラーの種類を報告します。

| 要素 | 説明 |
|---------|-------------|
| AK301 | 必須。X12 セグメント ID (NM1 など) を使用して、エラーを含むセグメントを識別します。 |
| AK302 | 必須: エラーを含むセグメントのセグメント カウントを識別します。 ST セグメントは `1` で、セグメントごとにセグメント カウントが 1 ずつ増えます。 |
| AK303 | 必須。境界ループを識別します。これは、ループ開始 (LS) セグメントとループ終了 (LE) セグメントで囲まれたループです。 AK303 には、エラーを含むセグメントの範囲を示す LS セグメントと LE セグメントの値が含まれます。 |
| AK304 | 省略可能。データ セグメント内のエラーのコードを指定します。 AK304 は省略可能ですが、識別されたセグメントにエラーが存在する場合は、要素が必須となります。 AK304 エラー コードについては、[「997 ACK エラー コード」のデータ セグメントの説明](#997-ack-error-codes)をご確認ください。 |
|||

### <a name="ak4"></a>AK4

AK4 セグメント (省略可能) は、データ要素または複合データ構造体のエラーを報告し、そのデータ要素の場所を示します。 AK4 セグメントは、AK304 データ要素が `"8", "Segment has data element errors"` の場合に送信され、各 AK3 セグメント内で最大 99 回繰り返すことができます。 AK4 セグメントでは、エラーを含む各データ要素または複合データ構造体の場所を指定し、以下のデータ要素を使用して、その場所で見つかった構文エラーの種類を報告します。

| 要素 | 説明 |
|---------|-------------|
| AK401 | 必須。AK41.1、AK41.2、および AK41.3 というフィールドがある複合データ要素 <p><p>- AK401.1: 数値カウントを使用して、エラーを含むデータ要素または複合データ構造体を識別します。 たとえば、セグメントの 2 番目のデータ要素にエラーが含まれている場合は、AK401 が `2` と等しくなります。 <br>AK401.2: エラーを含む複合データ構造体のコンポーネント データ要素の数値カウントを識別します。 AK401 で非複合データ構造体のエラーが報告される場合は、AK401.2 に値が設定されません。 <br>- AK41.3: 省略可能。このフィールドは繰り返されるデータ要素の位置です。 AK41.3 では受信 5010 準拠の 997 がサポートされます。 |
| AK402 | 省略可能。エラーを含む要素の単純な X12 データ要素番号を識別します。 たとえば、NM101 の場合、単純な X12 データ要素番号は 98 です。 |
| AK403 | 必須。識別された要素のエラーを報告します。 AK403 エラー コードについては、[「997 ACK エラー コード」のデータ要素の説明](#997-ack-error-codes)をご確認ください。 |
| AK404 | 省略可能。エラーを含む、識別されたデータ要素のコピーが含まれます。 文字が無効であることを示すエラーの場合、AK404 は使用されません。 |
|||

### <a name="ak5"></a>AK5

AK5 セグメントは、AK2 セグメントで識別されたトランザクション セットが受理されるか、拒否されるか、およびその理由を報告します。 省略可能な AK2 ループが受信確認に含まれる場合、AK5 セグメントは必須です。 AK4 セグメントでは、1 つの必須データ要素を使用してトランザクション セットの状態を指定し、トランザクション セットの構文編集に基づいて、1 から 5 個の省略可能なデータ要素を使用するエラー コードを提供します。

| 要素 | 説明 |
|---------|-------------|
| AK501 | 必須。識別されたトランザクション セットを受理するか拒否するかを指定します。 AK501 エラー コードについては、[「997 ACK エラー コード」のトランザクション応答トレーラー](#997-ack-error-codes)をご確認ください。 |
| AK502 から AK506 | 省略可能。エラーの性質を示します。 AK502 エラー コードについては、[「997 ACK エラー コード」のランザクション セット応答トレーラー](#997-ack-error-codes)をご確認ください。 |
|||

### <a name="ak9"></a>AK9

AK9 セグメント (必須) は、AK1 セグメントで識別された機能グループが受理されるか、拒否されるか、およびその理由を示します。 AK9 セグメントでは、4 つの必須データ要素を使用して、トランザクション セットの状態とエラーの性質を指定します。 セグメントでは、1 から 5 個の省略可能な要素を使用して、示されたエラーを指定します。

| 要素 | 説明 |
|---------|-------------|
| AK901 | 必須。AK1 で識別された機能グループを受理するか拒否するか指定します。 AK901 エラー コードについては、[「997 ACK エラー コード」の機能グループ応答トレーラー](#997-ack-error-codes)をご確認ください。 |
| AK902 | 必須。識別された機能グループ トレーラー (GE01) に含まれるトランザクション セットの数を指定します。 |
| AK903 | 必須。受信されたトランザクション セットの数を指定します。 |
| AK904 | 必須。識別された機能グループ内で受理されたトランザクション セットの数を指定します。 |
| AK905 から AK909 | 省略可能。識別された機能グループに示された 1 から 5 のエラーを示します。 AK905 から AK909 のエラー コードについては、[「997 ACK エラー コード」の - 機能グループ応答トレーラー](#997-ack-error-codes)をご確認ください。 |
|||

<a name="997-ack-error-codes"></a>

## <a name="997-ack-error-codes"></a>997 ACK エラー コード

このセクションでは、[997 ACK セグメント](#997-ack-segments)で使用されるエラー コードについて説明します。  次の表は、X12 仕様で定義されている、Azure Logic Apps での X12 メッセージ処理用のサポート対象およびサポート非対象のエラー コードの一覧です。

### <a name="ak304-error-codes---data-segment-note"></a>AK304 エラー コード - データ セグメントの説明

次の表は、AK3 セグメントの AK304 データ要素 (データ セグメントの説明) で使用されているエラー コードの一覧です。

| エラー コード | 条件 | サポート対象 |
|------------|-----------|------------|
| 1 | セグメント ID が認識されません | はい |
| 2 | セグメントが正しくありません | はい |
| 3 | 必須セグメントがない | はい |
| 4 | 最大時間を超えるループが発生 | はい |
| 5 | セグメントの最大使用回数を超えた | はい |
| 6 | 定義済みの変換セットにセグメントが含まれていない | はい |
| 7 | セグメントのシーケンスが不適切 | はい |
| 8 | セグメントでデータ要素エラーが発生しています | はい |
| 511 | 末尾に区切り文字がある (カスタム コード) | はい |
||||

### <a name="ak403-error-codes----data-element-note"></a>AK403 エラー コード - データ要素の説明

次の表は、AK4 セグメントの AK403 データ要素 (データ要素の説明) で使用されるエラー コードの一覧です。

| エラー コード | 条件 | サポート対象 |
|------------|-----------|------------|
| 1 | 必須のデータ要素がありません | はい |
| 2 | 必須の条件付きデータ要素がありません | はい |
| 3 | データ要素が多すぎます | はい |
| 4 | データ要素が短すぎる | はい |
| 5 | データ要素が長すぎる | はい |
| 6 | データ要素に無効な文字があります | はい |
| 7 | 無効なコード値 | はい |
| 8 | 無効な日付 | はい |
| 9 | 無効な時刻 | はい |
| 10 | 除外条件の違反 | はい |
||||

### <a name="ak501-error-codes---transaction-set-response-trailer"></a>AK501 エラー コード - トランザクション セット応答トレーラー

次の表は、AK5 セグメントの AK501 データ要素 (変換セット応答トレーラー) で使用されるエラー コードの一覧です。

| エラー コード | 条件 | サポート対象 |
|------------|-----------|------------|
| A | 同意 | はい |
| E | 受理したがエラーが検出された | はい <p><p>**注**: エラー コードがない場合、`E` の状態になります。 |
| M | 拒否。メッセージ認証コード (MAC) は失敗しました | いいえ |
| P | 一部受理、1 つ以上のトランザクション セットが拒否された | はい |
| R | 拒否 | はい |
| W | 拒否、検証テストに失敗した | いいえ |
| X | 拒否、暗号解読後のコンテンツを分析できない | いいえ |
||||

### <a name="ak502-to-ak506-error-codes---transaction-set-response-trailer"></a>AK502 から AK506 のエラー コード - トランザクション セット応答トレーラー

次の表は、AK5 セグメントの AK502 から AK506 のデータ要素 (トランザクション セット応答トレーラー) で使用されるエラー コードの一覧です。

| エラー コード | 条件 | サポート対象または <br>AK501 との関連性 |
|------------|-----------|-----------------------------------------|
| 1 | トランザクション セットがサポートされていない | はい、R |
| 2 | トランザクション セットのトレーラーがない | はい、R |
| 3 | ヘッダーとトレーラーのトランザクション セット制御番号が一致しない | はい、R |
| 4 | 含まれているセグメントの数が実際の数と一致しない | はい、R |
| 5 | エラーに 1 つ以上のセグメントが含まれています | はい、R |
| 6 | トランザクション セット識別子がない、または無効 | はい、R |
| 7 | トランザクション セット制御番号がない、または無効。トランザクション番号が重複しているおそれがあります | はい、R |
| 8 ～ 27 | - | いいえ |
||||

### <a name="ak901-error-codes---functional-group-response-trailer"></a>AK901 エラー コード - 機能グループ応答トレーラー

次の表は、AK9 セグメントの AK901 データ要素 (機能グループ応答トレーラー) で使用されるエラー コードの一覧です。

| エラー コード | 条件 | サポート対象または <br>AK501 との関連性 |
|------------|-----------|-----------------------------------------|
| A | 同意 | はい |
| E | 受理したが、エラーが検出された | はい |
| M | 拒否。メッセージ認証コード (MAC) は失敗しました | いいえ |
| P | 一部受理、1 つ以上のトランザクション セットが拒否された | はい |
| R | 拒否 | はい |
| W | 拒否、検証テストに失敗した | いいえ |
| X | 拒否、暗号解読後のコンテンツを分析できない | いいえ |
||||

### <a name="ak905-to-ak909-error-codes---functional-group-response-trailer"></a>AK905 から AK909 のエラー コード - 機能グループ応答トレーラー

次の表は、AK9 セグメントの AK905 から AK909 のデータ要素 (機能グループ応答トレーラー) で使用されるエラー コードの一覧です。

| エラー コード | 条件 | サポート対象または <br>AK501 との関連性 |
|------------|-----------|-----------------------------------------|
| 1 | 機能グループがサポートされていません。 | いいえ |
| 2 | 機能グループのバージョンがサポートされていません。 | いいえ |
| 3 | 機能グループ トレーラーがありません。 | はい |
| 4 | 機能グループ ヘッダーとトレーラーのグループ制御番号が一致しません。 | はい |
| 5 | 含まれているトランザクション セットの数が実際の数と一致しません。 | はい |
| 6 | グループ制御番号の構文が不正。グループ制御番号が重複しているおそれがあります | はい |
| 7 から 26 | - | いいえ |
||||

## <a name="next-steps"></a>次のステップ

* [B2B エンタープライズ統合用の X12 メッセージを交換する](logic-apps-enterprise-integration-x12.md)